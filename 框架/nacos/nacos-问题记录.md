[toc]

---

# 关于配置动态更新不起效果问题

```yaml
spring:
  cloud:
    nacos:
      discovery:
        # 命名空间
        namespace: dev
        # 注册中鼎
        server-addr: 127.0.0.1:8848
      config:
        # 命名空间
        namespace: dev
        # 配置中心地址
        server-addr: 127.0.0.1:8848
        # 配置文件格式
        file-extension: yml
        # 共享配置
        shared-configs:
          - application-${spring.profiles.active}.${spring.cloud.nacos.config.file-extension}
 
 # 换成这种就可以拉取，但是动态更新会异常
#           - ${spring.application.name}-${spring.profiles.active}.${spring.cloud.nacos.config.file-extension}



```

