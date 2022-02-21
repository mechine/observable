# Elasticsearch

| 关系型数据库      | Elasticsearch                      |
| ----------- | ---------------------------------- |
| 数据库Database | 索引Index，支持全文检索                     |
| 表Table      | 类型Type                             |
| 数据行Row      | 文档Document，但不需要固定结构，不同文档可以具有不同字段集合 |
| 数据列Column   | 字段Field                            |
| 模式Schema    | 映像Mapping                          |

#### 索引(Index)

ElasticSearch把数据存放到一个或者多个索引(indices)中。如果用关系型数据库模型对比，索引(index)的地位与数据库实例(database)相当。索引存放和读取的基本单元是文档(Document)。我们也一再强调，ElasticSearch内部用Apache Lucene实现索引中数据的读写。读者应该清楚的是：在ElasticSearch中被视为单独的一个索引(index)，在Lucene中可能不止一个。这是因为在分布式体系中，ElasticSearch会用到分片(shards)和备份(replicas)机制将一个索引(index)存储多份。

#### 文档(Document)

在ElasticSearch的世界中，文档(Document)是主要的存在实体(在Lucene中也是如此)。所有的ElasticSearch应用需求到最后都可以统一建模成一个检索模型：检索相关文档。文档(Document)由一个或者多个域(Field)组成，每个域(Field)由一个域名(此域名非彼域名)和一个或者多个值组成(有多个值的值称为多值域(multi-valued))。在ElasticSeach中，每个文档(Document)都可能会有不同的域(Field)集合；也就是说文档(Document)是没有固定的模式和统一的结构。文档(Document)之间保持结构的相似性即可(Lucene中的文档(Document)也秉持着相同的规定)。实际上，ElasticSearch中的文档(Document)就是Lucene中的文档(Document)。从客户端的角度来看，文档(Document)就是一个JSON对象(关于JSON格式的相关信息,请参看hhtp://en.wikipedia.org/wiki/JSON)。

#### 参数映射(Mapping)

在  1.1节 认识Apache Lucene  中已经提到，所有的文档(Document)在存储之前都必须经过分析(analyze)流程。用户可以配置输入文本分解成Token的方式；哪些Token应该被过滤掉；或者其它的的处理流程，比如去除HTML标签。此外，ElasticSearch提供的各种特性，比如排序的相关信息。保存上述的配置信息，这就是参数映射(Mapping)在ElasticSearch中扮演的角色。尽管ElasticSearch可以根据域的值自动识别域的类型(field type)，在生产应用中，都是需要自己配置这些信息以避免一些奇的问题发生。要保证应用的可控性。

#### 文档类型(Type)

每个文档在ElasticSearch中都必须设定它的类型。文档类型使得同一个索引中在存储结构不同文档时，只需要依据文档类型就可以找到对应的参数映射(Mapping)信息，方便文档的存取。

#### 节点(Node)

单独一个ElasticSearch服务器实例称为一个节点。对于许多应用场景来说，部署一个单节点的ElasticSearch服务器就足够了。但是考虑到容错性和数据过载，配置多节点的ElasticSearch集群是明智的选择。

#### 集群(Cluster)

集群是多个ElasticSearch节点的集合。这些节点齐心协力应对单个节点无法处理的搜索需求和数据存储需求。集群同时也是应对由于部分机器(节点)运行中断或者升级导致无法提供服务这一问题的利器。ElasticSearch提供的集群各个节点几乎是无缝连接(所谓无缝连接，即集群对外而言是一个整体，增加一个节点或者去掉一个节点对用户而言是透明的<个人理解，仅供参考>)。在ElasticSearch中配置一个集群非常简单，在我们看来，这是在与同类产品中竞争所体现出的最大优势。

#### 分片索引(Shard)

前面已经提到，集群能够存储超出单机容量的信息。为了实现这种需求，ElasticSearch把数据分发到多个存储Lucene索引的物理机上。这些Lucene索引称为分片索引，这个分发的过程称为索引分片(Sharding)。在ElasticSearch集群中，索引分片(Sharding)是自动完成的，而且所有分片索引(Shard)是作为一个整体呈现给用户的。需要注意的是，尽管索引分片这个过程是自动的，但是在应用中需要事先调整好参数。因为集群中分片的数量需要在索引创建前配置好，而且服务器启动后是无法修改的，至少目前无法修改。

#### 索引副本(Replica)

通过索引分片机制(Sharding)可以向ElasticSearch集群中导入超过单机容量的数据，客户端操作任意一个节点即可实现对集群数据的读写操作。当集群负载增长，用户搜索请求阻塞在单个节点上时，通过索引副本(Replica)机制就可以解决这个问题。索引副本(Replica)机制的的思路很简单：为索引分片创建一份新的拷贝，它可以像原来的主分片一样处理用户搜索请求。同时也顺便保证了数据的安全性。即如果主分片数据丢失，ElasticSearch通过索引副本使得数据不丢失。索引副本可以随时添加或者删除，所以用户可以在需要的时候动态调整其数量。
