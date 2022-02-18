# 缓存



分布式缓存：本地缓存：缓存失效：容量、时\[]间、热度高并发：锁缓存回写–回源雪崩监听器缓存的分类XX缓存的应用场景XX缓存的不适用场景XX缓存的特性XX缓存的实现

> 缓存应用场景CPU缓存操作系统缓存HTTP缓存数据库缓存静态文件缓存本地缓存分布式缓存

#### 缓存加载

CacheLoaderCallable –如果有缓存则返回；否则运算、缓存、然后返回显式插入

**CDN缓存回源**

当 cdn 缓存服务器中没有符合客户端要求的资源的时候，缓存服务器会请求上一级缓存服务器，以此类推，直到获取到。

#### 缓存刷新

#### 缓存回收

基于容量回收定时回收expireAfterAccess(long, TimeUnit)expireAfterWrite(long, TimeUnit)CacheBuilder.ticker(Ticker) 时间源基于引用回收CacheBuilder.weakKeys():使用弱引用存储键。CacheBuilder.weakValues():使用弱引用存储值。CacheBuilder.softValues():使用软引用存储值。显式清除个别清除:Cache.invalidate(key)批量清除:Cache.invalidateAll(keys)清除所有缓存项:Cache.invalidateAll()

***

[使用缓存的9大误区-上](https://kb.cnblogs.com/page/138696/)[使用缓存的9大误区-下](https://kb.cnblogs.com/page/144396/)

> 使用缓存的9大误区太过于依赖默认的序列化机制缓存大对象使用缓存机制在线程间进行数据的共享认为调用缓存API之后，数据会被立刻缓存起来缓存大量的数据集合，而读取其中一部分缓存大量具有图结构的对象导致内存浪费缓存应用程序的配置信息使用很多不同的键指向相同的缓存项没有及时的更新或者删除再缓存中已经过期或者失效的数据

[浅谈分布式缓存那些事儿](http://os.51cto.com/art/201306/397999.htm)

> 典型应用场景1) 页面缓存.用来缓存Web 页面的内容片段,包括HTML、CSS 和图片等,多应用于社交网站等;2) 应用对象缓存.缓存系统作为ORM 框架的二级缓存对外提供服务,目的是减轻数据库的负载压力,加速应用访问;3) 状态缓存.缓存包括Session 会话状态及应用横向扩展时的状态数据等,这类数据一般是难以恢复的,对可用性要求较高,多应用于高可用集群;4) 并行处理.通常涉及大量中间计算结果需要共享;5) 事件处理.分布式缓存提供了针对事件流的连续查询(continuous query)处理技术,满足实时性需求;6) 极限事务处理.分布式缓存为事务型应用提供高吞吐率、低延时的解决方案,支持高并发事务请求处理,多应用于铁路、金融服务和电信等领域.分布式缓存与NoSQL分布式缓存与极限事务处理

[缓存穿透，缓存击穿，缓存雪崩解决方案分析](https://blog.csdn.net/zeb\_perfect/article/details/54135506)[缓存穿透、缓存并发、缓存失效之思路变迁](http://ifeve.com/concurrency-cache-cross/)[缓存更新的套路](https://coolshell.cn/articles/17416.html)

> 缓存更新Cache aside- 失效：应用程序先从cache取数据，没有得到，则从数据库中取数据，成功后，放到缓存中。- 命中：应用程序从cache中取数据，取到后返回。- 更新：先把数据存到数据库中，成功后，再让缓存失效。Read throughRead Through 套路就是在查询操作中更新缓存，也就是说，当缓存失效的时候（过期或LRU换出），Cache Aside是由调用方负责把数据加载入缓存，而Read Through则用缓存服务自己来加载，从而对应用方是透明的。Write throughWrite Through 套路和Read Through相仿，不过是在更新数据时发生。当有数据更新的时候，如果没有命中缓存，直接更新数据库，然后返回。如果命中了缓存，则更新缓存，然后再由Cache自己更新数据库（这是一个同步操作）Write behind cachingWrite Behind 又叫 Write Back。 Write Back套路，一句说就是，在更新数据的时候，只更新缓存，不更新数据库，而我们的缓存会异步地批量更新数据库。
