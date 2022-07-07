---
layout:     post
title:      Hyperleger简介
subtitle:   超级账本孵化器简介
date:       2019-06-22
author:     Bingo
header-img: img/post-bg-linux.jpg
catalog: true
tags:
    - Hyperledger
    - 区块链
---

> [Hyperledger](https://wiki.hyperledger.org/start)是一项开源协作，旨在推动跨行业区块链技术的发展。 这是一项全球合作，包括银行，金融，物联网，制造，供应链和技术领域的领导者。 Linux Foundation在此基础上托管Hyperledger。 要了解更多信息，请访问：hyperledger.org。 Hyperledger不会推广单个区块链代码库或单个区块链项目。 相反，它使全球开发人员社区能够一起工作并分享想法，基础架构和代码。

### 为什么要用Hyperledger
当我们与别人公用数据库时存在以下问题：
- 您信任谁来共享数据
- 你怎么告诉某人谁在线
- 允许他们对数据库做什么操作
- 当总公司和某经销商想出售相同的商品怎么办
- 谁解决冲突或争议
Blockchain就是用来解决上述问题的技术
区块链（Blockchain）是比特币（Bitcoin）和其它加密数字货币（cryptocurrency）的底层技术原理

### 区块链（Blockchain）
- 区块链是一个去中心化的分布式数据库，他用来解决信任危机（你想分享使用一个数据库，但是对其他人的置信度不高）很有帮助
- 区块链的权限（permissions）与共识（consensus）
当两个节点对数据库中数据操作发生冲突时怎么办呢？区块链会用一个共识的过程解决上面这个问题。区块链使用共识系统来确保数据库中的信息总是最新的。例如，共识系统会根据预先定义好的规则判断哪个节点优先获得资源的使用权，并上锁。共识系统有许多不同的形式和名称。

### Hyperledger的诞生

**2015**：Hyperledger始于2015年，当时许多不同的公司都对区块链技术感兴趣，就合作开展了Hyperledger项目。<br>
**Linux**：Hyperledger是一个开源的项目，并且被置于Linux基金会的监护之下，在过去的几年里增长迅速。<br>
**数量惊人**：截至2017年，Hyperledger共有会员**230**多个，**10**个项目，**360W**行代码，来自世界多地近**2.8W**名参与者

#### 开源的好处：
- 持久性强
- 没有固定的供应商，客户可以随意切换
- 高质量的解决方案！（毕竟人多力量大）
- 可以通过访问源代码，定制和修复bug
- 降低成本<br>
（ps.当然还有诸多好处，比如脱离政府啊，什么的)

![Hyperledger生态圈](https://img-blog.csdnimg.cn/20190621205913493.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3lhbmh1aWJpbjMxNQ==,size_16,color_FFFFFF,t_70)

### Hyperledger设计理念
- 模块化
- 高度安全
- 可互操作
- 健全的api
- 利用加密的数字货币

![Hyperledger设计理念](https://img-blog.csdn.net/20180808120545256?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3lhbmh1aWJpbjMxNQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

# 框架
### Hyperledger推广和孵化的一些框架
 - 分布式分类框架
 - 智能合约引擎
 - 客户端库
 - 图形化界面
 - 实用工具库
 - 样本应用程序

##### 主流的Hyperledger框架
**[Hyperledger Borrow](https://www.hyperledger.org/projects/hyperledger-burrow)**：一个模块化的区块链客户端，自带一个智能合约解释器EVM。包括的组件：
 - 共识引擎
 - 区块链应用接口
 - 智能合约应用引擎
 - 网关

**[Hyperledger Fabric](https://www.hyperledger.org/projects/fabric)**：用于构建分布式分类帐解决方案的平台，其模块化结构可提供高度的机密性，灵活性，弹性和可扩展性，适用于任何行业。

**[Hyperledger Indy](https://www.hyperledger.org/projects/hyperledger-indy)**：一个分布式分类账本，专为提供给分散身份权限而构建的工具、库和可重用组件。Indy的特点：
 - 自主权
 - 隐私
 - 可验证的索赔

**[Hyperledger Iroha](https://www.hyperledger.org/projects/iroha)**：区块链框架，设计简单，易于整合到企业基础架构项目中。特点：
- 结构简单
- 模块化，C++设计
- 重视移动应用程序开发
- Sumeragi：一种新的，基于区块的拜占庭容错算法

**[Hyperledger Sawtooth](https://www.hyperledger.org/projects/sawtooth)**：用于构建，部署和运行分布式分类帐的模块化平台。Sawtooth采用一种新的共识，经过时间PoET的证明，它比工作证明PoW消耗的资源少得多。Sawtooth的技术创新：
- 动态共识
- 利用时间验证
- 事务家族：抽象了智能合约，用户能够以他们选择的语言编写智能合约。
- Compatability with Ethereum contracts
- 并行执行事务
- 私有事务

**[Hyperledger Grid](https://www.hyperledger.org/projects/grid)**：Hyperledger Grid是一个用于构建包含分布式分类帐组件的供应链解决方案的平台。它提供了一套不断增长的工具，可加速供应链智能合约和客户端界面的开发。此项目不是分布式分类帐或客户端应用程序的实现。相反，Hyperledger Grid提供以供应链为中心的库，数据模型和软件开发工具包（SDK）作为模块化，可重用的组件。
Hyperledger Grid项目包括几个存储库：此存储库包含核心组件，例如以供应链为中心的数据类型和智能权限代码。
- grid-contrib存储库包含智能合约（也称为“事务系列”）的示例域模型和参考实现。
- grid-rfcs存储库包含针对Hyperledger Grid的建议和批准更改的RFC（注释请求）。

# 工具
### 主流的Hyperledger工具
**[Hyperledger Aries](https://www.hyperledger.org/projects/aries)**：Hyperledger Aries是Linux基金会托管的Hyperledger项目之一。Hyperledger Aries提供了一个共享，可重用，可互操作的工具包，专为创建，传输和存储可验证数字凭证的计划和解决方案而设计。它是基于区块链的点对点交互的基础设施。它包括用于区块链客户端的共享加密+++钱包以及用于允许这些客户端之间的分类账交互的通信协议。该项目消耗Hyperledger Ursa提供的加密支持，以提供安全的秘密管理和分散的密钥管理功能。Hyperledger Aries最初由来自Sovrin Foundation，不列颠哥伦比亚省政府和其他Indy社区开发者的开发人员提供。

**[Hyperledger Caliper](https://www.hyperledger.org/projects/caliper)**：区块链基准测试工具，通过使用一组预定义的用例来测量任何区块链的性能。

**[Hyperledger Cello](https://www.hyperledger.org/projects/cello)**：通过自动化方式配置和管理区块链操作，将按需部署模型引入区块链生态系统，从而减少工作量。

**[Hyperledger Composer](https://www.hyperledger.org/projects/composer)**：一个开源的开发工具集和框架，可以更轻松的开发区块链应用程序。Composer支持Fabric框架。

**[Hyperledger Explorer](https://www.hyperledger.org/projects/explorer)**：用于查看网络信息的仪表板，包括块，节点日志，统计信息，智能合约和事务。Explorer支持Fabric框架。

**[Hyperledger Quilt](https://www.hyperledger.org/projects/quilt)**：一组通过实施ILP提供互操作性的工具，ILP主要是旨在跨分布式和非分布式分类帐传输价值的支付协议。

> 总结：Hyperledger工作组拥有许多出色的技术资源，并且对任何对其主题感兴趣的人开放。 例如，架构工作组有关于许可区块链基本原理的大量文档。 如果您正在寻找技术细节，那么该组是一个很好的资源。 特定于应用程序的工作组也是学习的好地方。 例如，身份工作组花了很多时间讨论和记录区块链如何启用身份解决方案。