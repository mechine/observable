# 指标梳理

### 监控指标梳理：

目的 ：容量？降级？可拓展性？ 源： 来自某监控系统或计算逻辑 分析： 结论：业务系统人员 闭环：指标后续跟踪

#### 指标的性质

指标的时间颗粒度越小（越实时，灵敏）：越精确，波动大，计算成本越高，用于定位与告警 指标的时间颗粒度越大（越离线，迟钝）：越平缓，计算成本低，用于评估与预警

指标的描述范围越大： 指标的描述范围越小：

指标越底层：理解需要越多专业知识 指标越偏业务：更容易理解

#### 容量日报的要点

1. 如果某天业务量突然急剧上涨，我们是否来得及处理？

> 根据历史走势：每天峰值时间发生在09:30-09:32 ，属于刚开盘。

能处理业务逐步增加的情况，无法处理业务暴增的情况

1. 重复或无用的指标

> 基础监控的指标异常的应该已经有告警体现，如果告警无体现，一个数字也会很难看出异常

1. 被遗忘的指标

> 错误率，网络丢包率，网络时延

1. 指标的展现

> 能用图形就别用数字，能用趋势就不用单点

1. 不同监控系统的比较

> 监控系统的范围实际上有较大的重叠部分，这部分数据接入大数据之后可以做交叉验证。 可以梳理当前所有的监控系统

1. 专人负责专项更利于闭环也可更专注。（指标的改进/预测等等）

> 系统健壮性（系统内部单点分析） 可拓展性 业务服务质量（错误率，响应时间） 基础服务质量（网络抖动，丢包率，jvm gc时间等） 监控系统评估 准确性，实时性 成本管理等等