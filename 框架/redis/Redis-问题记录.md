[toc]

---

# RedisConnectionException异常
> io.lettuce.core.RedisConnectionException: Unable to connect to 127.0.0.1:6379

## 方法1
追踪问题一直以为redis密码错误,最好尝试很多办法依旧没有解决，使用jedis连接却是正常的！！！

解决办法最后在redis 官网问题反馈里找到了答案：

因为使用的spring boot 高版本导致的:
```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.3.5</version>
    <relativePath/> 
</parent>
```

降低版本就行了
```xml
<dependency>
    <groupId>io.lettuce</groupId>
    <artifactId>lettuce-core</artifactId>
    <version>5.1.4.RELEASE</version>
</dependency>
```