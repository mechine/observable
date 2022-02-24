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

timseries



sample

## 指标标签规范

**指标名：**

\[a-zA-Z\_:]\[a-zA-Z0-9\_:]\*

冒号保留用于用户定义的recording rules。 它们不应被exporter或直接仪表使用

**标签名：**

`[a-zA-Z_][a-zA-Z0-9_]*`。带有`_`下划线的标签名称被保留内部使用。

**标签值：**

标签labels值包含任意的Unicode码。

## 指标分类

counter

gauge

histogram

summary

untype

## 采集过程元数据

在Prometheus中任何被采集的目标Target被称为Instance，通常对应单个进程。 相同类型的Instance被称为Job

job

instance

target

relabel:&#x20;

对target信息中的一些维度进行转换变成指标中的维度，发生在采集前

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



