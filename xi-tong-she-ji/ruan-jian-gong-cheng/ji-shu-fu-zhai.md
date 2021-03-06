# 技术负债

本文主要介绍一下内容：

a. 确认技术债务量化维度 ；

b. 技术债务的识别与量化： 包括负面影响有多大，修复需要多大的成本(利息） ；

c. 技术债务有他相关的上下文;&#x20;

d.如何偿还技术债务

## 产生原因

故意：意识到债务&#x20;

无意：未识别债务&#x20;

鲁莽：未考虑债务的成本

&#x20;谨慎：经过了相当的评估

## 产生阶段

```
建筑债务
构建债务
代码债务
缺陷债务
设计债务
文件债务
基础设施债务
人员债务
处理债务
需求债务
服务债务
测试自动化债务
测试债务
```

## 影响对象

客户：经常被bug缠绕、长期缺失的feature不能上线

运营：不合理的界面设计、缺失的文档、算法或系统响应慢等等让运营工作如履薄冰；作为连接客户和内部的第一接口，前有客户的抱怨、后有不齐全的工具文档等。

市场：增加客户的开发成本等

运维：频繁的bug修复上线管理层：各方的抱怨让管理层崩溃，尤其是bug、延期等问题

研发：开发人员的工作比较多面，一方面开发新的代码，另一方面又需要维护他人遗留的代码和系统。没有人愿意产出低质量的产品、也没有人愿意接手满是坑的代码。

## 影响

可维护性（Maintainability）&#x20;

狭义上的代码问题，即代码本身可读性如何、是否容易被他人所理解、是否有明显的代码坏味道、是否容易扩展和增强。

可演进性（Evolvability）&#x20;

系统适应变化的能力，本质上一种架构的元特征（Meta-Characteristic），描述的是软件架构趋于目标演进的能力，演进目标并不仅局限于支撑功能快速迭代（Iteration）的灵活性（Flexibility），也可以是其他的架构属性（Quality Attribute），比如高可用性、可扩展性。

可见性（Visibility）

针对可见性的分析可以依赖于外部视角：对于最终用户来说，软件功能、设计和用户体验等方面的缺陷，导致用户无法顺利完成既定的业务流程，那么对于用户不可见的代码问题就升级为了可见的质量问题；对于需求提供方来说，臃肿的技术架构、散落各处的业务逻辑导致产品无法快速响应需求变化，导致交付延期，那么对于无技术背景的业务人员来说，难以理解的、不可见的架构问题就升级为了可见的软件交付风险。

### 技术债治理

#### 核心领域优于其他子域&#x20;

识别领域、子域是DDD战略设计的重要步骤，在识别子域之后我们还需要进一步分析哪些是核心域（Core Domain），哪些是支撑子域（Supporting SubDomain）和通用子域（Generic Subdomain）。核心域在业务上至关重要，它提供了区别于行业竞争对手的差异化优势，承载了业务背后最核心的基础理念。

#### 可演进性优于可维护性&#x20;

技术债导致的可演进性问题大多和架构相关，比如服务和服务之间的循环依赖、模块和模块之间的过度耦合、缺少模块化和服务边界的“大泥球”组件等，在添加新的功能时，这些架构的坏味道会给产品功能的迭代造成不少麻烦。比如服务之间如果存在循环依赖的问题，当你对系统进行少量更改时，它可能会对其他模块产生连锁反应，这些模块可能会产生意想不到的错误或者异常。此外，如果两个模块过度耦合、相互依赖，则单个模块的重用也变得极其困难。

可演进性问题可能会直接导致开发速度滞后，功能无法按期交付，使项目出现重大的交付风险。而且问题发生的时候往往已经“积重难返”，引入的技术债务没有在合适的时间得到解决，其产生的影响会像“滚雪球”一样越滚越大。在我所经历过的项目中有一个不太合理的模型设计，由于错过了最佳的纠正时间，为了实现新的业务功能最终不得不做服务拆分时，发现需要修改的调用点竟有1000多处，而且这些修改点很难借助于IDE或者重构工具来一次性解决，不但增加了团队的负担还直接导致了后续功能需求的交付延期。

和可演进性问题相比，高复杂度、霰弹式修改等代码级别问题也很重要，但是相对来说我们更加关注软件适应变化的能力，通过提升软件系统的适应性减少软件最终交付价值的前置时间，快速收集真实用户的反馈，持续不断迭代产品、完善设计

#### 明确清晰的责任定义优于松散无序的任务分配

有时候浮现式设计（Emergent Design）反而成了一种心理安慰的借口：“我知道这里有问题，但是我觉得这个变更需要通过需求来驱动”。

&#x20;服务责任人制度（Service Owner), 一个小组负责一个或者多个微服务，每个微服务只由一个小组负责，在分配特性功能、技术债务和线上问题时，需要把服务责任人制度作为首要遵循的原则。由于业务知识和技术上下文的相对集中，在解决具体的软件缺陷时不再浮于表面，团队成员可以更加深入地从需求、技术方案、软件架构等方面着手解决根本问题；针对线上问题通过清晰、明确的责任制度，倒逼团队在开发阶段主动关注软件的内建质量，谨慎判断是否引入技术债。

#### 主动预防优于被动响应

这个原则本质上是缩短反馈周期，提前发现潜在问题，除了必要的代码审查流程（Code Review）、提升团队能力之外还可以借助于自动化工具来提前发现问题。

在技术债治理的过程中，实践可以剪裁，甚至原则也可以妥协，因为比这几条原则更重要的是获得关键干系人的支持。作为技术人员或者技术领导者，不仅要有前瞻性的技术洞察力、锐意变革的魄力，还需要以“旁观者”视角，置身事外地观察自己所处的环境，思考技术改进究竟对于自己、他人、团队、公司和客户究竟产生了什么价值。

## 判断指标

```
新增Bug和修复Bug
可维护指数 = 已解决问题数量  / 总问题数量 (权重 = 优先级)
代码度量:圈复杂度、类耦合和继承深度
时间周期:从第一次提交代码到上线部署消耗的时间
代码搅动:特定代码被删除、替换或重写次数
代码覆盖率: 未被执行的代码行/总代码行数
代码所有制:  有多人懂团队其他人的代码
技术负债率(TDR) = 修复成本/开发成本
开发成本 = 构建产品或功能所需代码行数/每行代码执行时的计算资源消耗量
修复成本 <- 代码度量
```

常用工具：

Stepsize SonarQube Teamscale Velocity Jira



## 参考资料

Robert Nord 《The Future of Managing Technical Debt》&#x20;

Brent Barton 《Managing Software Debt: Building for Inevitable Change》&#x20;

Neal Ford、 Rebecca Parsons 《Building Evolutionary Architecture》&#x20;

杨政权 《技术债治理的四条原则》





