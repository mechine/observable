# 指标

### Counter 单调增

rate, irate, increase, and resets

//通过rate()函数获取HTTP请求量的增长率

rate(http\_requests\_total\[5m])

//查询当前系统中，访问量前10的HTTP地址

topk(10, http\_requests\_total)

### Gauge（仪表盘）

delta

predict\_linear

### Histogram（直方图）

\_bucket{le="<上边界>"} 这个值表示指标值小于等于上边界的所有样本数量。

\_sum 所有样本值的大小总和

\_count = \_bucket{le="+Inf"} 样本总数

histogram\_quantile

### Summary（摘要）

与 Histogram 类型类似，用于表示一段时间内的数据采样结果（通常是请求持续时间或响应大小等），但它直接存储了分位数（通过客户端计算，然后展示出来），而不是通过区间来计算。

{quantile="<φ>"}

\_sum

\_count

Histogram 需要通过 \_bucket 来计算分位数，而 Summary 则直接存储了分位数的值。



\= : 选择与提供的字符串完全相同的标签。&#x20;

!= : 选择与提供的字符串不相同的标签。&#x20;

\=\~ : 选择正则表达式与提供的字符串（或子字符串）相匹配的标签。

&#x20;!\~ : 选择正则表达式与提供的字符串（或子字符串）不匹配的标签。&#x20;

例如：选择指标名称为 http\_requests\_total，环境为 staging、testing 或 development，HTTP 方法为 GET 的时间序列：

```
http_requests_total{environment=~"staging|testing|development",method!="GET"}
```
