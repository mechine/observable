# 存储

### 远程读写

Prometheus 配置文件中指定 Remote Write（远程写）的 URL 地址，一旦设置了该配置项，Prometheus 将采集到的样本数据通过 HTTP 的形式发送给适配器（Adaptor）

Promthues 的 Remote Read（远程读）也通过了一个适配器实现。在远程读的流程当中，当用户发起查询请求后，Promthues 将向 remote\_read 中配置的 URL 发起查询请求（matchers,ranges），`Adaptor` 根据请求条件从第三方存储服务中获取响应的数据。同时将数据转换为 Promthues 的原始样本数据返回给 Prometheus Server。

当获取到样本数据后，Promthues 在本地使用 PromQL 对样本数据进行二次处理。

> 启用远程读设置后，Prometheus 仅从远程存储读取一组时序样本数据（根据标签选择器和时间范围），对于规则文件的处理，以及 Metadata API 的处理都只基于 Prometheus 本地存储完成。这也就意味着远程读在扩展性上有一定的限制，因为所有的样本数据都要首先加载到 Prometheus Server，然后再进行处理。所以 Prometheus 暂时不支持完全分布式处理。

### 本地存储

Prometheus 按照两个小时为一个时间窗口，将两小时内产生的数据存储在一个块（Block）中。每个块都是一个单独的目录，里面含该时间窗口内的所有样本数据（chunks），元数据文件（meta.json）以及索引文件（index）。其中索引文件会将指标名称和标签索引到样板数据的时间序列中。此期间如果通过 API 删除时间序列，删除记录会保存在单独的逻辑文件 `tombstone` 当中。

当前样本数据所在的块会被直接保存在内存中，不会持久化到磁盘中。为了确保 Prometheus 发生崩溃或重启时能够恢复数据，Prometheus 启动时会通过预写日志（write-ahead-log(WAL)）重新记录，从而恢复数据。预写日志文件保存在 `wal` 目录中，每个文件大小为 `128MB`。wal 文件包括还没有被压缩的原始数据，所以比常规的块文件大得多。一般情况下，Prometheus 会保留三个 wal 文件，但如果有些高负载服务器需要保存两个小时以上的原始数据，wal 文件的数量就会大于 3 个。



```bash
./data 
   |- 01BKGV7JBM69T2G1BGBGM6KB12 # 块
      |- meta.json  # 元数据
      |- wal        # 写入日志
        |- 000002
        |- 000001
   |- 01BKGTZQ1SYQJTR4PB43C8PD98  # 块
      |- meta.json  #元数据
      |- index   # 索引文件
      |- chunks  # 样本数据
        |- 000001
      |- tombstones # 逻辑数据
   |- 01BKGTZQ1HHWHV8FBJXW1Y3W0K
      |- meta.json
      |- wal
        |-000001
```



容量规划：

> needed\_disk\_space =
>
> &#x20;retention\_time\_seconds \* ingested\_samples\_per\_second \* bytes\_per\_sample
