# 可观测性

### Elastic 可观测性

Elastic 可观测性通过监测、发现和针对基本的 SLI 和 SLO 发送告警，能够自动完成事件响应生命周期。该解决方案涵盖监测的四个领域：Uptime、APM、Metrics和 Logs。Uptime 通过向服务终端发送外部心跳信息来监测可用性。APM 通过直接在应用程序内测量和采集事件来监测延迟和质量。Metrics 通过衡量基础设施的资源利用率来监测饱和度。Logs 通过从系统和服务采集消息来监测正确性。



* **标题** -_“发生了什么事件”_
* **严重程度** -_“这一事件的优先级如何？”_
* **阐释** -_“这一事件会造成什么业务影响？”_
* **观测到的状态** -_“发生什么了？”_
* **理想状态** -_“应该发生什么？”_
* **上下文** -_“环境的状态如何？”_使用告警中的数据来描述[时间](https://www.elastic.co/guide/en/ecs/current/ecs-base.html)、[云](https://www.elastic.co/guide/en/ecs/current/ecs-cloud.html)、[网络](https://www.elastic.co/guide/en/ecs/current/ecs-network.html)、[操作系统](https://www.elastic.co/guide/en/ecs/current/ecs-os.html)、[容器](https://www.elastic.co/guide/en/ecs/current/ecs-container.html)、[进程](https://www.elastic.co/guide/en/ecs/current/ecs-process.html)以及有关事件的其他上下文。
* **链接** -_“接下来我要查看什么？”_使用告警中的数据来创建链接，从而将相应人员带到仪表板、错误报告或其他有用的目标位置。



### 可观测性数据模型 (metric logging tracing profiling)

https://mirror.xyz/0xFd007bb46C47D8600C139E34Df9DfceC86F0B319/hw60dfH7YMtM3jd5dT22spTpPGSS7T8yxskkddTXXro

Metric、Logging、Tracing 和 Profiling

Profiling：一般叫做 Continuous Profiling，持续分析，它反映的是程序内部的运行状态，比如栈调用、执行时间等。可以把 Profiling 可视化成火焰图方面分析问题。

一般来说，基于这些度量处理故障的流程是：

Metric → Tracing → Logging → Profiling

在 Tracing 领域，之前有两个项目，一个是 OpenTracing，它是一个规范，Jaeger 就是基于OpenTracing 的开源实现，另一个是 OpenCensus，它是 Google 开源的度量工具。这两个项目功能高度重合，在 CNCF 主导下合并成了 OpenTelemetry，而 OpenTracing 和 OpenCensus 也不再维护。

**LabelSet + Timestamp + Number**

**LabelSet + Timestamp + String**

**Operation Name + Start / End Timestamp + Attributes + Events + Parent + SpanContext**

1. Operation Name：操作名
2. Start / End Timestamp：开始和结束时间
3.  Attributes：KV 对，包括 Status（比如 OK、Cancelled、Permission Denied）、

    SpanKind（CLIENT、SERVER、PRODUCER、CONSUMER、INTERNAL 等）、自定义信息等
4. Events：若干个元组列表，每个元组包括 timestamp、name、Attributes，用于记录一系列重要事件
5. Parent 包含父亲的 Span ID、Trace ID
6. SpanContext 包含自身的 Span ID、Trace ID

**LabelSet + Timestamp + \[]byte**



第一，Metric 和 Tracing 融合。

这里要用到 Exemplar，Exemplar 最早被用在 Google 的 StackDriver 中，后面成为了 OpenMetrics 标准的一部分，在应用通过标准 /metrics 端口暴露 Metric 时，Exemplar 信息也会被一起暴露。

第二，Logging 和 Tracing 融合。

只要使用带有 Tracing 库的 SDK，每个请求都会带上 Trace ID，并把这些 ID 打在日志中。

通过 Trace ID 可以定位到一个唯一的 Tracing， 跳转到 Tracing 系统的 UI 进行查询。

第三，Metric 和 Profiling 融合。

基于 Exemplar，把 Profiling ID 也放入 Exemplar 中，Prometheus 支持存储和查询即可。

至于展示，可以在 Grafana 上开发一个 pprof 的 Panel 插件，这样可以展示 Profiling。



当前业内主要有三个 Prometheus 集群方案，分别是 Prometheus Operator、Thanos 和 Cortex。

Prometheus Operator 是基于 k8s Operator 机制实现的一套集群方案，它启动两个 Prometheus，同时采集指标，以保证高可用。

但是它有两个致命的缺点：

1. 采集能力不可扩展，Prometheus 单机采集能力有限，在大规模集群下，比如数万台，单机采集有瓶颈
2. 存储不可扩展，数据保存在本地，无法保存长期的数据

对于 Prometheus Operator 的采集和存储不可扩展的两个问题，Thanos 和 Cortex 都解决了，只是解决的方案有所不同。

对于采集的扩展性，Thanos 使用了多个 Prometheus 集群 （每个集群是两个 Prometheus replica） 去采集，意味着需要做额外的配置管理，而 Cortex 使用了一个 Scraper 集群（基于 Prometheus 裁剪 ）去采集，对任务自动做 Sharding 以解决采集能力的可扩展性问题。

对于存储的扩展性，Thanos 和 Cortex 都使用对象存储，这一点是一样的。

![](.gitbook/assets/1644395092\(1\).png)



对于写入流来说：

1. Distributor 会基于 Ingester 实例构建一个一致性哈希环，基于二分查找找到对应的 Ingester 实例写入，每个 Ingester 实例会持有有若干个 token，以便写入相对均衡。为了数据可靠性，Distributor 会写多个 Ingester 实例。
2. Ingester 是一个半有状态服务，它内部有 Cache，相当于热数据，它会定期把数据同步到对象存储（冷数据）。
3. Compactor 是一个压缩程序，因为 Distributor 会写多个 Ingester 实例，所以对象存储中有数据冗余，它负责合并掉冗余数据（包括索引）。另外，对于 Metric，它还会把多个 Block（默认 2h）合并成更长时间的 Block，以便节省资源和提升查询效率。

对于查询流来说：

1. Query-FrontEnd 是接收查询请求的前端，它内部有队列的设计，防止太多请求把服务打挂，而且把请求做分割，然后合并每个分割的结果，另外还会把数据存入 Result Cache，以便下次提升查询结果。
2. Querier 是具体查询的组件，它会访问 Ingester 和 Store-Gateway，Ingester 负责查询热数据，Store-Gateway 负责查询冷数据（Logging、Tracing、Profiling 当前没有 Store-Gateway）。
3. Store-Gateway 负责从对象存储中读取数据，它会通过 HTTP 206 的方式请求部分内容，并且它内部会保存部分的索引数据，并通过 Index Cache 提升查询效率。
4. Ruler 是计算程序，比如 Metric 的聚合计算和告警计算、Logging 基于日志生成指标的计算或告警的计算 等等。





### 可观测性与存储（metrics logging tracing events）

**Metrics**

Metrics，通常是数值类型的时间序列数据。这类需求的存在如此广泛，以至于衍生了专门服务于这个目标的数据库子类，时间序列数据库，TSDB。

TSDB 经历了大约如下几个方面的重要演进

* 数据模型。描述信息从 metric naming 中剥离出来，形成 tag。现代的 tsdb 通常都已采用 tag 化的数据模型。
* 数据类型。从简单的数值记录，到为不同场景衍生出 gauge, counter, timer 等等更多的数据类型
* 索引结构。索引结构跟数据模型密切相关，在 tag 为主的现代 tsdb, 倒排索引已经是主流索引结构。
* 数据存储。从 rrdtool 写环形队列到文件的时代，到 OpenTSDB 这样自行编解码写入底层数据库，再到 Facebook 提出的时序数据压缩算法，通常会是若干种技术的综合使用，并且针对不同的数据类型采用不同方案

Metrics 存储，或者是 TSDB 的研究和演进，我们会有另外的文章专门介绍。

**logging**

logging 通常会是工程师定位生产环境问题最直接的手段。日志的处理大约在如下几个方面进行演进

* 集中存储/检索。使得工程师免于分别登陆机器查看日志之苦，日志被统一采集，集中存储于日志服务，并提供统一的检索服务。这个过程牵扯到例如日志格式统一，解析，结构化等等问题。
* 日志的监控。
* 原文中的关键字，例如 error, fatal 大概率意味着值得关注的错误产生
* 从日志中提取的 metrics，例如 access log 中携带的大量数据，都可以被提取成有用的信息。至于提取的手段，有的通过客户端在日志本地进行解析，有的在集中存储过程中进行解析。

**tracing**

随着互联网工程日渐复杂，尤其是微服务的风潮下，分布式 tracing 通常是理解系统，定位系统故障的最重要手段。

在存储层面，tracing 已经有相对明确的方案，无论是 OpenZipkin，还是 CNCF 的 Jaeger ，都提供几乎开箱即用的后端软件，其中当然包括存储。

Tracing 的存储设计主要考虑

1\. 稀疏数据：tracing 数据通常是稀疏的，这通常有几个原因

* 不同业务的 trace 路径通常不同，也就是 span 不同，因而稀疏
* 同种业务的 trace ，在不同内外部条件下，路径也不同。例如访问数据库，是否命中缓存，都会产生不同的 span 链
* 访问正常/异常的 trace ，产生不同 span

2\. 多维度查询：通常的解决思路

* 二级索引：在以 HBase, Cassandra 为基础的方案中比较常见
* 引入倒排索引，在二级索引方案无法满足全部查询请求时，可能会引入 Elasticsearch 辅助索引，提升查询灵活性

**Events**

同样是一个难以定义，但是很容易描述的术语。我们把，一次部署，一次配置变更，一次dns 切换，诸如此类的变更，称为事件。

它们通常意味着生产环境的变更。而故障，通常因为不恰当的变更引起。

对 events 的处理主要包括

* 集中存储：事件种类很多，较难归纳共同的查询纬度，所以倒排索引在这种无法事先确定查询纬度的场景下，是非常合适的存储结构
* Dashboard：以恰当的方式，把事件查询 /展示出来。上文提到 Etsy 的博客中，展示了很好的实践方法，使得工程师能够通过 dashboard ，非常轻松确认网站登陆失败，与登录模块部署事件之间的关系。































