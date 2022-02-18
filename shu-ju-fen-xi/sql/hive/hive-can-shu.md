# hive参数

### 功能类

#### 队列设置

#### 任务名设置

#### 打印头部

#### 打印汇总信息

#### 临时文件及日志

#### UDF设置

#### 动态分区

hive.exec.dynamic.partition查询结果是否动态分区

hive.exec.dynamic.partition.mode strict该模式下必须指定一个静态分区，nostrict该模式下不必指定静态分区，没有限制

hive.exec.max.dynamic.partitions.pernode在每一个mapper/reducer节点允许创建的最大分区数hive.exec.max.dynamic.partitions允许创建的最大分区数

#### 正则选择列

set hive.support.quoted.identifiers=none\`(inc\_day|inc\_month)?+.+\`

#### 整合hbase

### 综合类

#### 小文件的处理

**输入的小文件**

方式一：

set mapred.max.split.size=256000000

set mapred.min.split.size.per.node=256000000

set Mapred.min.split.size.per.rack=256000000

set hive.input.format=org.apache.hadoop.hive.ql.io.CombineHiveInputFormat

方式二：

设置hive参数，将额外启动一个MR Job打包小文件hive.merge.mapredfiles = false是否合并 Reduce 输出文件，默认为 Falsehive.merge.size.per.task = 256\*1000\*1000合并文件的大小

**输出的小文件**

set hive.merge.mapfiles=true是否合并Map输出文件：（默认值为真）set hive.merge.mapredfiles=false是否合并Reduce 端输出文件：（默认值为假）set hive.merge.size.per.task=256\*1000\*1000合并文件的大小：（默认值为 256000000）

#### JVM设置

**JVM重用**

set mapred.job.reuse.jvm.num.tasks=10缺点是，开启JVM重用将会一直占用使用到的task插槽，以便进行重用，直到任务完成后才能释放。如果某个“不平衡“的job中有几个reduce task 执行的时间要比其他reduce task消耗的时间多得多的话，那么保留的插槽就会一直空闲着却无法被其他的job使用，直到所有的task都结束了才会释放。

**内存设置**

内存种类：GC种类：

**mapper**

\`set mapreduce.map.memory.mb=4096set mapreduce.map.java.opts=-Xmx3600mset mapred.map.child.java.opts=-Xmx2g -XX:+UseConcMarkSweepGC

**reducer**

set mapreduce.reduce.memory.mb=1024set mapred.reduce.child.java.opts=-Xmx2g -XX:+UseConcMarkSweepGCset hive.exec.reducers.max=999

### 输入类

### 输出类

### 计算类

#### 并发度控制

**并行执行**

适用于union allset hive.exec.parallel=true开启任务并行执行set hive.exec.parallel.thread.number=8同一个sql允许并行任务的最大线程数

**mapper个数控制**

set mapred.max.split.size=1024000000\`set mapred.min.split.size.per.node=1024000000set mapred.min.split.size.per.rack=1024000000mapred.max.split.size <= mapred.min.split.size.per.node <= mapred.min.split.size.per.rack

**mapper个数计算**

**reduce个数控制**

set mapred.reduce.tasks=10set hive.exec.reducers.bytes.per.reducer=1073741824

**mapper,reducer个数过多或引起的问题**

#### JOIN类

**MapJoin**

set hive.auto.convert.join=true;set hive.mapjoin.smalltable.filesize=300000000;set hive.auto.convert.join.noconditionaltask=true;set hive.auto.convert.join.noconditionaltask.size=300000000;使用mapjoin时，会先执行一个本地任务(mapreduce local task)将小表转成hashtable并序列化为文件再压缩，随后这些hashtable文件会被上传到hadoop缓存，提供给各个map.oin使用。这里有三个参数我们需要注意：

> hive.mapjoin.localtask.max.memory.usage ： 将小表转成hashtable的本地任务的最大内存使用率,默认0.9hive.mapjoin.followby.gby.localtask.max.memory.usage ： 如果mapjoin后面紧跟着一个group by任务，这种情况下 本地任务的最大内存使用率，默认是0.55hive.mapjoin.check.memory.rows ： localtask每处理完多少行，就执行内存检查。默认为100000。如果我们的localtask的内存使用超过阀值，任务会直接失败。

### 预备知识

#### 流程步骤及组件名

**splitter**

**mapper**

**shuffle**

**partitioner**

set hive.mapred.partitioner

**sorter**

**combiner**

set min.num.spills.for.combine=0set hive.exec.combiner=falseCombiner的作用是把一个map产生的多个\<KEY,VALUE>合并成一个新的\<KEY,VALUE>,然后再将新\<KEY,VALUE>的作为reduce的输入。减少 reduce 端的数据量减少 shuffle 过程的数据量在 map 端做了一次合并，提高分布式计算程序的整体性能Combiner 组件帮 reduce 分担压力， 因此其业务逻辑和 reduce 中的业务逻辑相似。与mapper和reducer不同的是，combiner没有默认的实现，需要显式的设置在conf中才有作用。并不是所有情况下都能使用Combiner，Combiner适用于对记录汇总的场景（如求和），但是，求平均数的场景就不能使用Combiner了。如果可以使用Combiner，一般情况下，和我们的reduce函数是一致的。只有操作满足结合律的才可设置combiner。combine操作类似于：opt(opt(1, 2, 3), opt(4, 5, 6))如果opt为求和、求最大值的话，可以使用，但是如果是求中值的话，不适用。有或没有都不能影响业务逻辑，都不能影响最终结果

**reducer**

做规约操作

**counter**

略

### 其他

这部分主要不是hive的配置，是通过某些特殊的sql语法来达到优化的目的。
