# 用户日活月活统计

Redis 的 HashMap，BitMap 和 HyperLogLog Elasticsearch 目前支持两种近似算法，分别是 cardinality 和 percentiles TDigest 是一个简单，快速，精确度高，可并行化的近似百分位算法，被 ElastichSearch、Spark 和 Kylin 等系统使用。TDigest 主要有两种实现算法，一种是 buffer-and-merge 算法，一种是 AV L树的聚类算法。
