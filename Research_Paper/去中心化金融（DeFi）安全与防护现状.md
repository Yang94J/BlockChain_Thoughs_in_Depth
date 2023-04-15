**去中心化金融（DeFi）安全与防护现状**

# 前情
依然是以类似论文的形式去分享我关于区块链安全的一些看法，只是一点拙见，欢迎大家多多交流。DeFi在我看来，不只是有关虚拟货币，而是一种崭新的金融、甚至社交形式，其安全不可忽略。

# 前言
去中心化金融（DeFi,Decentralized Finance）目前是区块链与加密货币行业增长最为快速的领域之一。相较于传统金融，DeFi应用利用去中心化网络的透明性及开放性为用户提供多种类型的无治理机构金融服务^[1]^。近年来，人们对DeFi的认知有了很大的提升，其锁定的价值在2018年10月仅为34亿美元，但在2021年底就超过了1000亿美元^[2]^。

然而，DeFi原生的开源性、开放性也对其安全性提出了巨大的挑战。在其巨大的潜力背后，诸多黑客攻击、漏洞及骗局为DeFi的光明带来了不确定性。美国CNBC报道称“人们需要知道DeFi投资是具有高度风险的”^[3]^。据统计，仅仅是在 2021 年上半年，整个区块链生态就发生了 78 起较大的安全事件，其中涉及 DeFi 的安全事件更是高达50余起。在2021年8月，54%的安全事件都与DeFi有关，而这一比例在一年之前仅为3%。因此，总结DeFi应用安全的常见问题，并归纳业界对DeFi安全的探索，能有效提升DeFi的安全性，保障金融服务的持续稳定运行。本文结合实际DeFi安全事件分析了DeFi常见的安全问题，并探讨了业界对DeFi安全的多种探索方式，具有重要的学术意义与工程意义。

# 去中心化金融常见安全问题概述
DeFi从发展至今，出现过许多的安全问题，不少Defi项目因此遭受到巨额损失。由于Defi项目大多基于智能合约运行，而后者并不像传统APP可以随时下架，一旦部署便不可撤回。当合约出现问题或遭受攻击时，状态往往不可逆转。

总结DeFi安全问题，可以发现，DeFi既存在诸如退出骗局（Exit Scam）、市场操纵（Market Manipulation）、重入攻击等常见问题，也存在如闪电贷、预言机攻击、回滚攻击、流动性耗尽等Defi专有安全问题。本文将从DeFi合约本身技术风险、DeFi项目方作恶、黑客攻击等角度对DeFi常见安全问题作出阐述，值得注意的是往往项目方及黑客的攻击都是通过合约的漏洞实现的，所以项目方问题、黑客问题往往也都伴随着合约问题。由于私钥窃取问题比较常见，本文对此不再进行具体阐述。

## 智能合约编码安全
DeFi的许多漏洞很复杂，但有时候如果没有经过仔细的代码审查，智能合约中简单的编码错误也会带来巨大的损失。DeFi合约编码安全常见问题如下：

### 溢出问题
溢出分为上溢出和下溢出。Solidity在0.8之前的版本并不会对数值运算进行整型溢出检查，而0.8之后的版本对unchecked关键字也缺乏相应的溢出检查手段。举例来说，DeFi项目对加密资产的锁定会有一个类型为uint256的期限限制变量如lockTime，如果通过增加锁定时间导致溢出，锁定资产将被迅速释放^[4]^。PoWHC（CVE-2018-10299）由于并未对合约的上/下溢出作检查，高达80以太从合约中被窃取^[5]^。Defi作为金融服务应用，如果对溢出问题未加重视，很可能会遭受重大损失。

### 特权函数暴露
合约中需要定义函数的可见性，用户往往通过ABI调用外部或公共函数，经过复杂的检查（如合约所有者权限、关键参数检查），才能访问到内部函数直接对数据进行更改。如果内部特权函数的可见性对外暴露出来，将允许攻击者进行特权调用，造成重要损失。AVATerra Finance这一DeFi项目将铸币函数mint的可见性设置为public，这导致任意攻击者都能进行铸币操作，从而对项目和用户造成伤害^[6]^ 。

### 校验缺失漏洞
在DeFi合约中如果没有做好验证，就可能导致函数不按照预期结果执行。检查的缺失会导致攻击的入侵。如果未做所有者权限检查、目标地址检查等，会导致攻击的侵入。

## 黑客典型攻击手段
由于DeFi项目中往往存有大量资金，所有往往会引起黑客对于项目的觊觎。黑客会想方设法对合约展开攻击，以下是DeFi项目中黑客常见的攻击手段。

### 重入攻击
重入攻击是区块链合约中较为常见的攻击手段。对于受攻击合约，如果其对外部合约的调用出现在账本更新之前，且外部合约可以受到用户的控制，则黑客可以利用项目的重入风险，循环对受攻击合约中的资金进行操作。早在2017年，DAO遭受的重入攻击就已带来1.2亿美元的损失^[7]^。而就在2022年3月31日，Ola Finance遭遇智能合约重入攻击，损失约为467万美元^[8]^。
### 闪电贷攻击
闪电贷就是在同一区块中完成链上的借款和还款而无需抵押。由于区块可以包含多种操作，黑客可以以极低的成本操纵巨额资金，利用价差实现套利。2022年4月17日，算法稳定币项目Beanstalk Farms遭黑客攻击，黑客通过闪电贷发起提案，获利近8000万美元^[9]^。但其实，闪电贷本质上是一种中性的借贷应用，但却被经常用来操纵大笔资金以实现攻击。

### 操纵预言机攻击
预言机是区块链行业的基础设施，能够将区块链外的信息写入区块链内，而DeFi应用常常使用它来作为代币的价格来源。当攻击者利用闪电贷获得的大量资金（或通过其他方式）对价格源进行操纵后，DeFi应用的价值判断标准发生了变化。攻击者即可以在此错误的环境下，通过借贷、抵押等手段牟取暴利。在2020年，DeFi借贷协议Warp Finance遭到黑客攻击，造成了近800万美元的资产损失，起因就是用户通过闪电贷操纵了Uniswap上的价格，导致价格出现突变^[10]^。

### 治理攻击（提案攻击）
DAO已经形成DeFi项目的新形式。DAO协议的资金管理、重要参数的修改、协议升级都通过 DAO合约进行管理。而治理代币的持有者可以向 DAO合约发起提案、投票、执行提案。当DAO的提案模型设计得不够合理时，黑客可以通过购买（或通过闪电贷借出）大量代币以增持投票权。当黑客发起不合理提案但仍能通过投票通过时，就可以对项目进行攻击。2022年，DeFi项目Build Finance DAO收到了治理攻击，攻击者可以随意铸造货币并卖出，实现了47万美元的不当牟利^[11]^。

### 回滚攻击
许多DeFi项目允许用户通过发送金额并在同一区块内随机（或根据一定规律）做出回应。但如果黑客使用回滚攻击，即如果项目的回应并未达到自己的预期，即可主动进行回滚，从而规避风险，达成稳赚不赔的交易。

### 抢跑攻击
对于许多DeFi项目而言，其会持续从预言机中获取价格，通过该价格为用户的交易提供支撑。当存在抢跑机器人时，则会在预言机更新项目区块前，通过高额的Gas费用，实现抢跑，并从中套利。从2019年9月到2020年2月，Synthetix协议的套利机器人不断地在预言机更新之前运行，当Synthetix的预言机在链上发布价格反馈时，机器人会通过支付高额的交易费用，在预言机有机会更新价格之前被纳入以太坊区块之中^[12]^。

## 项目方作恶
### Rug-pull
Rug pull是指项目方在收益达到一定程度后，选择放弃当前项目，撤出流动性并退出。由于项目方拥有最大的权限，所以一般项目方作恶往往难以阻止。2021全年，DeFi项目Rug-pull涉及的总金额已经达到了28亿美元^[13]^，占全年DeFi攻击事件损失的37%，远远超过过了2020年的1%。

### 谎称黑客攻击
项目方往往会故意留下后门，并以黑客的形式，主动展开攻击，在得手后又会将事件的缘由推到黑客攻击上。由于区块链的匿名性，这种类型作恶方式往往难以识别并判定。

## 其他风险

### 政策风险
DeFi项目存在多种形式，但DeFi项目很可能会由于任意性来违反当地的法律政策，譬如ICO(Initial Coin Offering)。ICO是区块链项目的融资方式，但由于其未经批准和监管，在许多国家都属于违法行为。如果没有针对政策因素进行考量，DeFi项目很容易就失去发展的土壤。

# 去中心化金融常见防护技术研究
## DeFi合约审计
DeFi项目智能合约安全审计是对区块链应用程序智能合约的彻底分析，以纠正设计问题、代码错误或安全漏洞。审计方与项目方通过就规格进行沟通，实现单元测试、集成测试，并完成项目合约的代码路径覆盖。同时审计方还可以利用自动错误检测工具（如符号执行工具），并通过人工审计找出隐藏的错误。目前比较出名的合约审计单位包括Certik、SecureBlock等等。当然我们不能仅仅依赖审计，提升合约编写能力，提升合约安全意识。

## 引入机器学习等手段实现自动识别 
对合约的审计不一定完全要通过审计方，也可以通过机器学习等手段进行合约识别。基于合约代码，可以实现特征抓取，根据已有训练集，即可以实现有监督学习，实现高精度、高反应的自动识别^[14]^。

## 提升预言机相关合约的抗风险能力
对使用预言机“喂”数据的合约来说，不应当对预言机的合法性、正义性进行完全的相信，这是因为预言机也具有脆弱性，如果被操控而合约却又不加以限制、控制，这将会带来很大的问题。在使用预言机时应当提升合约的抗风险能力。具体来说，可以引用著名的去中心化预言机（如Uniswap），这会使得价格难以被轻易操纵。同时，可以引入时间加权平均价格（TWAP），减少因价格突变而导致的影响。又或者合约可以M-of-N喂价，从而从多个来源的价格中选出最为合理的部分。

## 引入攻击探测能力实现快速响应
黑客的攻击有时是难以避免的，此时项目应做好快速相应的准备。搭建攻击探测器往往会是有效的做法。设置好探测器的嗅探范围，令其在链上监听每一笔交易，预估交易会带来的影响，如果结果难以接受，则可以实时发出告警，实现即时、快速的攻击探测。在攻击初期即发现攻击，探知攻击意图，从而挽回部分损失。部分产品，如BlockEye即支持主动分析合约漏洞，并实现链上的交易监听^[15]^
。

## 提升攻击回溯、追踪能力
在黑客实现攻击牟利后，项目方应与相关责任方配合，在链上实现对黑客的回溯和追踪。回溯包括对黑客暴露在链上的所有行为展开回顾，研究攻击方的意图变化情况。追踪包括对项目方得利后的链上行为进行追查，分析受损资金的去向，为后续处理提供依据。目前已有研究实现了追踪的可视化展示，有助于对攻击进行复盘和追回^[16]^。

# 总结
DeFi其实并不是一个很新的概念，当区块链作为去中心化的代表存在于网络上，人们就开始试图赋予其金融属性。然后并不是发行一个凭证就是所谓的金融，DeFi项目需要进行完善、全面的设计以用于完备的经济模式和抗风险能力。这就是为什么现在智能合约需要经过层层审计。有时攻击要比设计、防御简单的多。

# 引用
[1] Harvey, Campbell R., Ashwin Ramachandran, and Joey Santoro. DeFi and the Future of Finance. John Wiley & Sons, 2021.
[2] DeFi Pulse, https://www.defipulse.com/, 2022
[3] Here's what to know about cryptocurrency-based DeFi, https://www.cnbc.com/2021/06/18/whats-defi-crypto-based-decentralized-finance-explained.html, CNBC, 2021
[4] D., G. W. P., Antonopoulos, A. M. (2018). Mastering Ethereum: Building Smart Contracts and DApps. Germany: O'Reilly Media.
[5] CVE-2018-10299, https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2018-10299,2018
[6]AVATerra Faced Exploitation on the Day Of its Launch,  https://www.cryptotimes.io/avaterra-faced-exploitation-on-the-day-of-its-launch/ , 2021
[7] What is a Reentrancy Attack? https://www.certik.com/resources/blog/3K7ZUAKpOr1GW75J2i0VHh-what-is-a-reentracy-attack, 2022
[8] Ola Finance Says Attackers Stole $4.7M in 'Re-Entrancy' Exploit,  https://finance.yahoo.com/news/ola-finance-says-attackers-stole-074434812.html, 2022
[9] Beanstalk DeFi project robbed of $182 million in flash loan attack,  https://www.zdnet.com/article/beanstalk-defi-project-robbed-of-182-million-in-flash-loan-attack/, 2022
[10] Analysis of Warp Finance hacked incident, https://xn--wuslowmist-6y6t.medium.com/analysis-of-warp-finance-hacked-incident-cb12a1af74cc, 2021
[11] Build Finance DAO Suffers Governance Takeover Attack, https://cryptobriefing.com/build-finance-dao-suffers-governance-takeover-attack/, 2022
[12] Frontrunning Synthetix: a history, https://blog.synthetix.io/frontrunning-synthetix-a-history/ , 2021
[13] DeFi ‘Rug Pull’ Scams Pulled In $2.8B This Year: Chainalysis, https://www.coindesk.com/markets/2021/12/17/defi-rug-pull-scams-pulled-in-28b-this-year-chainalysis/ , 2022
[14] Trozze, Arianna & Kleinberg, Bennett & Davies, Toby. (2021). Detecting DeFi Securities Violations from Token Smart Contract Code with Random Forest Classification.
[15] Wang, Bin & Liu, Han & Liu, Chao & Yang, Zhiqiang & Qian, Ren & Zheng, Huixuan & Lei, Hong. (2021). BLOCKEYE: Hunting For DeFi Attacks on Blockchain.
[16] Wu, Zhiying & Liu, Jieli & Wu, Jiajing & Zheng, Zibin. (2022). TRacer: Scalable Graph-based Transaction Tracing for Account-based Blockchain Trading Systems.