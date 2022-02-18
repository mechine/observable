# 浮动IP

简而言之： 两台机器，每台机器配置两个ip，一个私有ip各不相同，一个公用ip是相同的，设某台为机器为主节点，则默认可以通过公用ip访问主节点而不能访问备节点。如果备节点检测到主节点不可用，则启动备节点的公用ip，其备份作用。

***

A floating IP is usually a public, routable IP address that is not automatically assigned to an entity. Instead, a project owner assigns them to one or more entities temporarily. The respective entity has an automatically assigned, static IP for communication between instances in a private, non-routable network area, as well as via a manually assigned floating IP. This makes the entity’s services outside a cloud or network recognizable and therefore achievable.

一个浮动IP通常是一个公开的、可以路由到的IP地址，并且不会自动分配给实体设备。项目管理者临时分配动态IP到一个或者多个实体设备。这个实体设备有自动分配的静态IP用于内部网间设备的通讯。这个内部网使用私有地址，这些私有地址不能被路由到。通过浮动IP内网实体的服务才能被外网识别和访问。

In appropriately configured failover scenarios, an IP 'floats' to another active unit in the network so that it can take on the function of a dormant entity without a time delay, and can then answer incoming requests.

在一个配置好浮点IP的切换场景是，IP地址飘到网络中的另一台设备。新设备无延迟的接替当掉的设备，并对外提供服务。

How is a floating IP generated?

浮点IP是如何产生的？

Users obtain floating IPs for their projects from different pools that the system administrator configures and provides as server resources. As soon as a user receives a floating IP, they become the 'owner'. They can assign it to an entity, remove it, and then assign it to another at any time. Even if an entity is terminated, the user does not 'lose' the associated floating IP. It remains as a resource and can still be assigned to another entity when needed.

用户从系统管理员配置的资源池中为他们的项目获取IP地址。一旦用户获取一个浮动IP，就拥有了这个IP。他可以分配这个IP到一个计算实体，或者在任一时间移除分配给其他设备。就算设备关机，用户还拥有他属于他的浮动IP。浮动IP就像一种资源，当需要时可以分配给其他设备。

A major reason for using several parallel floating IP pools is that each pool can be operated by another internet service provider or can also be assigned by other external networks. This ensures that the connectivity or availability is maintainable even if an internet service provider should fail due to a malfunction.

使用多个平行的浮动IP主要是为了防止当其中的一个不可能用时使用其他地址以保证服务的正常可用。

When are floating IPs used?

什么时候会用浮动IP

Maximum availability is one of the key factors in every production environment. In the communication network, however, a single error can cause applications to fail. Developers do sleep better knowing that their applications are designed to withstand any conceivable error scenarios. The goal is to provide a highly available piece of infrastructure with minimal downtime.

最大的可用性是浮动IP在生产环境中使用的一个关键因素。在网络中，单个错误可能会导致应用的不可用。如果系统能成功应对任何可以想到的应用场景，开发人员就可以安枕无忧。浮动IP的目标就最小当机下提供高可用的基础设施。

A floating IP can serve as a flexible load balancing address, helping to balance peak loads by distributing incoming network traffic to different network nodes. Network nodes are devices which connect two (or more) transmission paths of a telecommunication network. As with a computer that distributes workflows across multiple processors, load balancing also handles large amounts of simultaneous requests or more complex calculations by splitting the load across multiple parallel systems.

浮动IP可以用于灵活的负载均衡地址，用于高峰时的负载均衡，分流访问流量到不同的网络节点。网络节点是连接到两个或者多个通讯网络。就像一台电脑分配工作流到不同的处理器，负载均衡大量并发的请求或者复杂的计算分配到并行系统中。

Failover and switchover

故障恢复和地址切换

If a primary load balancer or a central application server in a cluster fails on one side, a floating IP can be immediately assigned a redundant application server or a secondary load balancer in a correspondingly configured system. The IP 'floats' to the active unit, which immediately carries out the desired processes. An unplanned change between network services is referred to as 'failover'. This kind of protection is especially recommended for critical applications.

如果一个主要的负载均衡器或者集群中一个主要的业务服务器当掉，浮动IP立即被分配到冗余的应用器或者备用的负载均衡器，这些都需要提前配置好。当浮动IP飘到一个活动单元，活动单元立即承担相应的业务。故障恢复指的是非计划的网络服务切换。这种特别的保护推荐用于关键应用。

A planned change from a primary to a secondary system is referred to as a 'switchover'. The targeted transmission of services is not triggered by errors, but is usually controlled by a system administrator. A classic reason for a switchover is, for example, routine maintenance of the primary or secondary systems where a parallel instance temporarily takes over its function.

一个有计划的从主切换到从，通常被称为切换。切换不是由故障或者错误引起，而是系统管理员操作完成。切换的典型应用场景时，当对一个系统时行例常的维护时，由另一服务接替他的功能。

What advantages does a floating IP offer?

浮动IP优点

One of the main advantages of floating IPs is their flexibility – the free and needs-oriented assignability. Floating IPs are therefore suitable for use in both failover and switchover environments – for example, for performing upgrades of applications or entire sites with minimal downtime. While an upgrade is applied to one entity, another one takes on the traffic. Once the upgrade has been successfully completed, the traffic is redirected to the updated unit.

浮动IP的主要优点是灵活，自由的根据需要分配。浮动IP即适用于故障恢复又适用于服务切换。比如对某个应用或者整个站点的升级，并能保证对业务有最小的影响。当对一个应用升级时，另一个应用分配输入流量。一旦升级完成，流量会被重新导入到升级节点。

Another advantage: even if several or even many different entities are concealed behind a service being offered, the floating IP appears on the surface to users (who make use of the service) rather than the server’s IP that offers the respective service.

另一个优点是：浮动IP对外提供统一的IP，而不是实际对外提供服务的IP地址。
