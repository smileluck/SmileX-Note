[TOC]

---

# OpenFeign + LoadBalancer

## 依赖

```xml
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-loadbalancer</artifactId>
        </dependency>
```

## 代码实现

1. 启动类开启feignClient。@EnableFeignClient

```java
@SpringBootApplication
@EnableFeignClients
public class UserApplication {

    public static void main(String[] args) {
        SpringApplication.run(UserApplication.class, args);
    }

}    
```

2. 创建 feignClient代理类。
   
   ```java
   @FeignClient("article")
   public interface ArticleClient {
   
       @GetMapping("/article/publish")
       String articlePublish();
   
   }
   ```

3. 基于nacos实现。
   
   1. 服务提供者
      
      ```java
      @RestController
      @RequestMapping("/article")
      public class ArticleController {
      
          @GetMapping("/publish")
          public String publish() {
              System.out.println("article 被远程调用了！");
              return "success";
          }
      
      }
      ```
