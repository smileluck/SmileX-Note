[toc]

---

# Redisson NoClassDefFoundError RedisStreamCommands

在高版本的 `redisson-spring-boot-starter`  `v3.13.*` 中，本地执行redisson锁正常，但是放到服务器上后提示异常。这是因为缺少 `RedisStreamCommands` 类，换成较低版本即可

```xml
<dependency>
    <groupId>org.redisson</groupId>
    <artifactId>redisson-spring-boot-starter</artifactId>
    <version>3.11.5</version>
 </dependency>
```

