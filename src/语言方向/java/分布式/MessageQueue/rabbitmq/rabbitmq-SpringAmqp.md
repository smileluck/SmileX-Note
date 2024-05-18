[toc]

---

# 概念

## 什么是AMQP

全称 Advanced Message Queuing Protocol，是用于在应用程序之间传递业务消息的开放标准。该协议与语言和平台无关，更符合微服务中独立性的要求。

## SpringAMQP

 Spring AMQP是基于AMQP协议定义的一套API规范，提供了模板**用来发送和接收消息**。包含两部分，其中spring-amqp是基础抽象，spring-rabbit是底层的默认实现。 



# 前置知识

**推荐Bean方式创建**

## 创建队列

- name：队列名.
- durable：是否持久化.
- exclusive：是否独占.  true表示队列只能被一个消费者使用, false表示大家都能用.
- autoDelete：自动删除，为 true 表示没有人使用以后自动删除.
- arguments：扩展参数，自定义选项，例如实现，队列过期、队列消息过期、死信队列...

```java
@Configuration
public class MqConfig {
 
    /**
     * 创建队列
     * @return
     */
    @Bean
    public Queue simpleQueue() {
        return new Queue("queue", true);
    }
 
}
```



## 创建交换机

- name：交换机名.
- durable：是否持久化.
- autoDelete：自动删除，若当前交换机没人使用了，就会自动删除.
- arguments：表示创建交换机时可以指定一些额外的选项，例如延时消息、过期时间、死信交换机.

```java
@Configuration
public class MqConfig {
 
    /**
     *  创建直接交换机
     * @return
     */
    @Bean
    public DirectExchange directExchange() {
        return new DirectExchange("direct", true);
    }
 
}
```



## 创建绑定Key

```java
@Configuration
public class MqConfig {
 
    /**
     * 创建队列
     * @return
     */
    @Bean
    public Queue simpleQueue() {
        return new Queue("simple.queue", true);
    }
 
    /**
     *  创建直接交换机
     * @return
     */
    @Bean
    public DirectExchange directExchange() {
        return new DirectExchange("direct", true, true);
    }
 
    /**
     * 创建绑定
     * @return
     */
    @Bean
    public Binding simpleBinding() {
        return BindingBuilder.bind(simpleQueue()).to(directExchange()).with("simple");
    }
 
 
}
```



## @RabbitListener 注解

### queue存在

如果 queue 队列存在，以下使用就可以通过 @RabbitListener 注解监听队列消息，成为消费者.

用法：直接指定注解中的`queues`参数即可，参数值为对列名(queueName)

```java
@Component
public class RabbitListener {
    @RabbitListener(queues = "result_queue")
    public void rabbitMsg(String msg) {
        logger.info("receive: {}", msg);
    }
}
```

 当 queue 的 autoDelete 属性为 false 时，这种使用情况还是比较合适了；但是，当这个属性为 true 时，没有消费者队列就会自动删除了，可能会引发异常！ 



### queue 不存在

 如果队列存在，以下使用就可以通过 @RabbitListener 注解创建交换机、队列、绑定关系. 

```java
@Component
public class RabbitListener {
    @RabbitListener(bindings = @QueueBinding(
        value = @Queue("queue1"),
        exchange = @Exchange(name = "exchange1", type = ExchangeTypes.TOPIC),
        key = "test"
    ))
    public void testListener(String msg) {
        System.out.println("消费者收到消息: " + msg);
    }
}
```

- value: @Queue 注解，用于声明队列，value 为 queueName, durable 表示队列是否持久化, autoDelete 表示没有消费者之后队列是否自动删除
- exchange: @Exchange 注解，用于声明 exchange， type 就是交换机类型.
- key: 在 topic 方式下，这个就是我们熟知的 routingKey

### 发送

```jaav

@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringAMQPTest {
 
    @Resource
    private RabbitTemplate rabbitTemplate;
 
    @Test
    public void testSendMessageSimpleQueue() {
        String queueName = "simple.queue";
        String message = "hello, spring amqp!";
        rabbitTemplate.convertAndSend(queueName, message);
    }
}
```



### 为什么建议 @Bean 创建

建议使用 @Bean 而不是@RabbitListener的原因如下：

- 使用 @Bean 创建相比于 @RabbitListener 实现了配置和业务上的解耦合（队列、交换机、绑定的创建 和 业务代码 解耦合）.
- @Bean可以更好地控制RabbitMQ监听器的创建和销毁，而@RabbitListener则需要在运行时动态创建和销毁监听器，这可能会导致一些性能问题和内存泄漏问题。
- 使用@Bean还可以更好地管理RabbitMQ连接和通道，从而提高应用程序的可靠性和性能。





# 安装和使用

## Maven引入

```xml
<!--AMQP依赖，包含RabbitMQ-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

## 基础配置

```yaml
spring:
  rabbitmq:
    host: 127.0.0.1 # 主机名
    port: 5672 # 端口
    virtual-host: / # 虚拟主机 
    username: root # 用户名
    password: root # 密码
```

## 连接重试配置

```yaml
spring:
  rabbitmq:
	template:
      retry:
        enabled: true # 开启rabbit初始化重试机制
		max-interval: 1000ms #最大重试间隔
        max-attempts: 3 #最大重试次数
        multiplier: 1 #间隔乘数
        initial-interval: 1000ms #初始化的时间间隔
```

## 异常重试配置

```yaml
# 异常重试机制设置[未加死信队列 五次异常后 直接抛弃了]
spring:
  rabbitmq:
  	listener:
  	  simple:
  	  	default-requeue-rejected: true 
#设置是否重回队列 true即出现异常会将消息重新发送到队列中
		retry:
			enabled: true # 是否开启消息重试机制,默认为false
			max-attempts: 3 # 消息重试的最大次数，默认为3
			initial-interval: 1000ms # 消息重试的初始间隔,，默认为1000ms
			multiplier: 1 # 消息重试的时间间隔倍数，默认为1s
			max-interval: 1000ms # 消息重试的最大时间间隔，默认额外1000ms
```

## 确认模式

```yaml
spring:
  rabbitmq:
  	listener:
      simple: 
      	acknowledge-mode: auto # audo 自动确认，none 不确认， manual 手动
      	prefetch: 3 # 设置每次预抓取的数量是3,处理完之前不收下一条 默认250
```





# 消息预取

## 概念

消息预取机制（Prefetch Mechanism）是RabbitMQ中用于控制消息传递给消费者的一种机制。它定义了在一个信道上，消费者允许的最大未确认的消息数量。一旦未确认的消息数量达到了设置的预取值，RabbitMQ就会停止向该消费者发送更多消息，直到至少有一条未完成的消息得到了确认。

- 问题描述：一般消费者处理消息时，会先预取消息，然后处理。此时如果存在两个消费者，消费者A的处理性能是消费者B的2倍，但是由于消息预取，他们都会取到相同的消息数量，此时消息就会继续堆积在队列里。此时消费者A已经处理完后，消费者B还有没有处理的，降低了总体处理效率。
- 解决办法：可以通过设置消息预取数量，根据处理消息的速度决定。


## Code
配置文件

```yml
spring:
  rabbitmq:
    host: 127.0.0.1
    port: 5672
    username: root
    password: root
    virtual-host: /
    listener:
      simple:
        prefetch: 1
```



