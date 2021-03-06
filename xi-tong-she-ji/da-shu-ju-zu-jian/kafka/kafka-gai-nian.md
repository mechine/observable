# kafka概念

#### message

#### offset

#### zero-copy

传统的读取文件数据并发送到网络的步骤如下：（1）操作系统将数据从磁盘文件中读取到内核空间的页面缓存；（2）应用程序将数据从内核空间读入用户空间缓冲区；（3）应用程序将读到数据写回内核空间并放入socket缓冲区；（4）操作系统将数据从socket缓冲区复制到网卡接口，此时数据才能通过网络发送。有四次拷贝，两次系统调用零拷贝技术”只用将磁盘文件的数据复制到页面缓存中一次，然后将数据从页面缓存直接发送到网络中（发送给不同的订阅者时，都可以使用同一个页面缓存），避免了重复复制操作。如果有10个消费者，传统方式下，数据复制次数为4\*10=40次，而使用“零拷贝技术”只需要1+10=11次，一次为从磁盘复制到页面缓存，10次表示10个消费者各自读取一次页面缓存。

#### 服务端(broker)

#### 生产者(producer)

#### 消费者(consumer)

**消费组**

consumer group是kafka提供的可扩展且具有容错性的消费者机制。既然是一个组，那么组内必然可以有多个消费者或消费者实例(consumer instance)，它们共享一个公共的ID，即group ID。组内的所有消费者协调在一起来消费订阅主题(subscribed topics)的所有分区(partition)。当然，每个分区只能由同一个消费组内的一个consumer来消费。1）consumer group下可以有一个或多个consumer instance，consumer instance可以是一个进程，也可以是一个线程2）group.id是一个字符串，唯一标识一个consumer group3）consumer group下订阅的topic下的每个分区只能分配给某个group下的一个consumer(当然该分区还可以被分配给其他group)

**消费者组变更：**

新增消费者消费者下线消费者rebalance过多消费者的影响
