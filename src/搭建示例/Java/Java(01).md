##### <font size="4" color="red">01. Jvm中堆和栈</font>

**程序运行时内存分配策略：**静态的、栈式的、堆式的

**静态存储分配：**指在编译时就能确定每个数据目标在运行时刻的存储空间需求，因而在编译时就可以给他们固定的内存空间，这种分配策略要求程序代码中不允许有可变数据结构(比如：可变数组)的存在，也不允许有嵌套递归的结构出现

**栈式存储分配：**可称为动态存储分配，由一个类似于堆栈的运行栈来实现，程序对数据区的需求在编译是完全未知的，只有到运行的时候才能够知道，但是在规定在运行中进入一个程序模块时，必须知道该程序所需的数据区大小才能够为其分配内存，栈式存储分配按照先进后出的原则进行分配。

Java把内存划分成两种：一种是栈内存，一种是堆内存。 

**栈内存：**当在一段代码块定义一个变量时，Java就在栈中为这个变量分配内存空间，当超过变量的作用域后，Java会自动释放掉为该变量所分配的内存空间，该内存空间可以立即被另作他用。栈中的数据大小和生命周期是可以确定的，当没有引用指向数据时，这个数据就会消失。堆中的对象的由垃圾回收器负责回收，因此大小和生命周期不需要确定，具有很大的灵活性。 

**堆内存：**堆内存用来存放由new创建的对象和数组。在堆中分配的内存，由Java虚拟机的自动垃圾回收器来管理。

JVM是基于堆栈的虚拟机.JVM为每个新创建的线程都分配一个堆栈，也就是说对于一个Java程序来说，它的运行就是通过对堆栈的操作来完成的。堆栈以帧为单位保存线程的状态。JVM对堆栈只进行两种操作:以帧为单位的压栈和出栈操作。 

***

##### <font size="4" color="red">02. Http和Https的区别</font>

![](./img/img(01)/02-01.png)

(1) HTTP 明文传输，数据都是未加密的，安全性较差，HTTPS（SSL+HTTP） 数据传输过程是加密的，安全性较好。

(2) HTTPS 其实就是建构在 SSL/TLS 之上的 HTTP 协议，所以，要比较 HTTPS 比 HTTP 要更耗费服务器资源

***

##### <font size="4" color="red">03. Netty的特点</font>

​		Netty是一个异步事件驱动的网络应用程序框架，用于开发可维护的高性能协议服务器和客户端，Netty基于nio的，它封装了Jdk的nio，让使用更加灵活。

(1) 高并发;Netty是一款基于NIO的开发网络通信框架，对比BIO，它的并发性能高

(2) 传输快;传输依赖于零拷贝特性，尽量减少不必要的内存拷贝，实现了更高效率的传输

(3) 封装好;Netty封装了NIO操作的很多细节，提供了易于使用调用的接口

***

##### <font size="4" color="red">04. Netty的优势</font>

(1) 使用简单：封装了NIO的很多细节，使用更简单

(2) 功能强大：预置了多种编码功能，支持多种主流协议

(3) 定制能力强：可以通过ChannelHandler对通信框架灵活地扩展

(4) 性能高：通过与其他业界主流的NIO框架对比，让开发人员可以专注于业务本身

(5) 稳定：Netty修复了已经发现的所有NIO的bug

***

##### <font size="4" color="red">05. Netty的应用场景</font>

阿里分布式服务框架Dubbo，默认使用Netty作为基础通信组件

RocketMQ使用Netty作为通讯的基础组件

***

##### <font size="4" color="red">06. Netty的高性能表现</font>

(1) Netty线程模型：同步非阻塞，用最少的资源做更多的事

(2) 内存零拷贝：尽量减少不必要的内存拷贝，实现了更高效率的传输。

(3) 内存池设计：申请的内存可以重用，主要指直接内存，内部实现用二叉查找树管理内存分配情况

(4) 串形化处理读写：避免使用锁带来的性能开销

(5) 高性能序列化协议：支持protobuf等高性能序列化协议

***

##### <font size="4" color="red">07. BIO、NIO和AIO区别</font>

(1) BIO一个连接一个线程，客户端有连接请求时服务器端就需要启动一个线程进行处理，线程开销大

(2) NIO一个请求一个线程，但客户端发送的连接请求都会注册到多路复用器上，多路复用器轮询到连接有I/O请求时才启动一个线程进行处理

(3) AIO一个有效请求一个线程，客户端的I/O请求都是由OS先完成了再通知服务器去启动线程进行处理

(4) BIO是面向流的，NIO是面向缓冲区的，BIO各种流是阻塞的，而NIO是非阻塞的；BIO的Stream是单向的，而NIO的channel是双向的。

NIO的特点：事件驱动模型、单线程处理多任务、非阻塞I/O、I/O多路复用提高了网络应用的可伸缩性和实用性，基于Reactor线程模型。

***

##### <font size="4" color="red">08. Netty的线程模型</font>

​		Netty通过Reactor模型基于多路复用器接收并处理用户请求，内部实现了两个线程池，boss线程池和work线程池，其中boss线程池的线程负责处理请求事件，当收到accept事件的请求时，把对应的socket封装到一个NioSocketChannel中，并交给work线程池，其中work线程池负责请求的read和write对应的Handler处理。

***

##### <font size="4" color="red">09. Tcp粘包/拆包的原因及解决方法</font>

​		`TCP`是以流的方式来处理数据，一个完整的包可能会被`TCP`拆分成多个包进行发送，也可能把小的封装成一个大的数据包发送。

`Tcp`粘包/分包的原因：

​		应用程序写入的直接大小大于套接字发送缓冲区的大小，会发生拆包现象，而应用程序写入数据小于套接字缓冲区大小，网卡将应用多次写入的数据发送到网络上，这将会发生粘包现象。

**解决办法：**

消息定长：FixedLengthFrameDecoder类

包尾增加特殊字符分割：行分隔符类：LineBasedFrameDecoder 或自定义分隔符类 ：DelimiterBasedFrameDecoder

将消息分为消息头和消息体：LengthFieldBasedFrameDecoder类。分为有头部的拆包与粘包、长度字段在前且有头部的拆包与粘包、多扩展头部的拆包与粘包。

***

##### <font size="4" color="red">10.  什么是Netty零拷贝技术</font>

(1) Netty 的接收和发送 ByteBuffer 采用 DIRECT BUFFERS，使用堆外直接内存进行 Socket 读写，不需要进行字节缓冲区的二次拷贝。如果使用传统的堆内存（HEAP BUFFERS）进行 Socket 读写，JVM 会将堆内存 Buffer 拷贝一份到直接内存中，然后才写入 Socket 中。相比于堆外直接内存，消息在发送过程中多了一次缓冲区的内存拷贝

(2) Netty提供了组合Buffer对象，可以聚合多个ByteBuffer对象，用户可以像操作一个Buffer那样方便的对组合Buffer进行操作，避免了传统通过内存拷贝的方式将几个小Buffer合并成一个大的Buffer。

(3) Netty 的文件传输采用了 transferTo 方法，它可以直接将文件缓冲区的数据发送到目标 Channel，避免了传统通过循环 write 方式导致的内存拷贝问题。

***

##### <font size="4" color="red">12.  默认情况Netty起有多少线程</font>

Netty 默认是 CPU 处理器数的两倍，bind 完之后启动。

***

##### <font size="4" color="red">13.  Netty支持哪些心跳设置</font>

`readerIdleTime`：为读超时时间（即测试端一定时间内未接受到被测试端消息）。

`writerIdleTime`：为写超时时间（即测试端一定时间内向被测试端发送消息）。

`allIdleTime`：所有类型的超时时间。

***

##### <font size="4" color="red">14.  Redis介绍</font>

​		Redis 可以存储键和五种不同类型的值之间的映射。键的类型只能为字符串，值支持五种数据类型：字符串、列表、集合、散列表、有序集合。

​		与传统数据库不同的是 Redis 的数据是存在内存中的，所以读写速度非常快，因此 redis 被广泛应用于缓存方向，每秒可以处理超过 10万次读写操作，是已知性能最快的Key-Value DB。另外，Redis 也经常用来做分布式锁。

***

##### <font size="4" color="red">15.  Redis有哪些优缺点</font>

**优点：**读写性能优异、支持数据持久化、支持事务、数据结构丰富、支持主从复制

**缺点：**

(1) 数据库容量受到物理内存的限制，不能用作海量数据的高性能读写，因此Redis适合的场景主要局限在较小数据量的高性能操作和运算上。

(2) Redis不具备自动容错和恢复功能，主机从机的宕机都会导致前端部分读写请求失败，需要等待机器重启或者手动切换前端的IP才能恢复。

(3) 主机宕机，宕机前有部分数据未能及时同步到从机，切换IP后还会引入数据不一致的问题，降低了系统的可用性。

(4) Redis 较难支持在线扩容，在集群容量达到上限时在线扩容会变得很复杂。为避免这一问题，运维人员在系统上线时必须确保有足够的空间，这对资源造成了很大的浪费。

***

##### <font size="4" color="red">16.  为什么要用Redis而不用map/guava做缓存</font>

​		缓存分为本地缓存和分布式缓存。以 Java 为例，使用自带的 map 或者 guava 实现的是本地缓存，最主要的特点是轻量以及快速，生命周期随着 jvm 的销毁而结束，并且在多实例的情况下，每个实例都需要各自保存一份缓存，缓存不具有一致性。使用 redis 或 memcached 之类的称为分布式缓存，在多实例的情况下，各实例共用一份缓存数据，缓存具有一致性。

***

##### <font size="4" color="red">17.  Redis为什么这么快</font>

(1) 完全基于内存，绝大部分请求是纯粹的内存操作，非常快速。数据存在内存中

(2) 数据结构简单，对数据操作也简单

(3) 采用单线程，避免了不必要的上下文切换和竞争条件，也不存在多进程或者多线程导致的切换而消耗 CPU，不用去考虑各种锁的问题，不存在加锁释放锁操作，没有因为可能出现死锁而导致的性能消耗

(4) 使用多路 I/O 复用模型，非阻塞 IO

(5) 使用底层模型不同，它们之间底层实现方式以及与客户端之间通信的应用协议不一样，Redis 直接自己构建了 VM 机制 ，因为一般的系统调用系统函数的话，会浪费一定的时间去移动和请求

***

##### <font size="4" color="red">18.  Redis支持的数据类型</font>

| 数据类型 | 可以存储的值           | 操作                                                         | 应用场景                                                     |
| -------- | ---------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| STRING   | 字符串、整数或者浮点数 | 对整个字符串或者字符串的其中一部分执行操作 对整数和浮点数执行自增或者自减操作 | 做简单的键值对缓存                                           |
| LIST     | 列表                   | 从两端压入或者弹出元素 对单个或者多个元素进行修剪， 只保留一个范围内的元素 | 存储一些列表型的数据结构，类似粉丝列表、文章的评论列表之类的数据 |
| SET      | 无序集合               | 添加、获取、移除单个元素 检查一个元素是否存在于集合中 计算交集、并集、差集 从集合里面随机获取元素 | 交集、并集、差集的操作，比如交集，可以把两个人的粉丝列表整一个交集 |
| HASH     | 包含键值对的无序散列表 | 添加、获取、移除单个键值对 获取所有键值对 检查某个键是否存在 | 结构化的数据，比如一个对象                                   |
| ZSET     | 有序集合               | 添加、获取、删除元素 根据分值范围或者成员来获取元素 计算一个键的排名 | 去重但可以排序，如获取排名前几名的用户                       |

***

##### <font size="4" color="red">19.  Redis持久化</font>

​		持久化就是把内存的数据写到磁盘中去，防止服务宕机了内存数据丢失

Redis 提供两种持久化机制 RDB（默认） 和 AOF 机制:RDB是Redis默认的持久化方式。按照一定的时间将内存的数据以快照的形式保存到硬盘中，对应产生的数据文件为dump.rdb。





***

##### <font size="4" color="red">20.  Redis回收算法</font>

LRU算法

***

##### <font size="4" color="red">21. Jdk和Jre的区别</font>

Jdk:Java开发工具包，提供了java的开发环境和运行环境

Jre：Java运行环境，为`java`的运行提供所需环境

***

##### <font size="4" color="red">22. Java字符串操作类</font>

​			`String`和`StringBuffer`、`StringBuilder`区别在于String声明的是不可变的对象，每次操作都会生成新的String对象，而`StringBuffer`、`StringBuilder`可以在原有对象的基础上进行操作，所以在经常改变字符串内容的情况下最好不要使用String

​		`StringBuffer`和`StringBuilder`最大的区别在于`StringBuffer`是线程安全的，而`StringBuilder`是非线程安全的，但`StringBuilder`的性能却高于`StringBuffer`，所以在单线程环境下推荐使用`StringBuilder`，多线程环境下推荐使用`StringBuffer`

***

##### <font size="4" color="red">23. Java中抽象类能使用final修饰吗</font>

不能，final修饰的类不能被继承

***

##### <font size="4" color="red">24. Java中HashMap和Hashtable区别</font>

(1) `HashMap`允许`key`和`value`为`null`，而`HashTable`不允许

(2) `HashTable`是同步的，而`HashMap`不是，所以`HashMap`适合单线程环境，`HashTable`适合多线程环境

***

##### <font size="4" color="red">25. Java中IO模型</font>

阻塞IO、非阻塞IO、多路复用IO、信号驱动IO、异步IO

***

##### <font size="4" color="red">26. Java中线程的实现方式</font>

(1) 继承`Thread`类

(2) 实现`Runnable`接口

(3) 使用`Callable`和`Future`

***

##### <font size="4" color="red">27. Java中线程种类</font>

Java线程有两种，一种是用户线程，一种是守护线程

***

##### <font size="4" color="red">28. Java中典型的守护线程</font>

GC(垃圾回收器)

***

##### <font size="4" color="red">29. Java中实现堆内存溢出</font>

```java
public class Cat {
    public static void main(String[] args){
        List list = new ArrayList();
        while(true){
            list.add(new Cat());
        }
    }
}
```

***

##### <font size="4" color="red">30. Java中OutOfMemoryError异常</font>

如果虚拟机在扩展栈时无法申请到足够的内存空间，则抛出OutOfMemoryError

***

##### <font size="4" color="red">31. Java中Jvm垃圾收集算法</font>

标记-清除算法、复制算法、标记-压缩算法、分代回收

***

##### <font size="4" color="red">32. Mysql中选择合适的存储引擎</font>

(1) `MyISAM`默认的`MySQL`插件式存储引擎，它是在`Web`、数据存储和其他应用环境下最常使用的存储引擎之一

(2) `InnoDB`用于事务处理应用程序，具有众多特性、包括`ACID`事务支持

(3) `Memory`将所有数据保存在`RAM`中，在需要快速查找引用和其他类似数据的环境下，可提供极快的访问。

(4) `Merge`允许`MySQL DBA`或开发人员将一系列等同的`MyISAM`表以逻辑方式组合在一起。

***

##### <font size="4" color="red">33. Mysql中char和vchar区别</font>

保存和检索的方式不同，它们的最大长度和是否尾部空格被保留等方面也不同，在存储或检索过程中不进行大小写转换。

***

##### <font size="4" color="red">34. Mysql中索引</font>

**索引：**是对数据库表中一列或多列的值进行排序的一种结构，它是某个表中一列或若干列值的集合和相应的指向表中物理标识这些值的数据页的逻辑指针清单。

(1) 搜索的索引列；最适索引的列是出现`Where`子句中的列，或连接子句中指定的列，而不是出现在select关键字后选择列表中的列

(2) 唯一索引；对于唯一值的列，索引的效果最好，而具有多个重复值的列，其效果最差。

(3) 使用短索引；如果对串列进行索引，应该指定一个前缀长度，例如：如果一个char(200)列，如果再前10个或20个字符内，多数值是唯一的，那么就不要对整个列进行索引

(4) 利用最左前缀；多列索引可起几个索引作用，因为可利用索引最左边的列集来匹配。

​		不要过度索引；每个额外的索引都要占用额外的磁盘空间，并降低写操作的性能，在修改表的内容时，索引必须进行更新，有时可能需要重构，因此，索引越多，所花的时间越长。

***

##### <font size="4" color="red">35. Mysql中索引分类</font>

数据结构角度：`Btree`、`Hash`、`Fulltext`、`R-tree`

物理存储角度：聚集索引、非聚集索引

逻辑角度：普通索引、唯一索引、主键索引、组合索引、全文索引

***

##### <font size="4" color="red">36. Mysql中非聚簇索引一定会回表查询吗</font>

不一定，这涉及到查询语句所要求的字段是否命中了索引，如果全部命中了索引，那么久不必在进行回表查询。

***

##### <font size="4" color="red">38. Mysql中查询数据库中最后一条数据</font>

```sql
select * from table_name order by id desc limit 1;
```

***

##### <font size="4" color="red">39. 3NF范式</font>

`1NF`指数据库表中的任何属性都具有原子性，不可再分割

`2NF`是对记录的唯一性约束，要求就有唯一标识，即实体唯一性

`3NF`是对字段冗余性的约束，即任何字段不能由其他字段派生出来，它要求字段没有冗余

***

##### <font size="4" color="red">40. Mysql中查看表结构</font>

```shell
desc <table_name>;
```

***

##### <font size="4" color="red">41. Mysql中删除表的几种方式</font>

(1) `delete`仅删除表中数据，支持条件过滤，支持回滚

```shell
delete from <table_name>;
```

(2) `truncate`仅删除所有数据，不支持条件过滤，不支持回滚，效率高于`delete`

```shell
truncate table <table_name>;
```

(3) `drop`删除表数据同时删除表结构，将表所占用空间都释放掉，删除效率最高

```shell
drop table <table_name>;
```

***

##### <font size="4" color="red">42. Like走索引吗</font>

```shell
xxx% #走索引
%xxx #不走索引
```

***

##### <font size="4" color="red">43. Mysql查看索引</font>

```shell
show index from <table_name>;
```

***

##### <font size="4" color="red">44. MVVM/MVC介绍</font>

**(1) MVVM介绍**

​		唯一的区别是，它采用双向绑定（data-binding）：View的变动，自动反映在 ViewModel，反之亦然。Angular 都采用这种模式。

**(2) MVC介绍**

1.View 传送指令到 Controller

2.Controller 完成业务逻辑后，要求 Model 改变状态

3.Model 将新的数据发送到 View，用户得到反馈

***

##### <font size="4" color="red">45. Aop介绍</font>

`Aop`面向切面编程，通过预编译方式和运行期间动态代理实现程序功能的统一维护的一种技术。

(1) 前置通知(Before)：目标方法被调用之前调用通知功能

(2) 后置通知(After)：目标方法完成之后调用通知

(3) 返回通知(After-returning)：目标方法成功执行之后调用通知

(4) 异常通知(After-throwing)：目标方法抛出异常后调用通知

(5) 环绕通知(Around)：在被通知的方法调用之前和调用之后执行自定义的行为

***

##### <font size="4" color="red">46. SpringMVC核心</font>

`DispatcherServlet`

***

##### <font size="4" color="red">47. SpringMVC的几个组件</font>

`DispatcherServlet`：前端控制器，也叫中央控制器，相关组件都是它来调度

`HandlerMapping`：处理器映射器，根据URL路径映射到不同的Handler

`HandlerAdapter`：处理器适配器，按照`HandlerAdapter`的规则去执行Handler

`Handler`：处理器由我们自己根据业务开发

`ViewResolver`：视图解析器，把逻辑视图解析成具体视图

`View`：一个接口，它的实现支持不同的视图类型

***

##### <font size="4" color="red">48. Mysql最多创建多少索引</font>

```
16
```

****

##### <font size="4" color="red">49. 为什么最好建立一个主键</font>

​		主键是数据库确保数据行在整张表的唯一性的保障，即使业务没有主键也建议添加一个自增长的ID列做为主键，设定了主键之后，在后续的删改查的时候可能更加快速度以及确保操作数据范围安全。

***

##### <font size="4" color="red">50. 为什么建议字段Not Null</font>

`null`值会占用更多字节，且会在程序中造成与很多预期不符的情况。

***

##### <font size="4" color="red">51. Spring Ioc注入哪几种方式</font>

(1) 构造函数

(2) setter注入

(3) 接口注入

***

##### <font size="4" color="red">52. Ioc优点、缺点</font>

**优点：**组件之间的解耦，提高程序的可维护性、灵活性

**缺点：**

(1) 对创建对象复杂，有一定学习成本

(2) 利用反射创建对象，效率上有损

***

##### <font size="4" color="red">53. Spring配置方式</font>

(1) 基于Xml

(2) 基于注解

(3) 基于Java

****

##### <font size="4" color="red">54. Aop动态策略</font>

(1) 如果目标对象实现了接口，默认采用Jdk动态代理方式

(2) 如果没有实现接口,采用CgLib进行动态代理

***

##### <font size="4" color="red">55. Bean是线程安全吗</font>

不是，具体线程问题需要开发人员处理。

***

##### <font size="4" color="red">56. Spring中接收路径参数</font>

```java
@PathVariable
```

***

##### <font size="4" color="red">57. Spring中@Cacheable注解</font>

用来标记缓存查询

> **注：**@CacheEvict清空缓存注解

***

##### <font size="4" color="red">58. Spring中事务注解</font>

```java
@Transaction
```

****

##### <font size="4" color="red">59. Nginx负载均衡几种算法</font>

(1) 轮循模式；每个请求按照时间顺序逐一分配到不同的后端服务器

(2) 权重模式；

(3) Ip_hash模式；每个请求结果按照ip/hash结果分配，这样每个访客固定访问一个后端服务器，可以解决session问题

(4) url_hash模式

(5) fair模式；按照服务器的响应速度来分配

****

##### <font size="4" color="red">60. Nginx开启压缩功能</font>

**好处：**压缩是可以节省带宽，提高传输效率

**坏处：**由于在服务器上进行压缩，会消耗服务器性能

***


