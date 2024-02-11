[toc]

---

# 前言
jvm启动参数分为三类：
1. 以 - 开头的为标准参数，所有的jvm都要实现这些参数，并向后兼容。（-D设置系统属性）
2. 以 -X 开头的为非标准参数，基本都是传给JVM的，默认JVM实现这些参数的功能，但是不保证所有JVM实现都满足，且不保证向后兼容。**可以使用java -X 查看当前jvm支持的非标准参数**。（-Xmx8g 设置堆内存8g）
3. 以 -XX 开头的为非稳定参数，专门用于控制JVM的行为，跟具体的JVM实现有关，随时可能在下一个版本取消。
    - -XX:+-Flag形式，+-是对布尔值进行开关
    - -XX:key=value形式，指定某个选项的值


# 特点和作用

下面按照一些特点和作用来分类
1. 系统属性参数
2. 运行模式参数
3. 堆内存设置参数
4. GC 设置参数
5. 分析诊断参数
6. javaAgent 参数

## 系统属性参数
系统属性参数主要是给我们系统提供一些**环境变量**和传递传递系统内需要使用的一些开关或者数值。

> 环境变量就是配置在系统变量里比如path等信息里面，用于影响所有启动项目的。

### 常用参数
- -Dfile.encoding=UTF-8。设置编码类型
- -Duser.timezone=GMT+08。设置时区
- -Dmaven.test.skip=ture。设置mavne跳过测试
- -Dio.netty.eventLoopThreads=8。设置netty线程数为8

## 自定义参数

```shell
java -Da=100 -jar xzxx.jar
```

等价于，在系统里执行下面代码

```java
System.setProperty("a","100");

// 获取代码
String a = System.getProperty("a");
```

linux下，还可以这样启动

```shell
a=100 java -jar xzxx.jar
```

## 运行模式
Jvm运行时，采取的运行模式。

### 常用模式
1. -server。设置jvm使用server模式，特点：
    - 启动速度慢，但运行时性能和内存管理效率很高，适用生产环境
    - 在具有64位能力的JDK环境将默认启用该模式，而忽略-client参数
2. -client。设置jvm使用client模式，特点：
    - 在jdk1.7之前在32位的x86机器上的默认值是-client选项。
    - 启动速度比较快，但运行时性能和内存管理效率不高，通常用与客户端应用程序或pc应用开发和调试。
3. -Xint。在解释模式（interpreted mode）下运行，特点是：
    - -Xint标记会强制JVM解释执行所有的字节码
    - 通常运行速度偏低，一般低10倍或者更多。
4. -Xcomp。与Xint相反，是编译模式，特点是：
    - JVM会在第一次使用时把所有的字节码编译成本地代码，从而带来最大程度的优化
    - 注意预热的问题
5. -Xmixed。混合模式，将解释模式和编译模式混合使用，特点是：
    - 由JVM决定，这是jvm的默认模式，也是推荐模式
    - 使用java -version可以看到mixed mode等信息

## 堆内存
配置堆内存分配情况

### 常用
- -Xmx。指定最大堆内存。如 -Xmx4g。这只是**限制了 Heap 部分的最大值为4g**。**这个内存不包括栈内存，也不包括堆外使用的内存**
- -Xms, 指定堆内存空间的初始大小。 如 -Xms4g。 而且指定的内存大小，并不是操作系统实际分配的初始值，而是GC先规划好，用到才分配。专用服务器上需要**保持-Xms和-Xmx一致**，否则启动后会有几次FullGC产生。当两者不一致时，堆内存扩容可能会导致性能抖动。
- -Xmn, 等价于 -XX:NewSize，**使用 G1 垃圾收集器 不应该设置该选项**，在其他的某些业务场景下可以设置。==**官方建议设置为 -Xmx 的 1/2 ~ 1/4**。==。默认Young区和Old区是1:2，所以是1/3.
- -XX：MaxPermSize=size, 这是 JDK1.7 之前使用的。Java8 默认允许的 Meta空间无限大，此参数无效。
- -XX：MaxMetaspaceSize=size, Java8 默认不限制 Meta 空间，**一般不允许设置该选项**。
- -XX：MaxDirectMemorySize=size，系统可以使用的最大堆外内存，这个参数跟 -Dsun.nio.MaxDirectMemorySize 效果相同。
- -Xss, 设置每个线程栈的字节数，影响栈的深度。 例如 -Xss1m 指定线程栈为1MB，与-XX:ThreadStackSize=1m 等价。 默认是1M

### 性能优化注意点
1. 专用服务器上需要保持 –Xms 和 –Xmx 一致。否则可能导致：
    - 应用刚启动可能就有好几个 FullGC。当两者配置不一致时，堆内存扩容可能会导致性能抖动。
    - 当内存还没到达 -Xmx配置的最大内存时，系统内存已经不够用了，然后溢出的问题。
2. -Xmn/-XX:NewSize。官方建阳设置为 -Xmx 的 1/2 ~ 1/4
3. 如果我们发现某些线程的调用栈发生了内存溢出，基本就是调用深度过深，比如死循环、2个方法来回调用，最终导致使用的总内存超出了Xss的限制(1M)，就会溢出了。
4. 尽量不要在线上做java应用程序的混合部署。每个应用都放在虚拟机或者docker里面，做到资源隔离，这样进程之间不会抢内存资源。
5. -Xmx在一般情况下可以配置内存的60%-80%，一定要留20-30%的预留空间，需要预留给堆外内存以及其它应用使用。


## GC相关
### 常用
- -XX：+UseG1GC：使用 G1 垃圾回收器
- -XX：+UseConcMarkSweepGC：使用 CMS 垃圾回收器
- -XX：+UseSerialGC：使用串行垃圾回收器
- -XX：+UseParallelGC：使用并行垃圾回收器
- -XX：+UnlockExperimentalVMOptions -XX:+UseZGC。Java 11+。
- -XX：+UnlockExperimentalVMOptions -XX:+UseShenandoahGC。Java 12+

### 各版本默认GC
- java7 - Parallel GC
- java8 - Parallel GC
- java9 - G1 GC
- java10 - G1 GC
- java11 - ZGC

## 分析诊断
### 常用
- -XX：+-HeapDumpOnOutOfMemoryError 选项，当 OutOfMemoryError 产生，即内存溢出（堆内存或持久代) 时，自动 Dump 堆内存快照。
    - 示例用法： java -XX:+HeapDumpOnOutOfMemoryError -Xmx256m ConsumeHeap
- -XX：HeapDumpPath 选项，与 HeapDumpOnOutOfMemoryError 搭配使用，**指定内存溢出时 Dump 文件的目录**。
    - 如果没有指定则**默认为启动 Java 程序的工作目录**。
    - 示例用法： java -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/usr/local/ConsumeHeap 自动 Dump 的 hprof 文件会存储到 /usr/local/ 目录下。
- -XX：OnError 选项，发生致命错误时（fatal error）执行的脚本。
    - 例如, 写一个脚本来记录出错时间, 执行一些命令，或者 curl 一下某个在线报警的 url。
    - 示例用法：java -XX:OnError="gdb - %p" MyApp。可以发现有一个 %p 的格式化字符串，表示进程 PID。
- -XX：OnOutOfMemoryError 选项，抛出 OutOfMemoryError 错误时执行的脚本。
- -XX：ErrorFile=filename 选项，致命错误的日志文件名,绝对路径或者相对路径。


### 远程调试
- -Xdebug -Xrunjdwp:transport=dt_socket,server=y,suspend=n,address=1506。


## JavaAgent
Agent 是 JVM 中的一项黑科技，可以通过**无侵入**方式来做很多事情，比如注入 AOP 代码，执行统计等等，权限非常大。这里简单介绍一下配置选项，详细功能需要专门来讲。

### 如何设置 agent
- -agentlib:libname[=options] 启用 native 方式的 agent，参考 LD_LIBRARY_PATH 路径。
- -agentpath:pathname[=options] 启用 native 方式的 agent。 -javaagent:jarpath[=options] 启用外部的 agent 库，比如 pinpoint.jar 等等。
- -Xnoagent 则是禁用所有 agent。


### 例子
#### CPU使用时间抽样分析
> JAVA_OPTS="-agentlib:hprof=cpu=samples,file=cpu.samples.log"