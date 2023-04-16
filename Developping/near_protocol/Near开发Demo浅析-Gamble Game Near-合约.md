<h1> [Near Protocol] Near开发Demo浅析-Gamble Game Near</h1>

<h2>智能合约 </h2>

大家好，Near Protocol是近年来比较引人注目的公链项目，其分片技术也使得其性能和稳定性出现了很大的提升和突破。我最近对Near相关开发的基础概念进行了学习，以掷色子游戏的概念为基础，开发了Near 合约+Dapp的Demo，现在和大家分享讨论，希望都能有所进益。

源代码在Github上可以找到，以下是智能合约链接：

```
https://github.com/Yang94J/Gamble_Game_Near
```

<hr>

<h3>简介</h3>

合约名称：**Gamble**
合约功能：**允许用户通过发送Near进行投注，通过随机数模拟掷色子活动，当掷到6时，可以获得6倍返还。**
实现语言：**Rust**
部署网络： **Near Testnet**
Near-sdk文档：**https://www.near-sdk.io**
<hr>

<h3>环境搭建</h3>

- Prerequisite : 系统中应当已经安装好Rust
- 安装wasm32（编译）：
```
		rustup target add wasm32-unknown-unknown
```
- 初始化项目（编译）：
```
		cargo new Gamble_Game_Near
```
- 在生成的工程上进行目录的修改，最终结构如下：
![工程目录示意图](https://img-blog.csdnimg.cn/1d989114d28b4828b5e4c15575752271.png)
其中lib.rs是合约文件，而Cargo.toml描述依赖（你可以理解为类似于Maven项目中的pom.xml），而Cargo.lock则包含依赖项的确切信息。

<h3>代码解析</h3>

**[Cargo.toml](https://github.com/Yang94J/Gamble_Game_Near/blob/master/Cargo.toml)**

以下是Cargo.toml重点部分

```
[lib]
crate-type = ["cdylib","rlib"]

[dependencies]
uint = {version = "0.9.0", default-features = false}
near-sdk = "4.0.0-pre.7"
near-contract-standards = "4.0.0-pre.7"

```
其中crate-type指定为C动态链接库及Rust链接库，依赖使用4.0.0-pre.7版本的near-sdk

**[lib.rs](https://github.com/Yang94J/Gamble_Game_Near/blob/master/src/lib.rs)**

lib.rs首先从near-sdk中导入相关函数及接口：

```
use near_sdk::{
    borsh::{self, BorshDeserialize, BorshSerialize},
    near_bindgen, Balance,PanicOnDefault,
    env, require, log,  Promise, 
};
```

同时通过`#[near_bindgen]`和`#[derive(BorshDeserialize, BorshSerialize,PanicOnDefault)]`来声明Gamble类为智能合约类。`#[near_bindgen]`宏用以生成Near合约所需的代码并对外暴露接口。`#[derive(BorshDeserialize, BorshSerialize,PanicOnDefault)]`则声明序列化、反序列化，且要求实现构造函数。Gamble类中包含`gamble_min_price`和`gamble_max_price`属性，为投注的最小值和最大值。

```
pub struct Gamble {

    // Minimum price that should be transfered to the contract, revert otherwise
    gamble_min_price : Balance,

    // Maximum price that should be transfered to the contract, revert otherwise
    gamble_max_price : Balance,

}
```
本文后续实现Gamble类的相关方法，（注意&self 和 &mut self的区别，类似于solidity view和change的区别）如下：

- pub fn new() -> Self ：用以初始化合约，设置`gamble_min_price`和`gamble_max_price`属性。

```
    #[init]
    pub fn new() -> Self {
        
        let account_balance = env::account_balance();
        let gamble_max_price = account_balance / (5 * FACTOR);
        log!("we have {} uints in total, be sure not to exceed the max gamble price limit {} to get {}X \n", account_balance, gamble_max_price, FACTOR);

        Self{
            gamble_max_price : gamble_max_price,
            gamble_min_price : 0,
        }
    }
```

- pub fn get_minimal_gamble_price(&self) -> u128 ：获取最小投注数量（ yoctoNEAR）

```
    // Get the Minimum amount of near to be transfered(Used for dapp, but usually won't as it's 0 all the time)
    pub fn get_minimal_gamble_price(&self) -> u128 {
        self.gamble_min_price
    }
```

- pub fn get_maximum_gamble_price(&self) -> u128：获取最大投注数量（ yoctoNEAR）

```
    // Get the Minimum amount of near to be transfered(Used for dapp)
    pub fn get_maximum_gamble_price(&self) -> u128 {
        self.gamble_max_price
    }    
```

- pub fn get_balance(&self) -> u128：获取合约余额（ yoctoNEAR）

```
    // Get contract balance U128
    pub fn get_balance(&self) -> u128 {
        env::account_balance()
    }  
```

- pub fn update_price(&mut self) ：更新投注最大值和最小值

```
    // Update price everytime the account balance changes
    // Only contract call
    fn update_price(&mut self){
        let account_balance = env::account_balance();
        self.gamble_max_price = account_balance / (5 * FACTOR);
        log!("we have {} uints in total, be sure not to exceed the max gamble price limit {} to get {}X \n", account_balance, self.gamble_max_price, FACTOR);
    }
```

- pub fn sponsor(&mut self)：赞助函数，即用户向合约账号发送near（ yoctoNEAR），最后更新投注上限和下限

```
    // The user could sponsor the contract(maybe only the owner will...)
    #[payable]
    pub fn sponsor(&mut self){
        let sponsor_id = env::signer_account_id();
        let deposit = env::attached_deposit();
        log!("sponsor {} has add {} to the game to increase balance, thank you ~ \n", sponsor_id, deposit);
        self.update_price();
    }
```

- pub fn gamble(&mut self) -> u8：投掷函数，返回投掷出的点数（1-6），最后更新投注上限和下限

```
    // The user could transfer near to get a chance to gamble
    // return the dice throwed by the user (Randomly generated)
    #[payable]
    pub fn gamble(&mut self) -> u8{
        let gambler_id = env::signer_account_id();
        let deposit = env::attached_deposit();

        require!(deposit>=self.gamble_min_price,"The gamble price must exceed gamble_min_price");
        require!(deposit<=self.gamble_max_price,"The gamble price must not exceed gamble_max_price");
        
        let num = self.rand_dice();

        if num == FACTOR as u8 {
            let amount = (deposit as f32 ) *(FACTOR as f32) * TAX;
            let amount_u128 = amount  as u128;
            log!("Congratuations to {}, he has won the gamble, the prize is {} \n",gambler_id,deposit);
            Promise::new(gambler_id).transfer(amount_u128);
        }
        self.update_price();
        return num;
    }
```

-  pub fn rand_dice(&self) -> u8 ：基于random_seed()，生成随机数并通过取余映射到1-6

```
    // Generate random number from 1 to 6
    pub fn rand_dice(&self) -> u8 {
        *env::random_seed().get(0).unwrap()%6+1
    }
```
后续则是单元测试部分，通过`fn get_context(input: Vec<u8>) -> VMContext`设置了测试背景，由`fn rand_test()`测试了随机数生成，`fn gamble_test()`则测试了合约初始化后的变量情况。

<h3>合约部署</h3>

- 在此之前请先安装[Near-cli](https://docs.near.org/docs/tools/near-cli)
- 编译生成

```
		cargo build --target wasm32-unknown-unknown --release
```
- 部署（先生成子账号再部署）

```
		near create-account gamble_game1.XXX.testnet --masterAccount XXX.testnet --initialBalance 100
		near deploy --wasmFile target/wasm32-unknown-unknown/release/gamble_game_near.wasm --accountId gamble_game1.XXX.testnet
```

- 测试

```
		near call gamble_game1.XXX.testnet new  --accountId gamble_game1.XXX.testnet
		near view gamble_game1.XXX.testnet get_balance
		near view gamble_game1.XXX.testnet get_maximum_gamble_price
		near call gamble_game1.XXX.testnet sponsor --deposit 1  --accountId XXX.testnet
		near call gamble_game1.XXX.testnet gamble --deposit 1  --accountId XXX.testnet
```

<hr>
至此，智能合约部分完成，后续会带来
[Near Protocol] Near开发Demo浅析-Gamble Game Near（二）：Dapp部分的介绍。