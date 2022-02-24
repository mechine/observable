# 概念

## 数据类型

**瞬时向量**（Instant vector） :

**区间向量**（Range vector）：

&#x20;时间范围通过时间范围选择器 \[] 进行定义。&#x20;

s m h d w y&#x20;

sum(http\_requests\_total{method="GET"}\[5m] offset 5m)&#x20;

位移 offset

&#x20;sum(http\_requests\_total{method="GET"} offset 5m)

**标量**（Scalar） - 一个浮点型的数据值

**字符串**（String） - 一个简单的字符串值

\---&#x20;

**timseries (一个指标一段时间的值)**

streams of timestamped values belonging to the same metric and the same set of labeled dimensions.

**sample（一个指标某一瞬间的值）**

Samples form the actual time series data. Each sample consists of:

* a float64 value
* a millisecond-precision timestamp

## 指标标签规范

**指标名：**

\[a-zA-Z\_:]\[a-zA-Z0-9\_:]\*

冒号保留用于用户定义的recording rules。 它们不应被exporter或直接仪表使用

**标签名：**

`[a-zA-Z_][a-zA-Z0-9_]*`。带有`_`下划线的标签名称被保留内部使用。

**标签值：**

标签labels值包含任意的Unicode码。

## 指标分类

### counter

Counter可以简单理解为计数器，是个比较简单但又常用的类型。适用于生成请求次数、错误次数等指标。

### gauge

`Gauge`是一个用来记录实时值的指标，常用于表示CPU使用率、内存使用率、线程数等指标。

`gauge`类型指标最常见的就是用来标识服务是否存活的`up`指标了，这个指标在大多的exporter上都会有，属于一个可以建通用警报规则的指标。

### summary

* 根据 Exporter 提供的分位点，样本会被计算后拆分成多行数据，每行使用标签”quantile”区分，”quantile”的值包括 Exporter 提供的所有分位点。
* 数据的排列顺序必须是按照标签”quantile”值递增；
* 提供两行数据分别表示该监控指标所有样本的和、样本数量，命名格式为：`<监控指标名称>_sum`、`<监控指标名称>_count`

```
# HELP go_gc_duration_seconds A summary of the pause duration of garbage collection cycles.
# TYPE go_gc_duration_seconds summary
go_gc_duration_seconds{quantile="0"} 3.9794e-05
go_gc_duration_seconds{quantile="0.25"} 0.000119721
go_gc_duration_seconds{quantile="0.5"} 0.00024831
go_gc_duration_seconds{quantile="0.75"} 0.000588213
go_gc_duration_seconds{quantile="1"} 0.001718785
go_gc_duration_seconds_sum 0.007368653
go_gc_duration_seconds_count 17
```

### histogram &#x20;

* 根据 Exporter 提供的 Bucket 值，样本会被计算后拆分成多行数据，每行使用标签”le”区分，”le”为 Exporter 提供的 Buckets；
* 数据的排列顺序必须是按照标签”le”值递增；
* 必须要有一行数据的标签 le=”+Inf”，值为该监控指标的样本总数

```
# HELP prom_req_duration_secs Histogram of latencies for HTTP requests.
# TYPE prom_req_duration_secs histogram
prom_req_duration_secs_bucket{handler="/api/v1/label/",le="0.1"} 1
prom_req_duration_secs_bucket{handler="/api/v1/label/",le="0.2"} 1
prom_req_duration_secs_bucket{handler="/api/v1/label/",le="0.4"} 1
prom_req_duration_secs_bucket{handler="/api/v1/label/",le="1"} 1
prom_req_duration_secs_bucket{handler="/api/v1/label/",le="3"} 1
prom_req_duration_secs_bucket{handler="/api/v1/label/",le="8"} 1
prom_req_duration_secs_bucket{handler="/api/v1/label/",le="20"} 1
prom_req_duration_secs_bucket{handler="/api/v1/label/",le="60"} 1
prom_req_duration_secs_bucket{handler="/api/v1/label/",le="120"} 1
prom_req_duration_secs_bucket{handler="/api/v1/label/",le="+Inf"} 1
prom_req_duration_secs_sum{handler="/api/v1/label/"} 0.001304263
prom_req_duration_secs_count{handler="/api/v1/label/"} 1
```

### untype

## Retrieval 数据源

* Retrieval：中定义了Prometheus Server需要从哪些地方拉取数据
  * Jobs / Exporters：Prometheus可以从Jobs或Exporters中拉取监控数据。Exporter以Web API的形式对外暴露数据采集接口。
  * Prometheus Server：Prometheus还可以从其他的Prometheus Server中拉取数据
  * Pushgateway：对于一些以临时性Job运行的组件，Prometheus可能还没有来得及从中pull监控数据的情况下，这些Job已经结束了，Job运行时可以在运行时将监控数据推送到Pushgateway中，Prometheus从Pushgateway中拉取数据，防止监控数据丢失
  * Service：是指Prometheus可以动态的发现一些服务，拉取数据进行监控，如从DNS，Kubernetes，Consul中发现





## 采集过程元数据

在Prometheus中任何被采集的目标Target被称为Instance，通常对应单个进程。 相同类型的Instance被称为Job

relabel: 对target信息中的一些维度进行转换变成指标中的维度，发生在采集前,比如截取address中的ip或者port单独作为一个tag

```
## target 自带的属性
__address__：当前Target实例的访问地址<host>:<port>
__scheme__：采集目标服务访问地址的HTTP Scheme，HTTP或者HTTPS
__metrics_path__：采集目标服务访问地址的访问路径
__param_<name>：采集任务目标服务的中包含的请求参数
##consul自动发现时常带的元数据信息
__meta_consul_address：consul地址
__meta_consul_dc：consul服务所在的数据中心
__meta_consulmetadata：服务的metadata
__meta_consul_node：consul服务node节点的信息
__meta_consul_service_address：服务访问地址
__meta_consul_service_id：服务ID
__meta_consul_service_port：服务端口
__meta_consul_service：服务名称
__meta_consul_tags：服务包含的标签信息
```



## 采集器

Exporter是Prometheus的一类数据采集组件的总称。它负责从目标处搜集数据，并将其转化为Prometheus支持的格式。与传统的数据采集组件不同的是，它并不向中央服务器发送数据，而是等待中央服务器主动前来抓取，默认的抓取地址为[http://CURRENT\_IP:9100/metrics](http://current\_ip:9100/metrics)

node-exporter用于采集服务器层面的运行指标，包括机器的loadavg、filesystem、meminfo等基础监控，类似于传统主机监控维度的zabbix-agent

node-export由prometheus官方提供、维护，不会捆绑安装，但基本上是必备的exporter

exporter会生成一个**UP**指标，记录job下有多少个活跃instance

Exporter 通过 HTTP 接口以文本形式向 Prometheus 暴露样本数据，格式简单，没有嵌套，可读性强。每个监控指标对应的数据文本格式如下：

```
# HELP <监控指标名称> <监控指标描述>
# TYPE <监控指标名称> <监控指标类型>
<监控指标名称>{ <标签名称>=<标签值>,<标签名称>=<标签值>...} <样本值1> <时间戳>
<监控指标名称>{ <标签名称>=<标签值>,<标签名称>=<标签值>...} <样本值2> <时间戳>
...
```



* 以 # 开头的行，如果后面跟着”HELP”，Prometheus 将这行解析为监控指标的描述，通常用于描述监控数据的来源
* 以 # 开头的行，如果后面跟着”TYPE”，Prometheus 将这行解析为监控指标的类型，支持的类型有：Counter、Gauge、Histogram、Summary、Untyped。Prometheus 在存储数据时是不区分数据类型的，所以当你在犹豫一个数据类型应该用 Counter 或 Gauge 时，可以试试 Untype
* 以 # 开头的行，如果后面没有跟着”HELP”或”TYPE”，则 Prometheus 将这行视为注释，解析时忽略
* 如果一个监控指标有多条样本数据，那么每条样本数据的标签值组合应该是唯一的
* 每行数据对应一条样本数据
* 时间戳应为采集数据的时间，是可选项，如果 Exporter 没有提供时间戳的话，Prometheus Server 会在拉取到样本数据时将时间戳设置为当前时间
* Summary 和 Histogram 类型的监控指标要求提供两行数据分别表示该监控指标所有样本的和、样本数量，命名格式为：`<监控指标名称>_sum`、`<监控指标名称>_count`

## PushGateway

[Pushgateway](https://github.com/prometheus/pushgateway) 是 Prometheus 生态中一个重要工具，使用它的原因主要是：

* Prometheus 采用 pull 模式，可能由于不在一个子网或者防火墙原因，导致 Prometheus 无法直接拉取各个 target 数据。
* 在监控业务数据的时候，需要将不同数据汇总, 由 Prometheus 统一收集。

由于以上原因，不得不使用 pushgateway，但在使用之前，有必要了解一下它的一些弊端：

* 将多个节点数据汇总到 pushgateway, 如果 pushgateway 挂了，受影响比多个 target 大。
* Prometheus 拉取状态 `up` 只针对 pushgateway, 无法做到对每个节点有效。
* Pushgateway 可以持久化推送给它的所有监控数据。

因此，即使你的监控已经下线，prometheus 还会拉取到旧的监控数据，需要手动清理 pushgateway 不要的数据。

## AlertManager

Prometheus监控系统中，采集与警报是分离的。警报规则在 **Prometheus** 定义，警报规则触发以后，才会将信息转发到给独立的组件**Alertmanager** ，经过 **Alertmanager** r对警报的信息处理后，最终通过接收器发送给指定用户。

### 分组&#x20;

Grouping 是 Alertmanager 把同类型的警报进行分组，合并多条警报到一个通知中&#x20;

### 抑制&#x20;

Inhibition 是 当某条警报已经发送，停止重复发送由此警报引发的其他异常或故障的警报机制。&#x20;

### 静默&#x20;

Silences 提供了一个简单的机制，根据标签快速对警报进行静默处理；对传进来的警报进行匹配检查，如果接受到警报符合静默的配置，Alertmanager 则不会发送警报通知。

