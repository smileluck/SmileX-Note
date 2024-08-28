[TOC]

---

# 集群配置

使用clusterServersConfig

```yml
#  Redisson配置 spring.redisson.clusterServersConfig
spring:
    redisson:
        file: classpath: clusterServersConfig.yml
```

```yaml
# 集群模式
clusterServersConfig:
  # 假若当前连接池里的连接数量超过了最小空闲连接数，同时有连接空闲时间超过了该数值，那么这些连接将会自动被关闭，并从连接池里去掉。时间单位是毫秒。默认10000
  idleConnectionTimeout: 1000
  # 同节点建立连接时的等待超时。单位:毫秒, 默认10000（10秒）
  connectTimeout: 1000
  # 等待节点回复命令的时间。该时间从命令发送成功时开始计时。默认3000
  timeout: 1000
  # 命令失败重试次数,如果尝试达到 retryAttempts（命令失败重试次数） 仍然不能将命令发送至某个指定的节点时，将抛出错误。
  # 如果尝试在此限制之内发送成功，则开始启用 timeout（命令等待超时） 计时。
  retryAttempts: 3
  # 在一条命令发送失败以后，等待重试发送的时间间隔。时间单位是毫秒。默认1500
  retryInterval: 1500
  # 失败从节点重连间隔时间
  failedSlaveReconnectionInterval: 3000
  # 失败的从节点校验时间间隔
  failedSlaveCheckInterval: 60000
  # 用于节点身份验证的密码。默认null
  password: null
  # 每个连接的最大订阅数量。默认值5
  subscriptionsPerConnection: 5
  # 在Redis节点里显示的客户端名称。默认null
  clientName: rick
  # 在使用多个Redis服务节点的环境里，可以选用以下几种负载均衡方式选择一个节点：
  # org.redisson.connection.balancer.WeightedRoundRobinBalancer - 权重轮询调度
  # org.redisson.connection.balancer.RoundRobinLoadBalancer - 轮询调度
  # org.redisson.connection.balancer.RandomLoadBalancer - 随机调度
  # 默认：RoundRobinLoadBalancer
  loadBalancer: !<org.redisson.connection.balancer.RoundRobinLoadBalancer> {}
  # 用于发布和订阅连接的最小保持连接数（长连接）。Redisson内部经常通过发布和订阅来实现许多功能。
  # 长期保持一定数量的发布订阅连接是必须的。默认值为1
  subscriptionConnectionMinimumIdleSize: 1
  # 多从节点的环境里，每个从服务节点里用于发布和订阅连接的连接池最大容量。连接池的连接数量自动弹性伸缩。默认值为50
  subscriptionConnectionPoolSize: 50
  # 多从节点的环境里，每个从服务节点里用于普通操作（非发布和订阅）的最小保持连接数（长连接）。
  # 长期保持一定数量的连接有利于提高瞬时读取反映速度， 默认值为32
  slaveConnectionMinimumIdleSize: 32
  # 多从节点的环境里，每个从服务节点里用于普通操作（非 发布和订阅）连接的连接池最大容量。连接池的连接数量自动弹性伸缩。默认64
  slaveConnectionPoolSize: 64
  # 多从节点的环境里，每个主节点的最小保持连接数（长连接）。长期保持一定数量的连接有利于提高瞬时写入反应速度。默认值为32
  masterConnectionMinimumIdleSize: 32
  # 多主节点的环境里，每个主节点的连接池最大容量。连接池的连接数量自动弹性伸缩。 默认64
  masterConnectionPoolSize: 64
  # 设置读取操作选择节点的模式。 可用值为：
  # SLAVE - 只在从服务节点里读取。 （默认）
  # MASTER - 只在主服务节点里读取。
  # MASTER_SLAVE - 在主从服务节点里都可以读取。
  readMode: "SLAVE"
  # 设置订阅操作选择节点的模式。可用值为：
  # （1）SLAVE - 只在从服务节点里订阅。默认
  # （2）MASTER - 只在主服务节点里订阅。
  subscriptionMode: "SLAVE"
  # 集群节点地址
  nodeAddresses:
    - "redis://10.2.51.100:6379"
    - "redis://10.2.51.101:6380"
    - "redis://10.2.51.102:6381"
    - "redis://10.2.51.103:6382"
  # 对主节点变化节点状态扫描的时间间隔。单位是毫秒。
  scanInterval: 1000
  # ping连接间隔
  pingConnectionInterval: 0
  # 是否保持连接
  keepAlive: false
  # tcp连接无延迟
  tcpNoDelay: false
# 这个线程池数量被所有RTopic对象监听器，RRemoteService调用者和RExecutorService任务共同共享。默认当前处理核数量 * 2
threads: 8
# 这个线程池数量是在一个Redisson实例内，被其创建的所有分布式数据类型和服务，以及底层客户端所一同共享的线程池里保存的线程数量。
nettyThreads: 8
# Redisson的对象编码类是用于将对象进行序列化和反序列化，以实现对该对象在Redis里的读取和存储。
# Redisson提供了多种的对象编码应用，以供大家选择：https://github.com/redisson/redisson/wiki/4.-data-serialization
# 默认 org.redisson.codec.MarshallingCodec，部分编码需要引入编码对于依赖jar包。
codec: !<org.redisson.codec.MarshallingCodec> {}
# 传输模式，默认NIO，可选参数：
# TransportMode.NIO,
# TransportMode.EPOLL - 需要依赖里有netty-transport-native-epoll包（Linux）
# TransportMode.KQUEUE - 需要依赖里有 netty-transport-native-kqueue包（macOS）
transportMode: "NIO"
```
