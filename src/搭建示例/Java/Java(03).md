##### <font size="4" color="red">01. 微服务架构含义</font>

​		微服务类似于`SOA`架构，但是比`SOA`架构粒度更细，更轻量。

​		`SOA`基于`WebService`和`ESP`实现，底层基于`HTTP`协议和使用`XML`方式传输，`XML`在网络传输过程中会产生大量冗余。微服务由`SOA`架构演变而来，继承了`SOA`架构的优点，同时对`SOA`架构缺点进行改善，数据传输采用`JSON`格式，相比于`XML`更轻量和快捷，粒度更细，更加便于敏捷开发。`SOA`数据库会存在共享，微服务提倡每个服务连接独立的数据库。

***

##### <font size="4" color="red">02. Springcloud含义</font>

​		`SpringCloud`是微服务的一种解决方案，依赖`SpringBoot`实现。包含注册中心(`eureka`)、客户端负载均衡(`Ribbon`)、网关(`zuul`)、分布式锁、分布式会话等。

***

##### <font size="4" color="red">03. Springcloud支持哪些注册中心</font>

​		`Eureka`、`Consul`、`Zookeeper`现在`Eureka`闭源了，可以通过什么注册中心替代`Eureka`呢？`Consul`或`Zookeeper`

***

##### <font size="4" color="red">04. Springcloud微服务治理的思想</font>

​		`Eureka`如何实现高可用启动多台`Eureka`服务器，然后作为`SpringCloud`服务互相注册，客户端从`Eureka`集群获取信息时，按照注册的`Eureka`顺序对第一个`Eureka`进行访问。

`@LoadBalanced`注解的作用：开启客户端负载均衡。

***

##### <font size="4" color="red">05. Nginx与Ribbon的区别</font>

​		`Nginx`是反向代理同时可以实现负载均衡，`nginx`拦截客户端请求采用负载均衡策略根据`upstream`配置进行转发，相当于请求通过`nginx`服务器进行转发。`Ribbon`是客户端负载均衡，从注册中心读取目标服务器信息，然后客户端采用轮询策略对服务直接访问，全程在客户端操作。

***

##### <font size="4" color="red">06. Ribbon底层实现原理</font>

​		`Ribbon`使用`discoveryClient`从注册中心读取目标服务信息，对同一接口请求进行计数，使用%取余算法获取目标服务集群索引，返回获取到的目标服务信息。

***

##### <font size="4" color="red">07. SpringCloud有几种调用接口方式</font>

使用`Feign`和`RestTemplate`

***

##### <font size="4" color="red">08. DiscoveryClient的作用</font>

​		可以从注册中心中根据服务别名获取注册的服务器信息。

***

##### <font size="4" color="red">09. 雪崩效应</font>

​		雪崩效应是在大型互联网项目中，当某个服务发生宕机时，调用这个服务的其他服务也会发生宕机，大型项目的微服务之间的调用是互通的，这样就会将服务的不可用逐步扩大到各个其他服务中，从而使整个项目的服务宕机崩溃.发生雪崩效应的原因有以下几点

```
1.单个服务的代码存在bug
2.请求访问量激增导致服务发生崩溃(如大型商城的枪红包，秒杀功能)
3.服务器的硬件故障也会导致部分服务不可用.
```

***

##### <font size="4" color="red">10. 微服务中如何保护服务</font>

​		一般使用使用`Hystrix`框架，实现服务隔离来避免出现服务的雪崩效应，从而达到保护服务的效果。当微服务中，高并发的数据库访问量导致服务线程阻塞，使单个服务宕机，服务的不可用会蔓延到其他服务，引起整体服务灾难性后果，使用服务降级能有效为不同的服务分配资源,一旦服务不可用则返回友好提示，不占用其他服务资源，从而避免单个服务崩溃引发整体服务的不可用.

***

##### <font size="4" color="red">11. Hystrix服务保护的原理</font>

​		通过服务降级、服务熔断、服务隔离为高并发服务提供保护。
**服务降级：**当客户端请求服务器端的时候，防止客户端一直等待，不会处理业务逻辑代码，直接返回一个友好的提示给客户端。
**服务熔断：**是在服务降级的基础上更直接的一种保护方式，当在一个统计时间范围内的请求失败数量达到设定值（`requestVolumeThreshold`）或当前的请求错误率达到设定的错误率阈值（`errorThresholdPercentage`）时开启断路，之后的请求直接走fallback方法，在设定时间（`sleepWindowInMilliseconds`）后尝试恢复。
**服务隔离：**就是`Hystrix`为隔离的服务开启一个独立的线程池，这样在高并发的情况下不会影响其他服务。服务隔离有线程池和信号量两种实现方式，一般使用线程池方式。
服务降级底层是如何实现的？
`Hystrix`实现服务降级的功能是通过重写`HystrixCommand`中的`getFallback()`方法，当`Hystrix`的`run`方法或`construct`执行发生错误时转而执行`getFallback()`方法。

***

##### <font size="4" color="red">12. 分布式配置中心框架</font>

```
Apollo(阿波罗)、zookeeper、springcloud config
springcloud config实时刷新采用SpringCloud Bus消息总线
```

***

##### <font size="4" color="red">13. 网关含义</font>

​		网关相当于一个网络服务架构的入口，所有网络请求必须通过网关转发到具体的服务。

***

##### <font size="4" color="red">14. 网关作用</font>

​		统一管理微服务请求，权限控制、负载均衡、路由转发、监控、安全控制黑名单和白名单等

***

##### <font size="4" color="red">15. 网关和过滤器的区别</font>

​		网关是对所有服务的请求进行分析过滤，过滤器是对单个服务而言。

***

##### <font size="4" color="red">16. 常用网关框架</font>

```
Nginx、Zuul、Gateway
```

***

##### <font size="4" color="red">17. Zuul与Nginx有什么区别</font>

​		`Zuul`是`java`语言实现的，主要为`java`服务提供网关服务，尤其在微服务架构中可以更加灵活的对网关进行操作。`Nginx`是使用`C`语言实现，性能高于`Zuul`，但是实现自定义操作需要熟悉`lua`语言，对程序员要求较高，可以使用`Nginx`做`Zuul`集群。

***

##### <font size="4" color="red">18. Nginx可以实现网关为什么还需要使用Zuul框架</font>

​		`Zuul`是`SpringCloud`集成的网关，使用`Java`语言编写，可以对`SpringCloud`架构提供更灵活的服务。

***

##### <font size="4" color="red">19. Nginx可以实现网关为什么还需要使用Zuul框架</font>

​		考虑到`API`接口的分类可以将`API`接口分为开发`API`接口和内网`API`接口，内网`API`接口用于局域网，为内部服务器提供服务。开放`API`接口用于对外部合作单位提供接口调用，需要遵循`Oauth2.0`权限认证协议。同时还需要考虑安全性、幂等性等。

***

##### <font size="4" color="red">20. ZuulFilter常用有那些方法</font>

```
Run()：过滤器的具体业务逻辑
shouldFilter()：判断过滤器是否有效
filterOrder()：过滤器执行顺序
filterType()：过滤器拦截位置
```

***

##### <font size="4" color="red">21. 动态Zuul网关路由转发</font>

​		通过`path`配置拦截请求，通过`ServiceId`到配置中心获取转发的服务列表，`Zuul`内部使用`Ribbon`实现本地负载均衡和转发。

***

##### <font size="4" color="red">22. Zuul网关如何搭建集群</font>

​		使用`Nginx`的`upstream`设置`Zuul`服务集群，通过`location`拦截请求并转发到`upstream`，默认使用轮询机制对`Zuul`集群发送请求。 

***

##### <font size="4" color="red">23. 为什么要用Springboot</font>

​		快速开发，快速整合，配置简化、内嵌服务容器

***

##### <font size="4" color="red">24. 为什么MySQL索引要用 B+tree</font>

**1.为什么不选二叉树**

​		假设此时用普通二叉树记录 id 索引列，我们在每插入一行记录的同时还要维护二叉树索引字段。此时`id=7`的那条数据时，它的查找过程和我们全表扫描也没什么很大区别。显而易见，二叉树对于这种依次递增的数据列其实是不适合作为索引的数据结构。

**2.为什么不采用Hash表**

​		`Hash`表：一个快速搜索的数据结构，搜索的时间复杂度`O(1)`，`Hash`函数：将一个任意类型的`key`，可以转换成一个`int`类型的下标，这时候开始查找 ，`id = 7`的树节点仅找了1次，效率非常高了。但`MySQL`的索引依然不采用能够精准定位的Hash 表。因为它不适用于范围查询。

**3.为什么不采用红黑树**

​		红黑树是一种特化的`AVL`树（平衡二叉树），都是在进行插入和删除操作时通过特定操作保持二叉查找树的平衡；若一棵二叉查找树是红黑树，则它的任一子树必为红黑树。

​		假设此时用红黑树记录`id`索引列，我们在每插入一行记录的同时还要维护红黑树索引字段。插入过程中会发现它与普通二叉树不同的是当一棵树的左右子树高度差 > 1 时，它会进行自旋操作，保持树的平衡。这时候开始查找`id = 7`的树节点只找了3次，比所谓的普通二叉树还是要更快的。

​		但`MySQL`的索引依然不采用能够精确定位和范围查询都优秀的红黑树。因为当`MySQL`数据量很大的时候，索引的体积也会很大，可能内存放不下，所以需要从磁盘上进行相关读写，如果树的层级太高，则读写磁盘的次数（`I/O`交互）就会越多，性能就会越差。

**4.`B-tree`树**

​		红黑树目前的唯一不足点就是树的高度不可控，所以现在我们的切入点就是树的高度。目前一个节点是只分配了一个存储 1 个元素，如果要控制高度，我们就可以把一个节点分配的空间更大一点，让它横向存储多个元素，这个时候高度就可控了。这么个改造过程，就变成了`B-tree`。

```
B-tree 是一颗绝对平衡的多路树。它的结构中还有两个概念
度（Degree）：一个节点拥有的子节点（子树）的数量。（有的地方是以度来说明 B-tree 的，这里解释一下）
阶（order）：一个节点的子节点的最大个数。（通常用 m 表示）
```

(1) 首先查找节点，由于 B-tree 通常是在磁盘上存储的所以这步需要进行磁盘IO操作；

(2) 查找关键字，当找到某个节点后将该节点读入内存中然后通过顺序或者折半查找来查找关键字。若没有找到关键字，则需要判断大小来找到合适的分支继续查找。

***

##### <font size="4" color="red">25. Seata分布式事务实现原理</font>

`Seata`是阿里开源的分布式事务解决方案中间件，对业务侵入小，在应用中Seata整体事务逻辑基于两阶段提交的模型，

**核心概念包含三个角色：**

`TM`：事务发起者。用来告诉`TC`全局事务的开始，提交，回滚。

`RM`：事务资源，每一个`RM`都会作为一个分支事务注册在`TC`。

`TC`：事务协调者，即独立运行的`seata-server`，用于接收事务注册，提交和回滚。

**实现步骤：**

**`AT(Auto Transaction)`模式：**

​		`Seata`里称为`DataSource Resource`。业务通过`JDBC`标准接口访问数据库资源时，`Seata`框架会对所有请求进行拦截，做一些操作。每个本地事务提交时，`Seata RM（Resource Manager，资源管理器）` 都会向 `TC（Transaction Coordinator，事务协调器） `注册一个分支事务。当请求链路调用完成后，发起方通知`TC`提交或回滚分布式事务，进入二阶段调用流程。此时，`TC`会根据之前注册的分支事务回调到对应参与者去执行对应资源的第二阶段。`TC`是怎么找到分支事务与资源的对应关系呢？每个资源都有一个全局唯一的资源`ID`，并且在初始化时用该`ID`向`TC`注册资源。在运行时，每个分支事务的注册都会带上其资源`ID`。这样`TC`就能在二阶段调用时正确找到对应的资源。

(1) `TM`向`TC` 申请开启一个全局事务，全局事务创建并生成一个全局唯一的`XID`。

(2) `XID`在微服务调用链路的上下文中传播。

(3) `RM`向`TC`注册分支事务，将其纳入`XID`对应全局事务的管辖。

(4) `TM`向`TC`发起针对`XID`的全局提交或回滚决议。

(5) `TC`调度`XID`下管辖的全部分支事务完成提交或回滚请求。

**`TCC`模式：**

`TCC`与`Seata AT`事务一样都是两阶段事务，它与`AT`事务的主要区别为：

```
(1) TCC对业务代码侵入严重，每个阶段的数据操作都要自己进行编码来实现，事务框架无法自动处理。
(2) TCC效率更高
(3) 不必对数据加全局锁，允许多个事务同时操作数据。
```

(1) 一阶段`prepare`行为：调用自定义的`prepare`逻辑。

(2) 二阶段`commit`行为：调用自定义的`commit`逻辑。

(3) 二阶段`rollback`行为：调用自定义的`rollback`逻辑。

**`MT(Manual Transaction)`模式：**

这个模式适合其他的场景，因为底层存储可能没有事务支持，需要自己实现`prepare`、`commit`和`rollback`的逻辑

***

##### <font size="4" color="red">26. Jta Atomiks分布式事务实现原理</font>

![](./img/img(03)/26-01.jpg)

**`JTA(Java Transaction Manager)` : 是`Java`规范,是`XA`在`Java`上的实现**

```
(1) TransactionManager : 这个TransactionManager可以通过管理多个ResourceManager来管理多个Resouce,也就是管理多个数据源
(2) XAResource : 针对数据资源封装的一个接口
(3) 两段式提交 : 多数据源事务提交的机制
```

​		主要的原理是两阶段提交,以上面的请求业务为例,当整个业务完成了之后只是第一阶段提交,在第二阶段提交之前会检查其他所有事务是否已经提交,如果前面出现了错误或是没有提交,那么第二阶段就不会提交,而是直接`rollback`操作,这样所有的事务都会做`Rollback`操作.

**1.`JTA`架构**

```
(1) 事务管理器(Transaction Manager)
(2) 一个或多个支持 XA 协议的资源管理器(Resource Manager)
```

​		`Java`事务编程接口`（JTA：Java Transaction API）`和`Java`事务服务` (JTS；Java Transaction Service) `为`J2EE`平台提供了分布式事务服务。分布式事务`（Distributed Transaction）`包括事务管理器`（Transaction Manager）`和一个或多个支持`XA`协议的资源管理器` ( Resource Manager )`。

​		`JTA`事务有效的屏蔽了底层事务资源，使应用可以以透明的方式参入到事务处理中；但是与本地事务相比，`XA`协议的系统开销大，在系统开发过程中应慎重考虑是否确实需要分布式事务。若确实需要分布式事务以协调多个事务资源，则应实现和配置所支持`XA`协议的事务管理器，如`JMS`、`JDBC`数据库连接池等。

**2.特点**

​		`JTA`的有点就是能够支持多数据库事务同时事务管理,满足分布式系统中的数据的一致性.但是也有对应的弊端:

```
(1) 两阶段提交
(2) 事务时间太长,锁数据太长
(3) 低性能,低吞吐量
(4) 同步阻塞，并发效率差
(5) 分布式架构曾在瓶颈
```

根据所面向对象的不同，我们可以将 JTA的事务管理器和资源管理器理解为两个方面：

```
(1) 面向开发人员的使用接口(事务管理器)
(2) 面向服务提供商的实现接口(资源管理器)
```

 		开发人员使用开发人员接口，实现应用程序对全局事务的支持；各提供商（数据库，`JMS`等）依据提供商接口的规范提供事务资源管理功能；事务管理器`（ TransactionManager ）`将应用对分布式事务的使用映射到实际的事务资源并在事务资源间进行协调与控制。 下面，本文将对包括`UserTransaction`、 `Transaction` 和 `TransactionManager `在内的三个主要接口以及其定义的方法进行介绍。

**UserTransaction 和 Transaction**

​		面向开发人员的接口为`UserTransaction `（使用方法如上例所示），开发人员通常只使用此接口实现`JTA`事务管理，其定义了如下的方法：

```
(1) begin()- 开始一个分布式事务，（在后台 TransactionManager 会创建一个 Transaction 事务对象并把此对象通过 * ThreadLocale 关联到当前线程上 )
(2) commit()- 提交事务（在后台 TransactionManager 会从当前线程下取出事务对象并把此对象所代表的事务提交）
(3) rollback()- 回滚事务（在后台 TransactionManager 会从当前线程下取出事务对象并把此对象所代表的事务回滚）
(4) getStatus()- 返回关联到当前线程的分布式事务的状态 (Status 对象里边定义了所有的事务状态，感兴趣的读者可以参考 API 文档 )
(5) setRollbackOnly()- 标识关联到当前线程的分布式事务将被回滚
```

面向提供商的实现接口主要涉及到`TransactionManager`和`Transaction`两个对象；

​			`Transaction`代表了一个物理意义上的事务，在开发人员调用`UserTransaction.begin()`方法时`TransactionManager`会创建一个`Transaction `事务对象（标志着事务的开始）并把此对象通过`ThreadLocale`关联到当前线程。`UserTransaction`接口中的`commit()`、`rollback()`，`getStatus() `等方法都将最终委托给`Transaction`类的对应方法执行。

***

##### <font size="4" color="red">27. 漏桶算法和令牌桶算法</font>

**漏桶算法：**假设有一个水桶，水桶有一定的容量，所有请求不论速度都会注入到水桶中，然后水桶以一个恒定的速度向外将请求放出，当水桶满了的时候，新的请求被丢弃。

**优点：**可以平滑请求，削减峰值。

**缺点：**瓶颈会在漏出的速度，可能会拖慢整个系统，且不能有效地利用系统的资源。 

**令牌桶算法：**有一个令牌桶，单位时间内令牌会以恒定的数量（即令牌的加入速度）加入到令牌桶中，所有请求都需要获取令牌才可正常访问。当令牌桶中没有令牌可取的时候，则拒绝请求。

**优点：**相比漏桶算法，令牌桶算法允许一定的突发流量，但是又不会让突发流量超过我们给定的限制（单位时间窗口内的令牌数）。即限制了我们所说的`QPS`(每秒查询率)。

​		漏桶算法能够强行限制数据的传输速率。l令牌桶算法能够在限制数据的平均传输速率的同时还允许某种程度的突发传输。需要说明的是：在某些情况下，漏桶算法不能够有效地使用网络资源。因为漏桶的漏出速率是固定的，所以即使网络中没有发生拥塞，漏桶算法也不能使某一个单独的数据流达到端口速率。因此，漏桶算法对于存在突发特性的流量来说缺乏效率。而令牌桶算法则能够满足这些具有突发特性的流量。通常，漏桶算法与令牌桶算法结合起来为网络流量提供更高效的控制。

***

##### <font color="red" size="4">28. MySql中事务控制和锁定</font>

​		`MyISAM`和`MEMORY`存储引擎支持`表级锁定(table-level locking)`，InnoDB 存储引擎支持`行级锁定(row-level locking)`，`BDB`存储引擎支持`页级锁定(page-level locking)`

页级锁：销和加锁时间界于表锁和行锁之间；会出现死锁；锁定粒度界于表锁和行锁之间，并发度一般

**表级锁：**表级锁是对整张表进行加锁，`MyISAM`和`MEMORY`主要支持表级锁，表级锁加锁快，不会出现死锁，锁的粒度比较粗，并发度最低

**行级锁：**行级锁可以说是 MySQL 中粒度最细的一种锁了，`InnoDB`支持行级锁，行级锁容易发生死锁，并发度比较好，同时锁的开销也比较大。

`MySQL`默认情况下支持表级锁定和行级锁定。但是在某些情况下需要手动控制事务以确保整个事务的完整性

***

##### <font color="red" size="4">29. Springboot启动方式</font>

​		主类`@SpringBootApplication`注解或添加`@ComponentScan`和`@EnableAutoConfiguration`注解，使用`@SpringBootApplication`时注意自动扫描当前包

***

##### <font color="red" size="4">30. Springboot如何实现异步执行</font>

​		在启动类添加`@EnableAsync`表示开启对异步任务的支持，在异步服务上添加`@Async`

***

##### <font color="red" size="4">31. Springboot性能优化</font>

​		如果项目比较大，类比较多，不使用`@SpringBootApplication`，采用`@Compoment`指定扫包范围在项目启动时设置`JVM`初始内存和最大内存相同
将springboot内置服务器由`tomcat`设置为`undertow`

***

##### <font color="red" size="4">32. Springboot执行流程</font>

​		使用`maven`父子包依赖关系加载相关jar包，使用`java`操作`Spring`的初始化过程生成`class`文件，然后用`java`创建`tomcat`服务器加载这些`class`文件

***

##### <font color="red" size="4">33. Zookeeper分布式锁和Redis分布式锁区别</font>

**`Zookeeper`分布式锁：**即使获取不到锁，创建对锁的监听即可，不需要不断去尝试获取锁，性能开销小`Zookeeper`实现分布式锁，创建的是临时节点，客户端挂了，节点自然删除，也就达到了自动释放锁的效果

**`Redis`分布式锁：**如果客户端获取到锁的时候遇到`bug`或挂了，还需要等到超时时间过了以后才能重新获取锁

***

##### <font color="red" size="4">34. 如何保证JMS可靠消息</font>

​		`ActiveMQ`采用消息签收机制保证数据的可靠性，消息签收有三种方式：自动签收、手动签收、事务，默认自动签收。如果是带事务的消息，事务执行完毕后自动签收，事务回滚则重新发送。

***

##### <font color="red" size="4">35. ActiveMQ服务端宕机了消息处理</font>

​		生产者可以通过`setDeliveryMode`方法设置消息模式，当设置未非持久化时服务器宕机后消息将销毁，重启服务器后无法继续消费。当设置为持久化时服务器宕机后消息将保存到服务器中，重启后消费者还可以继续消费未处理完毕的消息。

***

##### <font color="red" size="4">36. RabbitMQ与其他MQ区别</font>

​		`RabbitMQ`是用`erlang`语言实现，安装`RabbitMQ`之前需要先安装`erlang`环境。`RabbitMQ`只支持`AMQP`协议，用于对稳定性要求比较高的企业。

***

##### <font color="red" size="4">37. Http协议</font>

```
Http协议是基于TCP协议封装成超文本传输协议，包括请求(request）和响应(response),http协议请求（request）分为请求参数（request params）和方法类型(request method）、请求头（request hearder）、请求体(request body) ，
响应(response)分为 响应状态(response state）、响应头（response header）、响应体(response body）等
```

***

##### <font color="red" size="4">38. Redis淘汰机制</font>

**1.淘汰机制**

`Redis`缓存淘汰策略`6+2`种

```
1.不淘汰数据
(1) noeviction ，不进行数据淘汰，当缓存被写满后，Redis不提供服务直接返回错误。
2.在设置过期时间的键值对中
(1) volatile-random ，在设置过期时间的键值对中随机删除
(2) volatile-ttl ，在设置过期时间的键值对，基于过期时间的先后进行删除，越早过期的越先被删除。
(3) volatile-lru ， 基于LRU(Least Recently Used) 算法筛选设置了过期时间的键值对， 最近最少使用的原则来筛选数据
(4) volatile-lfu ，使用 LFU( Least Frequently Used ) 算法选择设置了过期时间的键值对, 使用频率最少的键值对,来筛选数据。
3.所有键值队中
(1) allkeys-random， 从所有键值对中随机选择并删除数据
(2) allkeys-lru， 使用 LRU 算法在所有数据中进行筛选
(3) allkeys-lfu， 使用 LFU 算法在所有数据中进行筛选
```

> **注：**`LRU`维护一个双向链表 ，链表的头和尾分别表示`MRU`端和`LRU`端，分别代表最近最常使用的数据和最近最不常用的数据。`LRU`算法在实际实现时，需要用链表管理所有的缓存数据，这会带来额外的空间开销。而且，当有数据被访问时，需要在链表上把该数据移动到`MRU`端，如果有大量数据被访问，就会带来很多链表移动操作，会很耗时，进而会降低`Redis`缓存性能。

***

#####  <font color="red" size="4">39. 为什么使用消息队列</font>

**1.解耦**

![](./img/img(03)/39-01.png)

​		如果使用`MQ`，`A`系统产生一条数据，发送到`MQ`里面去，哪个系统需要数据自己去`MQ`里面消费。如果新系统需要数据，直接从`MQ`里消费即可；如果某个系统不需要这条数据了，就取消对`MQ`消息的消费即可。这样下来，`A`系统压根儿不需要去考虑要给谁发送数据，不需要维护这个代码，也不需要考虑人家是否调用成功、失败超时等情况。

**2.异步**

![](./img/img(03)/39-02.png)

​		一般互联网类的企业，对于用户直接的操作，一般要求是每个请求都必须在`200`ms 以内完成，对用户几乎是无感知的。

**3.消峰**

![](./img/img(03)/39-03.png)

​		如果使用`MQ`，每秒`5k`个请求写入`MQ`，`A`系统每秒钟最多处理`2k`个请求，因为 MySQL 每秒钟最多处理`2k`个。A 系统从`MQ`中慢慢拉取请求，每秒钟就拉取`2k `个请求，不要超过自己每秒能处理的最大请求数量就完成，这样下来，哪怕是高峰期的时候，`A`系统也绝对不会挂掉。

***

##### <font color="red" size="4">40. Kafka、ActiveMQ、RabbitMQ、RocketMQ 有什么优缺点</font>

| 特性                     | ActiveMQ                              | RabbitMQ                                           | RocketMQ                                                     | Kafka                                                        |
| ------------------------ | ------------------------------------- | -------------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 单机吞吐量               | 万级，比 RocketMQ、Kafka 低一个数量级 | 同 ActiveMQ                                        | 10 万级，支撑高吞吐                                          | 10 万级，高吞吐，一般配合大数据类的系统来进行实时数据计算、日志采集等场景 |
| topic 数量对吞吐量的影响 |                                       |                                                    | topic 可以达到几百/几千的级别，吞吐量会有较小幅度的下降，这是 RocketMQ 的一大优势，在同等机器下，可以支撑大量的 topic | topic 从几十到几百个时候，吞吐量会大幅度下降，在同等机器下，Kafka 尽量保证 topic 数量不要过多，如果要支撑大规模的 topic，需要增加更多的机器资源 |
| 时效性                   | ms 级                                 | 微秒级，这是 RabbitMQ 的一大特点，延迟最低         | ms 级                                                        | 延迟在 ms 级以内                                             |
| 可用性                   | 高，基于主从架构实现高可用            | 同 ActiveMQ                                        | 非常高，分布式架构                                           | 非常高，分布式，一个数据多个副本，少数机器宕机，不会丢失数据，不会导致不可用 |
| 消息可靠性               | 有较低的概率丢失数据                  | 基本不丢                                           | 经过参数优化配置，可以做到 0 丢失                            | 同 RocketMQ                                                  |
| 功能支持                 | MQ 领域的功能极其完备                 | 基于 erlang 开发，并发能力很强，性能极好，延时很低 | MQ 功能较为完善，还是分布式的，扩展性好                      | 功能较为简单，主要支持简单的 MQ 功能，在大数据领域的实时计算以及日志采集被大规模使用 |

***

##### <font color="red" size="4">41. ElasticSearch集群架构设计</font>

```
ElasticSearch生产集群我们部署了5台机器，每台机器是6核64G的，集群总内存是320G。ElasticSearch集群的日增量数据大概是2000万条，每天日增量数据大概是500MB，每月增量数据大概是6亿，15G。目前系统已经运行了几个月，现在ElasticSearch集群里数据总量大概是100G左右。目前线上有5个索引（这个结合你们自己业务来，看看自己有哪些数据可以放ElasticSearch的），每个索引的数据量大概是 20G，所以这个数据量之内，我们每个索引分配的是8个shard，比默认的5个 shard多了3个shard。
```

***

##### <font color="red" size="4">41. 缓存用途</font>

1.高性能，对于一些需要复杂操作耗时查出来的结果，且确定后面不怎么变化，但是有很多读请求，那么直接将查询出来的结果放在缓存中

2.高并发，缓存是走内存的，内存天然支撑高并发。

***

##### <font color="red" size="4">42. Redis和Memcached区别</font>

```
(1) Redis比Memcached支持更丰富的数据操作
(2) 原生支持集群模式
(3) 由于 Redis只使用单核，而Memcached可以使用多核，所以平均每一个核上Redis在存储小数据时比Memcached性能更高。而在100k以上的数据中，Memcached 性能要高于Redis。虽然Redis最近也在存储大数据的性能上进行优化，但是比起Memcached，还是稍有逊色。
```

***

##### <font color="red" size="4">43. Redis单线程模型效率高原因</font>

```
(1) 纯内存操作
(2) 核心是基于非阻塞的IO多路复用机制
(3) C语言实现，一般来说，C语言实现的程序“距离”操作系统更近，执行速度相对会更快。
(4) 单线程反而避免了多线程的频繁上下文切换问题，预防了多线程可能产生的竞争问题。
```

***

##### <font color="red" size="4">44. Redis6.0引入多线程</font>

​		因为读写网络的`Read/Write`系统调用在`Redis`执行期间占用了大部分`CPU`时间，如果把网络读写做成多线程的方式对性能会有很大提升。Redis 的多线程部分只是用来处理网络数据的读写和协议解析，执行命令仍然是单线程。之所以这么设计是不想`Redis`因为多线程而变得复杂，需要去控制`key`、`lua`、事务、`LPUSH/LPOP`等等的并发问题。

***

##### <font color="red" size="4">45. Redis中过期策略</font>

(1) 定期删除：指的是`Redis`默认是每隔`100ms`就随机抽取一些设置了过期时间的`key`，检查其是否过期，如果过期就删除。

(2) 内存淘汰机制：

````
noeviction: 当内存不足以容纳新写入数据时，新写入操作会报错
allkeys-lru：当内存不足以容纳新写入数据时，在键空间中，移除最近最少使用的 key（这个是最常用的）。
allkeys-random：当内存不足以容纳新写入数据时，在键空间中，随机移除某个 key，这个一般没人用吧，为啥要随机，肯定是把最近最少使用的 key 给干掉啊。
volatile-lru：当内存不足以容纳新写入数据时，在设置了过期时间的键空间中，移除最近最少使用的 key（这个一般不太合适）。
volatile-random：当内存不足以容纳新写入数据时，在设置了过期时间的键空间中，随机移除某个 key。
volatile-ttl：当内存不足以容纳新写入数据时，在设置了过期时间的键空间中，有更早过期时间的 key 优先移除。
````

`LRU`算法实现：

```java
class LRUCache<K, V> extends LinkedHashMap<K, V> {
    private final int CACHE_SIZE;

    /**
     * 传递进来最多能缓存多少数据
     *
     * @param cacheSize 缓存大小
     */
    public LRUCache(int cacheSize) {
        // true 表示让 linkedHashMap 按照访问顺序来进行排序，最近访问的放在头部，最老访问的放在尾部。
        super((int) Math.ceil(cacheSize / 0.75) + 1, 0.75f, true);
        CACHE_SIZE = cacheSize;
    }

    /**
     * 钩子方法，通过put新增键值对的时候，若该方法返回true
     * 便移除该map中最老的键和值
     */
    @Override
    protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
        // 当 map中的数据量大于指定的缓存个数的时候，就自动删除最老的数据。
        return size() > CACHE_SIZE;
    }
}
```

***

##### <font color="red" size="4">46. Redis中如何实现高可用</font>

​		`redis`实现高并发主要依靠主从架构，一主多从，一般来说，很多项目其实就足够了，单主用来写入数据，单机几万`QPS`，多从用来查询数据，多个从实例可以提供每秒`10w`的`QPS`。

​		如果想要在实现高并发的同时，容纳大量的数据，那么就需要`redis`集群，使用`redis`集群之后，可以提供每秒几十万的读写并发。`redis`高可用，如果是做主从架构部署，那么加上哨兵就可以了，就可以实现，任何一个实例宕机，可以进行主备切换。

(1) 主从架构

(2) 哨兵模式

***

##### <font size="4" color="red">47. 大量Url找出相同Url</font>

​		给定`a`、`b`两个文件，各存放`50`亿个`URL`，每个`URL`各占`64B`，内存限制是`4G`。请找出`a`、`b`两个文件共同的`URL`。

**分析：**由于内存大小只有`4G`，因此，我们不可能一次性把所有`URL`加载到内存中处理。对于这种类型的题目，一般采用分治策略，即：把一个文件中的`URL`按照某个特征划分为多个小文件，使得每个小文件大小不超过`4G`，这样就可以把这个小文件读到内存中进行处理了。

**解决方案：**

(1) 分治策略

```
    首先遍历文件 a，对遍历到的 URL 求 hash(URL) % 1000 ，根据计算结果把遍历到的 URL 存储到 a0, a1, a2, ..., a999，这样每个大小约为 300MB。使用同样的方法遍历文件 b，把文件b中的 URL 分别存储到文件 b0, b1, b2, ..., b999 中。这样处理过后，所有可能相同的URL都在对应的小文件中，即 a0 对应 b0, ..., a999 对应 b999，不对应的小文件不可能有相同的URL。那么接下来，我们只需要求出这1000对小文件中相同的URL就好了。

    接着遍历ai( i∈[0,999] )，把URL存储到一个 HashSet 集合中。然后遍历bi中每个URL，看在HashSet集合中是否存在，若存在，说明这就是共同的URL，可以把这个URL保存到一个单独的文件中。
```

(2) 前缀树

​		一般而言，`URL`的长度差距不会不大，而且前面几个字符，绝大部分相同。这种情况下，非常适合使用字典树`（trie tree）`这种数据结构来进行存储，降低存储成本的同时，提高查询效率。

***

##### <font size="4" color="red">48. 从大量数据中找出高频词</font>

​		有一个`1GB`大小的文件，文件里每一行是一个词，每个词的大小不超过`16B`，内存大小限制是`1MB`，要求返回频数最高的`100`个词(`Top 100`)。

**分析：**由于内存限制，我们依然无法直接将大文件的所有词一次读到内存中。因此，同样可以采用分治策略，把一个大文件分解成多个小文件，保证每个文件的大小小于`1MB`，进而直接将单个小文件读取到内存中进行处理。

**解决方案：**

```
	首先遍历大文件，对遍历到的每个词x，执行 hash(x) % 5000 ，将结果为 i 的词存放到文件 ai 中。遍历结束后，我们可以得到5000个小文件。每个小文件的大小为200KB左右。如果有的小文件大小仍然超过 1MB，则采用同样的方式继续进行分解。
	接着统计每个小文件中出现频数最高的100个词。最简单的方式是使用HashMap来实现。其中key为词，value为该词出现的频率。具体方法是：对于遍历到的词 x，如果在 map 中不存在，则执行 map.put(x, 1) ；若存在，则执行 map.put(x, map.get(x)+1) ，将该词频数加1。
	上面我们统计了每个小文件单词出现的频数。接下来，我们可以通过维护一个小顶堆来找出所有词中出现频数最高的100个。具体方法是：依次遍历每个小文件，构建一个小顶堆，堆大小为 100。如果遍历到的词的出现次数大于堆顶词的出现次数，则用新词替换堆顶的词，然后重新调整为小顶堆，遍历结束后，小顶堆上的词就是出现频数最高的 100 个词。
```

**方法：**

```
1.分而治之，进行哈希取余；
2.使用 HashMap 统计频数；
3.求解最大的TopN个，用小顶堆；求解最小的 TopN 个，用大顶堆。
```

***

##### <font size="4" color="red">49. 查找网站访问频率高的IP地址</font>

​		现有海量日志数据保存在一个超大文件中，该文件无法直接读入内存，要求从中提取某天访问百度次数最多的那个`IP`。

**解决方案：**

```
	首先对文件进行一次遍历，把这一天访问百度 IP 的相关信息记录到一个单独的大文件中。对遍历到的每个词x，执行 hash(x) % 5000 ，将结果为 i 的词存放到文件 ai 中。遍历结束后，我们可以得到5000个小文件。每个小文件的大小为 200KB 左右。如果有的小文件大小仍然超过1MB，则采用同样的方式继续进行分解。接着统计每个小文件中IP出现的次数。最简单的方式是使用HashMap来实现。其中key为IP，value为该IP出现的频率。具体方法是：对于遍历到的IP，如果在map中不存在，则执行 map.put(x, 1) ；若存在，则执行 map.put(x, map.get(x)+1) ，将该词频数加 1。
	上面我们统计了每个小文件iP出现的频数。接下来，我们可以通过维护一个小顶堆来找出所有词中出现频数最高的100个。具体方法是：依次遍历每个小文件，构建一个小顶堆，堆大小为 100。如果遍历到的词的出现次数大于堆顶词的出现次数，则用新词替换堆顶的词，然后重新调整为小顶堆，遍历结束后，小顶堆上的词就是出现频数最高的IP
```

***

##### <font size="4" color="red">50. 从大量数据中找出不重复的整数</font>

​		在`2.5 `亿个整数中找出不重复的整数。注意：内存不足以容纳这`2.5`亿个整数。

**位图**，就是用一个或多个 bit 来标记某个元素对应的值，而键就是该元素。采用位作为单位来存储数据，可以大大节省存储空间。

位图通过使用位数组来表示某些元素是否存在。它可以用于快速查找，判重，排序等。不是很清楚？我先举个小例子。

假设我们要对 `[0,7]` 中的 5 个元素 (6, 4, 2, 1, 5) 进行排序，可以采用位图法。0~7 范围总共有 8 个数，只需要 8bit，即 1 个字节。首先将每个位都置 0：

```
0 0 0 0 0 0 0 0 Copy to clipboardErrorCopied
```

然后遍历 5 个元素，首先遇到 6，那么将下标为 6 的位的 0 置为 1；接着遇到 4，把下标为 4 的位 的 0 置为 1：

```
0 0 0 0 1 0 1 0Copy to clipboardErrorCopied
```

依次遍历，结束后，位数组是这样的：

```
0 1 1 0 1 1 1 0Copy to clipboardErrorCopied
```

每个为 1 的位，它的下标都表示了一个数：

```
for i in range(8):
    if bits[i] == 1:
        print(i)
```

**解决方案：**

```
与前面的题目方法类似，先将 2.5 亿个数划分到多个小文件，用 HashSet/HashMap 找出每个小文件中不重复的整数，再合并每个子结果，即为最终结果。
```

***

##### <font size="4" color="red">51. 查询热门的查询串</font>

​		搜索引擎会通过日志文件把用户每次检索使用的所有查询串都记录下来，每个查询串的长度不超过255字节。假设目前有1000w个记录（这些查询串的重复度比较高，虽然总数是1000w，但如果除去重复后，则不超过300w个）。请统计最热门的 10 个查询串，要求使用的内存不能超过1G。（一个查询串的重复度越高，说明查询它的用户越多，也就越热门）

**解决方案：**

```
当这些字符串有大量相同前缀时，可以考虑使用前缀树来统计字符串出现的次数，树的结点保存字符串出现次数，0表示没有出现。
```

> **注：**前缀树经常被用来统计字符串的出现次数。它的另外一个大的用途是字符串查找，判断是否有重复的字符串等。

***

##### <font size="4" color="red">52. 统计不同电话号码个数</font>

​		已知某个文件内包含一些电话号码，每个号码为8位数字，统计不同号码的个数。

**解决方案：**

```
申请一个位图数组，长度为 1 亿，初始化为 0。然后遍历所有电话号码，把号码对应的位图中的位置置为 1。遍历完成后，如果 bit 为 1，则表示这个电话号码在文件中存在，否则不存在。bit 值为 1 的数量即为 不同电话号码的个数。
```

***

##### <font size="4" color="red">53. 从5亿个数中找出中位数</font>

​		从5亿个数中找出中位数。数据排序后，位置在最中间的数就是中位数。当样本数为奇数时，中位数为 第 `(N+1)/2` 个数；当样本数为偶数时，中位数为 第 `N/2` 个数与第 `1+N/2`个数的均值。

**1.双堆法**

​		维护两个堆，一个大顶堆，一个小顶堆。大顶堆中最大的数**小于等于**小顶堆中最小的数；保证这两个堆中的元素个数的差不超过1。若数据总数为偶数，当这两个堆建好之后，中位数就是这两个堆顶元素的平均值。当数据总数为奇数时，根据两个堆的大小，中位数一定在数据多的堆的堆顶。

```java
class MedianFinder {

    private PriorityQueue<Integer> maxHeap;
    private PriorityQueue<Integer> minHeap;

    /** initialize your data structure here. */
    public MedianFinder() {
        maxHeap = new PriorityQueue<>(Comparator.reverseOrder());
        minHeap = new PriorityQueue<>(Integer::compareTo);
    }

    public void addNum(int num) {
        if (maxHeap.isEmpty() || maxHeap.peek() > num) {
            maxHeap.offer(num);
        } else {
            minHeap.offer(num);
        }

        int size1 = maxHeap.size();
        int size2 = minHeap.size();
        if (size1 - size2 > 1) {
            minHeap.offer(maxHeap.poll());
        } else if (size2 - size1 > 1) {
            maxHeap.offer(minHeap.poll());
        }
    }

    public double findMedian() {
        int size1 = maxHeap.size();
        int size2 = minHeap.size();

        return size1 == size2
            ? (maxHeap.peek() + minHeap.peek()) * 1.0 / 2
            : (size1 > size2 ? maxHeap.peek() : minHeap.peek());
    }
}
```

**2.分治法**

分治法的思想是把一个大的问题逐渐转换为规模较小的问题来求解。

对于这道题，顺序读取这 5 亿个数字，对于读取到的数字 num，如果它对应的二进制中最高位为 1，则把这个数字写到 f1 中，否则写入 f0 中。通过这一步，可以把这 5 亿个数划分为两部分，而且 f0 中的数都大于 f1 中的数（最高位是符号位）。

划分之后，可以非常容易地知道中位数是在 f0 还是 f1 中。假设 f1 中有 1 亿个数，那么中位数一定在 f0 中，且是在 f0 中，从小到大排列的第 1.5 亿个数与它后面的一个数的平均值。

> **提示**，5 亿数的中位数是第 2.5 亿与右边相邻一个数求平均值。若 f1 有一亿个数，那么中位数就是 f0 中从第 1.5 亿个数开始的两个数求得的平均值。

对于 f0 可以用次高位的二进制继续将文件一分为二，如此划分下去，直到划分后的文件可以被加载到内存中，把数据加载到内存中以后直接排序，找出中位数。

> **注意**，当数据总数为偶数，如果划分后两个文件中的数据有相同个数，那么中位数就是数据较小的文件中的最大值与数据较大的文件中的最小值的平均值。

***

##### <font size="4" color="red">54. 按照Qquery的频度排序</font>

​		有10个文件，每个文件大小为`1G`，每个文件的每一行存放的都是用户的 query，每个文件的`query`都可能重复。要求按照`query`的频度排序。

**1.`HashMap`法**

```
如果 query 重复率高，说明不同 query 总数比较小，可以考虑把所有的 query 都加载到内存中的 HashMap 中。接着就可以按照 query 出现的次数进行排序。
```

**2.分治法**

```
分治法需要根据数据量大小以及可用内存的大小来确定问题划分的规模。对于这道题，可以顺序遍历 10 个文件中的query，通过 Hash 函数 hash(query) % 10 把这些 query 划分到 10 个小文件中。之后对每个小文件使用 HashMap 统计 query 出现次数，根据次数排序并写入到零外一个单独文件中。接着对所有文件按照 query 的次数进行排序，这里可以使用归并排序（由于无法把所有 query 都读入内存，因此需要使用外排序）。
```

***

##### <font size="4" color="red">55. 找出排名前500的数</font>

​		有`20`个数组，每个数组有`500`个元素，并且有序排列。如何在这`20*500`个数中找出前`500`的数？

**解决方案：**

```
对于TopK问题，最常用的方法是使用堆排序。假设数组降序排列，可以采用以下方法：首先建立大顶堆，堆的大小为数组的个数，即为 20，把每个数组最大的值存到堆中。接着删除堆顶元素，保存到另一个大小为 500 的数组中，然后向大顶堆插入删除的元素所在数组的下一个元素。重复上面的步骤，直到删除完第 500 个元素，也即找出了最大的前 500个数。
```

```java
import lombok.Data;
import java.util.Arrays;
import java.util.PriorityQueue;

@Data
public class DataWithSource implements Comparable<DataWithSource> {
    /**
     * 数值
     */
    private int value;

    /**
     * 记录数值来源的数组
     */
    private int source;

    /**
     * 记录数值在数组中的索引
     */
    private int index;

    public DataWithSource(int value, int source, int index) {
        this.value = value;
        this.source = source;
        this.index = index;
    }

    /**
     *
     * 由于 PriorityQueue 使用小顶堆来实现，这里通过修改
     * 两个整数的比较逻辑来让 PriorityQueue 变成大顶堆
     */
    @Override
    public int compareTo(DataWithSource o) {
        return Integer.compare(o.getValue(), this.value);
    }
}

class Test {
    public static int[] getTop(int[][] data) {
        int rowSize = data.length;
        int columnSize = data[0].length;

        // 创建一个columnSize大小的数组，存放结果
        int[] result = new int[columnSize];

        PriorityQueue<DataWithSource> maxHeap = new PriorityQueue<>();
        for (int i = 0; i < rowSize; ++i) {
            // 将每个数组的最大一个元素放入堆中
            DataWithSource d = new DataWithSource(data[i][0], i, 0);
            maxHeap.add(d);
        }

        int num = 0;
        while (num < columnSize) {
            // 删除堆顶元素
            DataWithSource d = maxHeap.poll();
            result[num++] = d.getValue();
            if (num >= columnSize) {
                break;
            }

            d.setValue(data[d.getSource()][d.getIndex() + 1]);
            d.setIndex(d.getIndex() + 1);
            maxHeap.add(d);
        }
        return result;

    }

    public static void main(String[] args) {
        int[][] data = {
            {29, 17, 14, 2, 1},
            {19, 17, 16, 15, 6},
            {30, 25, 20, 14, 5},
        };

        int[] top = getTop(data);
        System.out.println(Arrays.toString(top)); // [30, 29, 25, 20, 19]
    }
}
```

***

##### <font size="4" color="red">56. Redis中复制集原理</font>

​		当启动一个`slave node`的时候，它会发送一个 `PSYNC` 命令给`master node`。如果这是`slave node`初次连接到`master node`，那么会触发一次 `full resynchronization` 全量复制。此时`master`会启动一个后台线程，开始生成一份 `RDB` 快照文件，同时还会将从客户端`client`新收到的所有写命令缓存在内存中。 `RDB` 文件生成完毕后， master 会将这个 `RDB` 发送给`slave`，`slave`会先写入本地磁盘，然后再从本地磁盘加载到内存中，接着`master`会将内存中缓存的写命令发送到`slave`，`slave`也会同步这些数据。`slave node`如果跟`master node `有网络故障，断开了连接，会自动重连，连接之后`master node`仅会复制给`slave`部分缺少的数据。

​		从`Redis2.8`开始，就支持主从复制的断点续传，如果主从复制过程中，网络连接断掉了，那么可以接着上次复制的地方，继续复制下去，而不是从头开始复制一份。`master node`会在内存中维护一个`backlog`，`master`和`slave`都会保存一个`replica offset`还有一个`master run id`，`offset`就是保存在`backlog`中的。如果 master 和`slave`网络连接断掉了，`slave`会让`master`从上次`replica offset`开始继续复制，如果没有找到对应的 offset，那么就会执行一次 `resynchronization` 。

***

##### <font size="4" color="red">57. Redis中无磁盘化复制</font>

```
repl-diskless-sync yes
# 等待 5s 后再开始复制，因为要等更多 slave 重新连接过来
repl-diskless-sync-delay 5
```

***

##### <font size="4" color="red">58. Redis中全量复制和增量复制</font>

**1.全量复制**

```
	master 执行 bgsave ，在本地生成一份 rdb 快照文件。master node 将 rdb 快照文件发送给 slave node，如果 rdb 复制时间超过 60 秒（repl-timeout），那么 slave node 就会认为复制失败，可以适当调大这个参数(对于千兆网卡的机器，一般每秒传输 100MB，6G 文件，很可能超过 60s)master node 在生成 rdb 时，会将所有新的写命令缓存在内存中，在 slave node 保存了 rdb 之后，再将新的写命令复制给 slave node。如果在复制期间，内存缓冲区持续消耗超过 64MB，或者一次性超过 256MB，那么停止复制，复制失败。
```

```
client-output-buffer-limit slave 256MB 64MB 60
```

**2.增量复制**

```
如果全量复制过程中，master-slave 网络连接断掉，那么 slave 重新连接 master 时，会触发增量复制。
master 直接从自己的 backlog 中获取部分丢失的数据，发送给 slave node，默认 backlog 就是 1MB。
master 就是根据 slave 发送的 psync 中的 offset 来从 backlog 中获取数据的。
```

**3.心跳检测**

```
主从节点互相都会发送 heartbeat 信息。master 默认每隔 10 秒发送一次 heartbeat，slave node 每隔 1 秒发送一个 heartbeat。
```

**4.异步复制**

```
master每次接收到写命令之后，先在内部写入数据，然后异步发送给 slave node。
```

***

##### <font size="4" color="red">59. Redis中持久化</font>

**RDB：**`RDB`持久化机制，是对`Redis`中的数据执行周期性的持久化。

**AOF：**`AOF`机制对每条写入命令作为日志，以 `append-only` 的模式写入一个日志文件中，在`Redis`重启的时候，可以通过回放`AOF`日志中的写入指令来重新构建整个数据集。

**RDB优点：**

```
(1) RDB会生成多个数据文件，每个数据文件都代表了某一个时刻中Redis的数据，这种多个数据文件的方式，非常适合做冷备，可以将这种完整的数据文件发送到一些远程的安全存储上去，比如说 Amazon 的 S3 云服务上去，在国内可以是阿里云的ODPS分布式存储上，以预定好的备份策略来定期备份Redis中的数据。
RDB对Redis对外提供的读写服务，影响非常小，可以让Redis保持高性能，因为 Redis 主进程只需要 fork 一个子进程，让子进程执行磁盘 IO 操作来进行 RDB 持久化即可。
(2)相对于AOF持久化机制来说，直接基于 RDB 数据文件来重启和恢复Redis进程，更加快速。
如果想要在Redis故障时，尽可能少的丢失数据，那么RDB没有AOF好。一般来说，RDB数据快照文件，都是每隔 5 分钟，或者更长时间生成一次，这个时候就得接受一旦 Redis 进程宕机，那么会丢失最近 5 分钟（甚至更长时间）的数据。
(3)RDB每次在fork子进程来执行RDB快照数据文件生成的时候，如果数据文件特别大，可能会导致对客户端提供的服务暂停数毫秒，或者甚至数秒。
```

**AOF优点：**

```
(1) AOF可以更好的保护数据不丢失，一般 AOF 会每隔 1 秒，通过一个后台线程执行一次 fsync 操作，最多丢失 1 秒钟的数据。
(2) AOF日志文件以 append-only 模式写入，所以没有任何磁盘寻址的开销，写入性能非常高，而且文件不容易破损，即使文件尾部破损，也很容易修复。
(3) AOF日志文件即使过大的时候，出现后台重写操作，也不会影响客户端的读写。因为在 rewrite log 的时候，会对其中的指令进行压缩，创建出一份需要恢复数据的最小日志出来。在创建新日志文件的时候，老的日志文件还是照常写入。当新的 merge 后的日志文件 ready 的时候，再交换新老日志文件即可。AOF 日志文件的命令通过可读较强的方式进行记录，这个特性非常适合做灾难性的误删除的紧急恢复。比如某人不小心用 flushall 命令清空了所有数据，只要这个时候后台 rewrite 还没有发生，那么就可以立即拷贝 AOF 文件，将最后一条 flushall 命令给删了，然后再将该 AOF 文件放回去，就可以通过恢复机制，自动恢复所有数据。
(4) 对于同一份数据来说，AOF 日志文件通常比 RDB 数据快照文件更大。
AOF 开启后，支持的写 QPS 会比 RDB 支持的写 QPS 低，因为 AOF 一般会配置成每秒 fsync 一次日志文件，当然，每秒一次 fsync ，性能也还是很高的。（如果实时写入，那么 QPS 会大降，Redis 性能会大大降低）
以前 AOF 发生过 bug，就是通过 AOF 记录的日志，进行数据恢复的时候，没有恢复一模一样的数据出来。所以说，类似 AOF 这种较为复杂的基于命令日志 merge 回放的方式，比基于 RDB 每次持久化一份完整的数据快照文件的方式，更加脆弱一些，容易有 bug。不过 AOF 就是为了避免 rewrite 过程导致的 bug，因此每次 rewrite 并不是基于旧的指令日志进行 merge 的，而是基于当时内存中的数据进行指令的重新构建，这样健壮性会好很多。
```

**RDB和AOF选择：**

```
(1) 不能仅仅使用RDB，因为会导致你丢失数据
(2) Redis支持同时开启两种持久化方式，我们可以综合使用AOF和RDB两种持久化机制，用AOF来保证数据不丢失，作为数据恢复的第一选择，用RDB来做不同程度的冷备，在AOF文件都丢失或则损坏不可用的时候，还可以用RDB来进行快速数据恢复
```


***

##### <font size="4" color="red">60. Redis中哨兵模式实现高可用</font>

**哨兵模式功能：**

```
(1) 集群监控；负责监控Redis master和slave进程是否正常工作
(2) 消息通知；如果某个Redis实例故障，那么哨兵负责发送消息作为报警通知给管理员
(3) 故障转移；如果master node挂掉了，会自动转移到slave node上
(4) 配置中心；如果故障转移发生了，通知client客户端新的master地址
```

**`Redis`哨兵主备切换数据丢失问题：**

(1) 异步复制导致数据丢失

```
    因为master-slave的复制是异步的，所以可能有部分数据还没复制到slave，master就宕机了，此时这部分数据就丢失了。
```

(2) 脑裂导致的数据丢失

```
   某个master机器突然脱离正常网络，跟其他slave不能连接，但是实际master还运行着，此时哨兵可能会认为这个master宕机了，然后开启选举，将其他slave切换成master，这时集群就会有两个master存在，这就是所谓的脑裂。
   此时虽然某个slave别切换成master，但可能client还没来得及切换到新的master，还继续向旧的master写入数据，因此旧的master再次回复的时候，会被作为一个slave重新挂到新额master上，自己的数据会被清空，重新的master复制数据，而新的master没有后来的client的数据，因此数据丢失。
```

***

