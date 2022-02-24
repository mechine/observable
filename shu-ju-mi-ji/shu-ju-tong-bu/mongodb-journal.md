---
description: oplog 是数据分阶段提交的最后一步，journal是整个事务
---

# mongodb-journal

### Journaling日志机制 <a href="#journaling-e6-97-a5-e5-bf-97-e6-9c-ba-e5-88-b6" id="journaling-e6-97-a5-e5-bf-97-e6-9c-ba-e5-88-b6"></a>

***

运行MongoDB如果开启了journaling日志功能，MongoDB先在内存保存写操作，并记录journaling日志到磁盘，然后才会把数据改变刷入到磁盘上的数据文件。为了保证journal日志文件的一致性，写日志是一个原子操作。本文将讨论MongoDB中journaling日志的实现机制。

### Journal日志文件 <a href="#journal-e6-97-a5-e5-bf-97-e6-96-87-e4-bb-b6" id="journal-e6-97-a5-e5-bf-97-e6-96-87-e4-bb-b6"></a>

如果开启了journal日志功能，MongoDB会在数据目录下创建一个`journal`文件夹，用来存放预写重放日志。同时这个目录也会有一个`last-sequence-number`文件。如果MongoDB安全关闭的话，会自动删除此目录下的所有文件，如果是崩溃导致的关闭，不会删除日志文件。在MongoDB进程重启的过程中，journal日志文件用于自动修复数据到一个一致性的状态。

journal日志文件是一种往文件尾不停追加内容的文件，它命名以`j._`开头，后面接一个数字（从0开始）作为序列号。如果文件超过1G大小，MongoDB会新建一个journal文件`j._1`。只要MongoDB把特定日志中的所有写操作刷入到磁盘数据文件，将会删除此日志文件。因为数据已经持久化，不再需要用它来重放恢复数据了。journal日志文件一般情况下只会生成两三个，除非你每秒有大量的写操作发生。

如果你需要的话，你可以使用`storage.smallFiles`参数来配置journal日志文件的大小。比如配置为`128M`。

### Journaling机制的存储视图 <a href="#journaling-e6-9c-ba-e5-88-b6-e7-9a-84-e5-ad-98-e5-82-a8-e8-a7-86-e5-9b-be" id="journaling-e6-9c-ba-e5-88-b6-e7-9a-84-e5-ad-98-e5-82-a8-e8-a7-86-e5-9b-be"></a>

Journaling功能用到了MongoDB存储层数据集内部的两个视图。

`shared`视图保存数据修改操作，用于刷入到磁盘数据文件。`shared`视图是MongoDB中唯一访问磁盘数据文件的视图。`mongod`进程请求操作系统把磁盘数据文件映射到虚拟内存的`shared`视图。操作系统只是映射数据与内存关系，并不马上加载数据到内存。当查询需要的时候，才会加载数据到内存，即按需加载。

`private`视图存储用于查询操作的数据。同时`private`视图也是MongoDB执行写操作的第一个地方。一旦journal日志提交完成，MongoDB会复制`private`视图中的改变到`shared`视图，再通过`shared`视图将数据刷入到磁盘数据文件。

`journal`视图是一个用来保证新的写操作的磁盘视图。当MongoDB在`private`视图执行完写操作后，在数据刷入磁盘之前，会先记录`journal`日志。`journal`日志保证了持久性。如果`mongod`实例在数据刷入磁盘之前崩溃，重启过程中`journal`日志会重放并写入`shared`视图，最终刷入磁盘持久化。

### Journaling如何纪录写操作 <a href="#journaling-e5-a6-82-e4-bd-95-e7-ba-aa-e5-bd-95-e5-86-99-e6-93-8d-e4-bd-9c" id="journaling-e5-a6-82-e4-bd-95-e7-ba-aa-e5-bd-95-e5-86-99-e6-93-8d-e4-bd-9c"></a>

MongoDB采用`group commits`方式将写操作批量复制到`journal`日志文件中。`group commits`提交方式能够最小化journal日志机制对性能的影响。因此`group commits`方式在提交过程中必须阻塞所有写入。`commitIntervalMs`参数可以用于配置日志提交的频率，默认是100ms。

Journaling存储以下原始操作：

* 文档插入或更新
* 索引修改
* 命名空间文件元数据的修改
* 创建和者删除数据库或关联的数据文件

当发生写操作，MongoDB首先写入数据到内存中的`private`视图，然后批量复制写操作到`journal`日志文件。写个`journal`日志实体来用描述写操作改变数据文件的哪些字节。

MongoDB接下来执行`journal`的写操作到`shared`视图。此时，`shared`视图与磁盘数据文件不一样。

默认每60s钟，MongoDB请求操作系统将`shared`视图刷入到磁盘。使数据文件更新到最新的写入状态。如果系统内存资源不足的时候，操作系统会选择以更高的频率刷入`shared`视图到磁盘。

MongoDB刷入数据文件完成后，会通知`journal`日志已经刷入。一旦`journal`日志文件只包含全部刷入的写操作，不再用于恢复，MongoDB会将它删除或者作为一个新的日志文件再次使用。

作为journaling机制的一部分，MongoDB会例行性请求操作系统重新将`shared`视图映射到`private`视图，为了节省物理内存。一旦发生重映射，操作系统能够识别到可以在`private`视图和`shared`视图共享的内存页映射。

### 配置项

storage.journal.enabled 决定是否开启journal

storage.journal.commitInternalMs 决定 journal 刷盘的间隔，默认为100ms

用户也可以通过写入时指定 writeConcern 为 {j: ture} 来每次写入时都确保 journal 刷盘。

默认情况下mongodb每100毫秒往journal文件中flush一次数据

数据文件和journal文件不在同一磁盘卷上时，默认刷新输出时间是30毫秒

值越低，刷新输出频率越高，数据安全度越高，但磁盘性能上的开销更高

### journal VS oplog

![](<../../.gitbook/assets/image (1) (1).png>)

![](<../../.gitbook/assets/image (1) (1) (1).png>)





