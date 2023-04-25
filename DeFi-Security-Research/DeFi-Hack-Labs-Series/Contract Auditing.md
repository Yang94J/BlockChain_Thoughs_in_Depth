合约审计入门

PS ： 参考DeFiHackLabs中的<a href="https://github.com/SunWeb3Sec/DeFiHackLabs/tree/main/academy/solidity">相关系列</a>，在学习过程中留下了笔记，非常感谢，都是干货。向大家推荐。

# 1.  合约审计方法与技巧

主要目的是建立属于自己的合约审计流程，从阅读文档、了解项目到建立大致框架，再通过工具进行静态攻击，再进行人工分析，通过POC进行测试并最终撰写报告。

一些可用工具：

- Solidity Metrics
- Solidity Visual Developer
- Mythril
- Foundry Fuzz

推荐了一些CTF：

- Ethernaut (已完成)
- Damn Vulnerable DeFi （已完成）
- CTF Protocol （已完成）
- QuillCTF
- Paradigm CTF （准备开始）

<hr />

# 2. Bug in Compound

这个Bug是通过存入大笔来影响价格：

```solidity
        if (totalSupply() == 0) {
            _shares = _amountToDeposit;
        } else {
            /**
             * # of shares owed = amount deposited / cost per share, cost per
             * share = total supply / total value.
             */
            _shares = (_amountToDeposit * totalSupply()) / (_valueBefore);
        }
        _mint(msg.sender, _shares);
```

当`_valueBefore`足够大时，`shares`应该是为0的。注意这里`totakSupply`和`valueBefore`对应的不是同一种通证。

<hr />

# 3 Liquid Staking Audit

流动性质押：质押ETH获得质押通证。

审计过程：

	- 检查功能是否齐备
	- 检查费用、通证经济学是否和白皮书对应
	- 查看机制，设置检查点，判断机制是否如预期