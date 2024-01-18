##### <font size="4" color="red"><font size="4" color="red">11. Springboot集成Zipkin分布式监控</font>

**(1) 依赖包**

```xml
<!-- springboot 父节点区域 -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.1.1.RELEASE</version>
</parent>
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-zipkin</artifactId>
    </dependency>
</dependencies>
```

**(2) 主启动目录中创建资源文件夹`resources`文件夹，并创建`application.properties`文件**

`application.properties`文件中内容：

```ini
server.port=8310
#服务追踪
spring.zipkin.service.name=zipkin-test
spring.zipkin.base-url=http://127.0.0.1:9411/
spring.zipkin.discovery-client-enabled=true
spring.zipkin.sender.type=web
spring.sleuth.sampler.probability=1
```

**(3) 创建`controller`包，并创建`controller.java`文件**

`controller.java`文件中内容：

```java
import org.springframework.web.bind.annotation.CrossOrigin;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;

@CrossOrigin
@RestController
public class controller {

    @RequestMapping(value = "/index",method = RequestMethod.GET)
    public String Index1() throws Exception {
        return "成功";
    }
}
```

**(4) 主启动目录中内容**

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.context.annotation.ComponentScan;

@EnableAutoConfiguration
@ComponentScan("controller")
public class Start {

    public static void main(String[] args) {
        SpringApplication.run(Start.class, args);
    }

}
```

***

##### <font size="4" color="red">12. Eclipse安装SpringBoot插件创建项目</font>01. Springboot(2.0.2)中集成SpringBootAdmin和Eureka监控服务</font>

​		注册中心可以是`Eureka`、`Consul`、`Dubbo`等

**1.`Eureka`注册中心**

**(1) 依赖包**

```xml
<!-- springboot 父节点区域 -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.0.2.RELEASE</version>
</parent>
<!--指定项目中公有的模块-->
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>Finchley.SR2</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
        <version>2.0.2.RELEASE</version>
    </dependency>
</dependencies>
```

**(2) 主启动目录中创建资源文件夹`resources`文件夹，并创建`application.properties`文件**

`application.properties`文件中内容：

```ini
server.port=8310
spring.application.name=eurekaServer
eureka.client.fetch-registry=false
eureka.client.register-with-eureka=false
eureka.instance.hostname = localhost
eureka.client.serviceUrl.defaultZone=http://localhost:${server.port}/eureka/
#展示全部细节信息
management.endpoint.health.show-details=always
management.endpoints.web.exposure.include=*
```

**(3) 主启动目录中内容**

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;

@EnableAutoConfiguration
@EnableEurekaServer
public class Start {
    public static void main(String[] args) {
        SpringApplication.run(Start.class, args);
    }
}
```

**2.`SpringbootAdmin`服务器端**

**(1) 依赖包**

```xml
<!-- springboot 父节点区域 -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.0.2.RELEASE</version>
</parent>
<!--指定项目中公有的模块-->
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>Finchley.SR2</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    </dependency>
    <dependency>
        <groupId>de.codecentric</groupId>
        <artifactId>spring-boot-admin-starter-server</artifactId>
        <version>2.0.2</version>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-webflux</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-mail</artifactId>
    </dependency>
</dependencies>
```

**(2) 主启动目录中创建资源文件夹`resources`文件夹，并创建`application.properties`文件**

`application.properties`文件中内容：

```ini
server.port=8311
spring.application.name=adminServer
eureka.client.registryFetchIntervalSeconds=5
eureka.client.serviceUrl.defaultZone=http://admin:123456@localhost:8310/eureka/
eureka.client.healthcheck.enabled = true
eureka.instance.leaseRenewalIntervalInSeconds=10
eureka.instance.health-check-url-path=/actuator/health
#展示全部细节信息
management.endpoint.health.show-details=always
management.endpoints.web.exposure.include=*
#配置邮件提醒
spring.mail.host=smtp.163.com
#spring.mainl.port=465
spring.mail.protocol=smtps
spring.mail.username=Koaliai@163.com
spring.mail.password=JGRHETDUPHQALPHH
# to和from都要配置，否则发送邮件时会报错
spring.boot.admin.notify.mail.to=838032263@qq.com
spring.boot.admin.notify.mail.from=Koaliai@163.com
```

**(3) 主启动目录中内容**

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import de.codecentric.boot.admin.server.config.EnableAdminServer;

@EnableAutoConfiguration
@EnableAdminServer
@EnableDiscoveryClient
public class Start {

    public static void main(String[] args) {
        SpringApplication.run(Start.class, args);
    }
}
```

**3.`SpringbootAdmin`客户端**

**(1) 依赖包**

```xml
<!-- springboot 父节点区域 -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.0.2.RELEASE</version>
</parent>
<!--指定项目中公有的模块-->
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>Finchley.SR2</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-webflux</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
</dependencies>
```

**(2) 主启动目录中创建资源文件夹`resources`文件夹，并创建`application.properties`文件**

`application.properties`文件中内容：

```ini
server.port=8312
spring.application.name=adminClient
eureka.client.registryFetchIntervalSeconds=5
eureka.client.serviceUrl.defaultZone=http://admin:123456@localhost:8310/eureka/
eureka.client.healthcheck.enabled = true
eureka.instance.leaseRenewalIntervalInSeconds=10
eureka.instance.health-check-url-path=/actuator/health
#展示全部细节信息
management.endpoint.health.show-details=always
management.endpoints.web.exposure.include=*
```

**(3) 主启动目录中内容**

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

@EnableAutoConfiguration
@EnableDiscoveryClient
public class Start {

    public static void main(String[] args) {
        SpringApplication.run(Start.class, args);
    }
}
```

> **注：**
>
> (1) 依次启动`Eureka`注册中心、`SpringbootAdmin`服务器端、`SpringbootAdmin`客户端
>
> (2) 访问`SpringbootAdmin`的`Web`管理页面
>
> ```
> http://localhost:8311
> ```

***

##### <font size="4" color="red">02. Springboot(2.4.1)中集成SpringBootAdmin监控服务</font>

​		`SpringbootAdmin`是面向`springboot`的一款监控组件

**1.服务器端**

**(1) 依赖包**

```xml
<!-- springboot 父节点区域 -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.4.1</version>
</parent>
<!--指定项目中公有的模块-->
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>Camden.SR6</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
        <dependency>
            <groupId>de.codecentric</groupId>
            <artifactId>spring-boot-admin-dependencies</artifactId>
            <version>2.3.1</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>de.codecentric</groupId>
        <artifactId>spring-boot-admin-starter-server</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-webflux</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
</dependencies>
```

**(2) 主启动目录中创建资源文件夹`resources`文件夹，并创建`application.properties`文件**

`application.properties`文件中内容：

```ini
server.port=8310
spring.application.name=adminServer
management.security.enabled=false
spring.security.user.name=admin
spring.security.user.password=123456
```

**(3) 主启动目录中内容**

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import de.codecentric.boot.admin.server.config.EnableAdminServer;

@EnableAutoConfiguration
@EnableAdminServer
public class Start {

    public static void main(String[] args) {
        SpringApplication.run(Start.class, args);
    }
}
```

(4) 访问`SpringbootAdmin`的`Web`管理页面，并进行监控客户端运行状况

```
http://localhost:8310/
```

**2.客户端**

**(1) 依赖包**

```xml
<!-- springboot 父节点区域 -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.4.1</version>
</parent>
<!--指定项目中公有的模块-->
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>Camden.SR6</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
        <dependency>
            <groupId>de.codecentric</groupId>
            <artifactId>spring-boot-admin-dependencies</artifactId>
            <version>2.3.1</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>de.codecentric</groupId>
        <artifactId>spring-boot-admin-starter-client</artifactId>
        <version>2.3.1</version>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
</dependencies>
```

**(2) 主启动目录中创建资源文件夹`resources`文件夹，并创建`application.properties`文件**

`application.properties`文件中内容：

```ini
server.port=8311
spring.application.name=adminClient
#admin工程的url
spring.boot.admin.client.url=http://127.0.0.1:8310
#展示全部细节信息
management.endpoint.health.show-details=always
management.endpoints.web.exposure.include=*
#允许admin工程远程停止本应用
management.endpoint.shutdown.enabled=true
#admin工程的账号密码
spring.boot.admin.client.username=admin
spring.boot.admin.client.password=123456
```

**(3) 主启动目录中内容**

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;

@EnableAutoConfiguration
public class Start {

    public static void main(String[] args) {
        SpringApplication.run(Start.class, args);
    }
}
```

***

##### <font size="4" color="red">03. Springboot(2.0)中集成Zuul网关(Token验证)</font>

**1.`Eureka`注册中心**

**(1) 依赖包**

```xml
<!-- springboot 父节点区域 -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.0.2.RELEASE</version>
</parent>
<!--指定项目中公有的模块-->
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>Finchley.SR2</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
        <version>2.0.2.RELEASE</version>
    </dependency>
</dependencies>
```

**(2) 主启动目录中创建资源文件夹`resources`文件夹，并创建`application.properties`文件**

`application.properties`文件中内容：

```ini
server.port=8300
spring.application.name=eureka-server
eureka.client.fetch-registry=false
eureka.client.register-with-eureka=false
eureka.instance.hostname = localhost
eureka.client.serviceUrl.defaultZone=http://localhost:${server.port}/eureka/
spring.security.user.name=admin
spring.security.user.password=123456
```

**(3) 主启动目录中内容**

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;

@SpringBootApplication
@EnableEurekaServer
public class Start {

    public static void main(String[] args) {
        SpringApplication.run(Start.class, args);
    }
}
```

**2.`Zuul`路由生产者**

**(1) 依赖包**

```xml
<!-- springboot 父节点区域 -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.0.2.RELEASE</version>
</parent>
<!--指定项目中公有的模块-->
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>Finchley.SR2</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
<dependencies>
    <!-- springboot依赖包 -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-zuul</artifactId>
    </dependency>
</dependencies>
```

**(2) 主启动目录中创建资源文件夹`resources`文件夹，并创建`application.properties`文件**

`application.properties`文件中内容：

```ini
server.port=8313
spring.application.name=five
eureka.client.serviceUrl.defaultZone=http://admin:123456@localhost:8300/eureka/
eureka.client.healthcheck.enabled = true
```

**(3) 创建`controller`包，并创建`controller.java`文件**

`controller.java`文件中内容：

```java
import org.springframework.web.bind.annotation.CrossOrigin;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;

@CrossOrigin
@RestController
@RequestMapping("/five")
public class controller {

    @RequestMapping(value="/index",method = RequestMethod.GET)
    public String Index() {
        try {
            return "支付成功";
        }
        catch(Exception e) {
            return "支付失败";
        }

    }

    @GetMapping("/hello/{name}")
    public String ZuulTestFive(@PathVariable("name") String name) {
        return "hello " + name + "  this is ZuulTestFive";
    }

}
```

**(4) 主启动目录中内容**

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;
import org.springframework.context.annotation.ComponentScan;

@EnableAutoConfiguration
@EnableEurekaClient
@ComponentScan("controller")
public class Start {

    public static void main(String[] args) {
        SpringApplication.run(Start.class, args);
    }
}
```

**3.`Zuul`路由消费者**

**(1) 依赖包**

```xml
<!-- springboot 父节点区域 -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.0.2.RELEASE</version>
</parent>
<!--指定项目中公有的模块-->
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>Finchley.SR2</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
<dependencies>
    <!-- springboot依赖包 -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-zuul</artifactId>
    </dependency>
</dependencies>
```

**(2) 主启动目录中创建资源文件夹`resources`文件夹，并创建`application.properties`文件**

`application.properties`文件中内容：

```ini
server.port=8312
spring.application.name=startGateway
eureka.client.serviceUrl.defaultZone=http://admin:123456@localhost:8300/eureka/
zuul.routes.three.path= /three/**
zuul.routes.three.service-id= three
zuul.routes.three.stripPrefix= false
zuul.routes.five.path= /five/**
zuul.routes.five.service-id= five
zuul.routes.five.stripPrefix= false
eureka.client.healthcheck.enabled = true
```

**(3) 创建`intector`包，并创建`MyFilter.java`文件**

`MyFilter.java`文件中内容：

```java
import javax.servlet.http.HttpServletRequest;
import org.springframework.cloud.netflix.zuul.filters.support.FilterConstants;
import org.springframework.stereotype.Component;
import com.netflix.zuul.ZuulFilter;
import com.netflix.zuul.context.RequestContext;

@Component
public class MyFilter extends ZuulFilter {
    @Override
    public String filterType() {
        return FilterConstants.PRE_TYPE;
    }

    @Override
    public int filterOrder() {
        return 0;
    }

    @Override
    public boolean shouldFilter() {
        return true;
    }

    @Override
    public Object run(){
        RequestContext ctx = RequestContext.getCurrentContext();
        HttpServletRequest request = ctx.getRequest();
        Object accessToken = request.getParameter("token");
        if(accessToken == null) {
            ctx.setSendZuulResponse(false);
            ctx.setResponseStatusCode(401);
            try {
                ctx.getResponse().setHeader("Content-Type", "text/html;charset=UTF-8");
                ctx.getResponse().getWriter().write("登录信息为空！");
            }catch (Exception e){}
            return null;
        }
        return null;
    }
}
```

**(4) 主启动目录中内容**

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;
import org.springframework.cloud.netflix.zuul.EnableZuulProxy;
import org.springframework.context.annotation.ComponentScan;

@EnableAutoConfiguration
@EnableEurekaClient
@EnableZuulProxy
@ComponentScan("intector")
public class Start {

    public static void main(String[] args) {
        SpringApplication.run(Start.class, args);
    }

}
```

>**注：**依次启动`Eureka`注册中心、`Zuul`路由生产者、`Zuul`路由消费者，然后访问地址验证
>
>```
>地址：http://localhost:8312/five/index
>```

***

##### <font size="4" color="red">04. Springboot(2.0)中集成Zuul网关限流(Redis存储)</font>

**1.`Eureka`注册中心**

**(1) 依赖包**

```xml
<!-- springboot 父节点区域 -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.0.2.RELEASE</version>
</parent>
<!--指定项目中公有的模块-->
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>Finchley.SR2</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
        <version>2.0.2.RELEASE</version>
    </dependency>
</dependencies>
```

**(2) 主启动目录中创建资源文件夹`resources`文件夹，并创建`application.properties`文件**

`application.properties`文件中内容：

```ini
server.port=8300
spring.application.name=eureka-server
eureka.client.fetch-registry=false
eureka.client.register-with-eureka=false
eureka.instance.hostname = localhost
eureka.client.serviceUrl.defaultZone=http://localhost:${server.port}/eureka/
spring.security.user.name=admin
spring.security.user.password=123456
```

**(3) 主启动目录中内容**

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;

@SpringBootApplication
@EnableEurekaServer
public class Start {

    public static void main(String[] args) {
        SpringApplication.run(Start.class, args);
    }
}
```

**2.`Zuul`路由生产者**

**(1) 依赖包**

```xml
<!-- springboot 父节点区域 -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.0.2.RELEASE</version>
</parent>
<!--指定项目中公有的模块-->
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>Finchley.SR2</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
<dependencies>
    <!-- springboot依赖包 -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-zuul</artifactId>
    </dependency>
</dependencies>
```

**(2) 主启动目录中创建资源文件夹`resources`文件夹，并创建`application.properties`文件**

`application.properties`文件中内容：

```ini
server.port=8313
spring.application.name=five
eureka.client.serviceUrl.defaultZone=http://admin:123456@localhost:8300/eureka/
eureka.client.healthcheck.enabled = true
```

**(3) 创建`controller`包，并创建`controller.java`文件**

`controller.java`文件中内容：

```java
import org.springframework.web.bind.annotation.CrossOrigin;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;

@CrossOrigin
@RestController
@RequestMapping("/five")
public class controller {

    @RequestMapping(value="/index",method = RequestMethod.GET)
    public String Index() {
        try {
            return "支付成功";
        }
        catch(Exception e) {
            return "支付失败";
        }
    }

    @GetMapping("/hello/{name}")
    public String ZuulTestFive(@PathVariable("name") String name) {
        return "hello " + name + "  this is ZuulTestFive";
    }

}
```

**(4) 主启动目录中内容**

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;
import org.springframework.context.annotation.ComponentScan;

@EnableAutoConfiguration
@EnableEurekaClient
@ComponentScan("controller")
public class Start {

    public static void main(String[] args) {
        SpringApplication.run(Start.class, args);
    }
}
```

**3.`Zuul`路由消费者**

**(1) 依赖包**

```xml
<!-- springboot 父节点区域 -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.0.2.RELEASE</version>
</parent>
<!--指定项目中公有的模块-->
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>Finchley.SR2</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
<dependencies>
    <!-- springboot依赖包 -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-zuul</artifactId>
    </dependency>
    <!--Zuul限流的相关依赖-->
    <dependency>
        <groupId>com.marcosbarbero.cloud</groupId>
        <artifactId>spring-cloud-zuul-ratelimit</artifactId>
        <version>1.3.4.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-redis</artifactId>
    </dependency>
</dependencies>
```

**(2) 主启动目录中创建资源文件夹`resources`文件夹，并创建`application.properties`文件**

`application.properties`文件中内容：

```ini
server.port=8312
spring.application.name=startGateway
eureka.client.serviceUrl.defaultZone=http://admin:123456@localhost:8300/eureka/
zuul.routes.three.path= /three/**
zuul.routes.three.service-id= three
zuul.routes.three.stripPrefix= false
zuul.routes.five.path= /five/**
zuul.routes.five.service-id= five
zuul.routes.five.stripPrefix= false

spring.redis.database=0
spring.redis.host=192.168.0.19
spring.redis.port=8302

#开启限流
zuul.ratelimit.enabled=true
#针对five路由限流
zuul.ratelimit.repository=redis
#60s内请求超过3次，服务端就抛出异常，60s后可以恢复正常请求
zuul.ratelimit.policies.five.limit=3
#限流类型
zuul.ratelimit.policies.five.type=url
zuul.ratelimit.policies.EURKA-CLIENT1.quota=50
zuul.ratelimit.policies.five.refresh-interval=60
#针对某个IP进行限流，不影响其他IP
zuul.ratelimit.policies.five.type=origin
eureka.client.healthcheck.enabled = true

#全局配置限流
#zuul.ratelimit.enabled=true
#zuul.ratelimit.default-policy.limit=3
#zuul.ratelimit.default-policy.refresh-interval=60
#zuul.ratelimit.default-policy.type=origin
```

**(3) 创建`intector`包，并创建`ExceptionHandler.java`文件**

`ExceptionHandler.java`文件中内容：

```java
import org.springframework.boot.web.servlet.error.ErrorController;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class ExceptionHandler implements ErrorController {

    @Override
    public String getErrorPath() {

        return "error";
    }

    @RequestMapping(value="/error")
    public String error(){
        return "{\"result\":\"访问太多频繁，请稍后再访问！！！\"}";
    }

}
```

**(4) 主启动目录中内容**

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;
import org.springframework.cloud.netflix.zuul.EnableZuulProxy;
import org.springframework.context.annotation.ComponentScan;

@EnableAutoConfiguration
@EnableEurekaClient
@EnableZuulProxy
@ComponentScan("intector")
public class Start {

    public static void main(String[] args) {
        SpringApplication.run(Start.class, args);
    }
}
```

>**注：**依次启动`Eureka`注册中心、`Zuul`路由生产者、`Zuul`路由消费者，然后访问地址验证
>
>```
>地址：http://localhost:8312/five/index
>```

***

##### <font size="4" color="red">05. Springboot(2.0)集成SpringGetWay(Token验证)</font>

**1.`Eureka`注册中心**

**(1) 依赖包**

```xml
<!-- springboot 父节点区域 -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.2.0.RELEASE</version>
</parent>
<!--指定项目中公有的模块-->
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>Hoxton.SR6</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-config-server</artifactId>
    </dependency>
    <!-- 这个依赖本来是可以不加的，但是配置没办法刷新，加上它可以手动刷新配置 -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
        <version>2.2.0.RELEASE</version>
    </dependency>
</dependencies>
```

**(2) 在主启动目录`resources`资源文件夹中创建`application.properties`文件**

`application.properties`文件中内容：

```ini
server.port=8300
spring.application.name=eurekaServer
eureka.client.fetch-registry=false
eureka.client.register-with-eureka=false
eureka.instance.hostname = localhost
eureka.client.serviceUrl.defaultZone=http://localhost:${server.port}/eureka/
spring.security.user.name=admin
spring.security.user.password=123456
```

**(3) 主启动目录中内容**

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;

@EnableAutoConfiguration
@EnableEurekaServer
public class Start {
    public static void main(String[] args) {
        SpringApplication.run(Start.class, args);
    }
}
```

**2.`GetWay`路由消费者1**

**(1) 依赖包**

```xml
<!-- springboot 父节点区域 -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.2.0.RELEASE</version>
</parent>
<!--指定项目中公有的模块-->
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>Hoxton.SR1</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
</dependencies>
```

**(2) 在主启动目录`resources`资源文件夹中创建`application.properties`文件**

`application.properties`文件中内容：

```ini
server.port=8310
spring.application.name=client-test
eureka.client.healthcheck.enabled = true
eureka.client.serviceUrl.defaultZone=http://admin:123456@localhost:8300/eureka/
eureka.instance.instance-id=${spring.cloud.client.ip-address}:${server.port}
eureka.instance.prefer-ip-address=true
eureka.instance.lease-renewal-interval-in-seconds=30
eureka.instance.lease-expiration-duration-in-seconds=90
```

**(3) 创建`controller`包，并创建`controller.java`文件**

`controller.java`文件中内容：

```java
import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.CrossOrigin;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;
@CrossOrigin
@RestController
public class controller {

    @Value("${spring.cloud.client.ip-address}:${server.port}")
    private String instance_id;

    @RequestMapping(value = "/index",method = RequestMethod.GET)
    public String Index() {
        return instance_id;
    }
}
```

**(4) 主启动目录中内容**

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;
import org.springframework.context.annotation.ComponentScan;

@EnableAutoConfiguration
@EnableEurekaClient
@ComponentScan("controller")
public class Start {
    public static void main(String[] args) {
        SpringApplication.run(Start.class, args);
    }
}
```

**3.`GetWay`路由消费者**

**(1) 依赖包**

```xml
<!-- springboot 父节点区域 -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.2.0.RELEASE</version>
</parent>
<!--指定项目中公有的模块-->
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>Hoxton.SR1</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-gateway</artifactId>
        <exclusions>
            <exclusion>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-starter-web</artifactId>
            </exclusion>
        </exclusions>
    </dependency>
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>fastjson</artifactId>
        <version>1.2.74</version>
    </dependency>
    <dependency>
        <groupId>io.projectreactor.netty</groupId>
        <artifactId>reactor-netty</artifactId>
        <version>0.9.4.RELEASE</version>
    </dependency>
</dependencies>
```

**(2) 在主启动目录`resources`资源文件夹中创建`application.properties`文件**

`application.properties`文件中内容：

```ini
server.port=8312
spring.application.name=startGateway
eureka.client.healthcheck.enabled = true
eureka.client.serviceUrl.defaultZone=http://admin:123456@localhost:8300/eureka/
eureka.instance.instance-id=gateway8312
eureka.instance.prefer-ip-address=true
eureka.instance.lease-renewal-interval-in-seconds=30
eureka.instance.lease-expiration-duration-in-seconds=90

spring.cloud.gateway.discovery.locator.enabled=false
#开启小写验证，默认feign根据服务名查找都是用的全大写
spring.cloud.gateway.discovery.locator.lowerCaseServiceId=true
spring.cloud.gateway.routes[0].id=client-test
spring.cloud.gateway.routes[0].uri=lb://client-test
#spring.cloud.gateway.routes[0].predicates[0]=Path=/api/**
spring.cloud.gateway.routes[0].predicates[0]=Path=/**
spring.cloud.gateway.routes[0].filters[0]=StripPrefix=1
```

**(3) 创建`domain`包，并创建`ResultCode.java`文件**

`ResultCode.java`文件中内容：

```java
import com.fasterxml.jackson.annotation.JsonFormat;

@JsonFormat(shape = JsonFormat.Shape.OBJECT)
public enum ResultCode {
    /*
    请求返回状态码和说明信息
     */
    SUCCESS(200, "成功"),
    BAD_REQUEST(400, "参数或者语法不对"),
    UNAUTHORIZED(401, "认证失败"),
    LOGIN_ERROR(401, "登陆失败，用户名或密码无效"),
    FORBIDDEN(403, "禁止访问"),
    NOT_FOUND(404, "请求的资源不存在"),
    OPERATE_ERROR(405, "操作失败，请求操作的资源不存在"),
    TIME_OUT(408, "请求超时"),

    SERVER_ERROR(500, "服务器内部错误"),
    ;
    private int code;
    private String msg;

    ResultCode(int code, String msg) {
        this.code = code;
        this.msg = msg;
    }

    public void setCode(int code) {
        this.code = code;
    }

    public int getCode() {
        return code;
    }

    public String getMsg() {
        return msg;
    }
}
```

**(4) 在`domain`包中创建`ResultJson.json`文件**

`ResultJson.java`文件中内容：

```java
public class ResultJson {

    private int code;
    private String msg;
    private Object data;

    public static ResultJson Success() {
        return Success("");
    }

    public static ResultJson Success(Object o) {
        return new ResultJson(ResultCode.SUCCESS, o);
    }

    public static ResultJson Failure(ResultCode code) {
        return Failure(code, "");
    }

    public static ResultJson Failure(ResultCode code, Object o) {
        return new ResultJson(code, o);
    }

    public ResultJson (ResultCode resultCode) {
        setResultCode(resultCode);
    }

    public ResultJson (ResultCode resultCode,Object data) {
        setResultCode(resultCode);
        this.data = data;
    }

    public void setResultCode(ResultCode resultCode) {
        this.code = resultCode.getCode();
        this.msg = resultCode.getMsg();
    }

    public void setCode(int code) {
        this.code = code;
    }
    public int getCode() {
        return code;
    }

    public void setMsg(String msg) {
        this.msg = msg;
    }

    public String getMsg() {
        return msg;
    }
    public void setData(Object data) {
        this.data = data;
    }

    public Object getData() {
        return data;
    }

    public ResultJson(int code, String msg, Object data) {
        this.code = code;
        this.msg = msg;
        this.data = data;
    }

    @Override
    public String toString() {
        return "{" +
            "\"code\":" + code +
            ", \"msg\":\"" + msg + '\"' +
            ", \"data\":\"" + data + '\"'+
            '}';
    }
}
```

**(5) 创建`Configure`包，并创建`AuthFilter.java`文件**

`AuthFilter.java`文件中内容：

```java
import java.nio.charset.StandardCharsets;
import org.springframework.cloud.gateway.filter.GatewayFilterChain;
import org.springframework.cloud.gateway.filter.GlobalFilter;
import org.springframework.core.io.buffer.DataBuffer;
import org.springframework.http.HttpStatus;
import org.springframework.http.server.reactive.ServerHttpResponse;
import org.springframework.stereotype.Component;
import org.springframework.web.server.ServerWebExchange;
import com.alibaba.fastjson.JSON;
import domain.*;
import reactor.core.publisher.Mono;

@Component
public class AuthFilter implements GlobalFilter{
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        String token = exchange.getRequest().getHeaders().getFirst("token");
        if ("token".equals(token)) {
            return chain.filter(exchange);
        }
        ServerHttpResponse response = exchange.getResponse();

        byte[] datas = JSON.toJSONString(ResultJson.Failure(ResultCode.UNAUTHORIZED)).getBytes(StandardCharsets.UTF_8);
        DataBuffer buffer = response.bufferFactory().wrap(datas);
        response.setStatusCode(HttpStatus.UNAUTHORIZED);
        response.getHeaders().add("Content-Type", "application/json;charset=UTF-8");
        return response.writeWith(Mono.just(buffer));
    }
}
```

**(6) 主启动目录中内容**

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;

@EnableAutoConfiguration
@EnableEurekaClient
public class Start {
    public static void main(String[] args) {
        SpringApplication.run(Start.class, args);
    }
}
```

> **注：**
>
> (1) 依次启动`Eureka`注册中心、`Getway`路由生产者1、`GetWay`路由消费者
>
> (2) 启动完所有服务，测试`Web`页面
>
> ```
> 地址：http://localhost:8312/client-test/index
> ```

***

##### <font size="4" color="red">06. Springboot(2.0)集成SpringGetWay(限流)</font>

**1.`Eureka`注册中心**

**(1) 依赖包**

```xml
<!-- springboot 父节点区域 -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.2.0.RELEASE</version>
</parent>
<!--指定项目中公有的模块-->
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>Hoxton.SR6</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-config-server</artifactId>
    </dependency>
    <!-- 这个依赖本来是可以不加的，但是配置没办法刷新，加上它可以手动刷新配置 -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
        <version>2.2.0.RELEASE</version>
    </dependency>
</dependencies>
```

**(2) 在主启动目录`resources`资源文件夹中创建`application.properties`文件**

`application.properties`文件中内容：

```ini
server.port=8300
spring.application.name=eurekaServer
eureka.client.fetch-registry=false
eureka.client.register-with-eureka=false
eureka.instance.hostname = localhost
eureka.client.serviceUrl.defaultZone=http://localhost:${server.port}/eureka/
spring.security.user.name=admin
spring.security.user.password=123456
```

**(3) 主启动目录中内容**

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;

@EnableAutoConfiguration
@EnableEurekaServer
public class Start {
    public static void main(String[] args) {
        SpringApplication.run(Start.class, args);
    }
}
```

**2.`GetWay`路由消费者1**

**(1) 依赖包**

```xml
<!-- springboot 父节点区域 -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.2.0.RELEASE</version>
</parent>
<!--指定项目中公有的模块-->
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>Hoxton.SR1</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
</dependencies>
```

**(2) 在主启动目录`resources`资源文件夹中创建`application.properties`文件**

`application.properties`文件中内容：

```ini
server.port=8310
spring.application.name=client-test
eureka.client.healthcheck.enabled = true
eureka.client.serviceUrl.defaultZone=http://admin:123456@localhost:8300/eureka/
eureka.instance.instance-id=${spring.cloud.client.ip-address}:${server.port}
eureka.instance.prefer-ip-address=true
eureka.instance.lease-renewal-interval-in-seconds=30
eureka.instance.lease-expiration-duration-in-seconds=90
```

**(3) 创建`controller`包，并创建`controller.java`文件**

`controller.java`文件中内容：

```java
import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.CrossOrigin;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;
@CrossOrigin
@RestController
public class controller {

    @Value("${spring.cloud.client.ip-address}:${server.port}")
    private String instance_id;

    @RequestMapping(value = "/index",method = RequestMethod.GET)
    public String Index() {
        return instance_id;
    }
}
```

**(4) 主启动目录中内容**

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;
import org.springframework.context.annotation.ComponentScan;

@EnableAutoConfiguration
@EnableEurekaClient
@ComponentScan("controller")
public class Start {
    public static void main(String[] args) {
        SpringApplication.run(Start.class, args);
    }
}
```

**3.`GetWay`路由消费者**

**(1) 依赖包**

```xml
<!-- springboot 父节点区域 -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.2.0.RELEASE</version>
</parent>
<!--指定项目中公有的模块-->
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>Hoxton.SR1</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-gateway</artifactId>
        <exclusions>
            <exclusion>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-starter-web</artifactId>
            </exclusion>
        </exclusions>
    </dependency>
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>fastjson</artifactId>
        <version>1.2.74</version>
    </dependency>
    <dependency>
        <groupId>io.projectreactor.netty</groupId>
        <artifactId>reactor-netty</artifactId>
        <version>0.9.4.RELEASE</version>
    </dependency>
</dependencies>
```

**(2) 在主启动目录`resources`资源文件夹中创建`application.properties`文件**

`application.properties`文件中内容：

```ini
server.port=8312
spring.main.allow-bean-definition-overriding=true
spring.application.name=startGateway
eureka.client.healthcheck.enabled = true
eureka.client.serviceUrl.defaultZone=http://admin:123456@localhost:8300/eureka/
eureka.instance.instance-id=gateway8312
eureka.instance.prefer-ip-address=true
eureka.instance.lease-renewal-interval-in-seconds=30
eureka.instance.lease-expiration-duration-in-seconds=90

spring.redis.host=127.0.0.1
spring.redis.port=8002
spring.redis.database=0

spring.cloud.gateway.discovery.locator.enabled=false
#开启小写验证，默认feign根据服务名查找都是用的全大写
spring.cloud.gateway.discovery.locator.lowerCaseServiceId=true
spring.cloud.gateway.routes[0].id=client-test
spring.cloud.gateway.routes[0].uri=lb://client-test
#spring.cloud.gateway.routes[0].predicates[0]=Path=/api/**
spring.cloud.gateway.routes[0].predicates[0]=Path=/**
spring.cloud.gateway.routes[0].filters[0].name=RequestRateLimiter
spring.cloud.gateway.routes[0].filters[0].args.key-resolver=#{@ipKeyResolver}
spring.cloud.gateway.routes[0].filters[0].args.redis-rate-limiter.replenishRate=1
spring.cloud.gateway.routes[0].filters[0].args.redis-rate-limiter.burstCapacity=3
spring.cloud.gateway.routes[0].filters[1]=StripPrefix=1
```

**(3) 创建`Configure`包，并创建`Configure.java`文件**

`Configure.java`文件中内容：

```java
import org.springframework.cloud.gateway.filter.ratelimit.KeyResolver;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;
import reactor.core.publisher.Mono;

@Configuration
public class Configure {

    //	@Bean
    //    KeyResolver apiKeyResolver() {
    //            //按URL限流,即以每秒内请求数按URL分组统计，超出限流的url请求都将返回429状态
    //            return exchange -> Mono.just(exchange.getRequest().getPath().toString());
    //            }
    //
    //    @Bean
    //    KeyResolver userKeyResolver() {
    //        //按用户限流
    //        return exchange -> Mono.just(exchange.getRequest().getQueryParams().getFirst("user"));
    //    }

    @Bean
    KeyResolver ipKeyResolver() {
        //按IP来限流
        return exchange -> Mono.just(exchange.getRequest().getRemoteAddress().getHostName());
    }
}
```

**(4) 主启动目录中内容**

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;
import org.springframework.context.annotation.ComponentScan;

@EnableAutoConfiguration
@EnableEurekaClient
@ComponentScan("Configure")
public class Start {

    public static void main(String[] args) {
        SpringApplication.run(Start.class, args);
    }
}
```

> **注：**
>
> (1) 依次启动`Eureka`注册中心、`Getway`路由生产者1、`GetWay`路由消费者
>
> (2) 启动完所有服务，测试`Web`页面
>
> ```
> 地址：http://localhost:8312/client-test/index
> ```

***

##### <font size="4" color="red">07. Springboot(2.0)集成SpringGetWay(熔断降级)</font>

**1.`Eureka`注册中心**

**(1) 依赖包**

```xml
<!-- springboot 父节点区域 -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.2.0.RELEASE</version>
</parent>
<!--指定项目中公有的模块-->
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>Hoxton.SR6</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-config-server</artifactId>
    </dependency>
    <!-- 这个依赖本来是可以不加的，但是配置没办法刷新，加上它可以手动刷新配置 -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
        <version>2.2.0.RELEASE</version>
    </dependency>
</dependencies>
```

**(2) 在主启动目录`resources`资源文件夹中创建`application.properties`文件**

`application.properties`文件中内容：

```ini
server.port=8300
spring.application.name=eurekaServer
eureka.client.fetch-registry=false
eureka.client.register-with-eureka=false
eureka.instance.hostname = localhost
eureka.client.serviceUrl.defaultZone=http://localhost:${server.port}/eureka/
spring.security.user.name=admin
spring.security.user.password=123456
```

**(3) 主启动目录中内容**

```java
import org.springframework.boot.SpringApplication;import org.springframework.boot.autoconfigure.EnableAutoConfiguration;import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;@EnableAutoConfiguration@EnableEurekaServerpublic class Start {    public static void main(String[] args) {        SpringApplication.run(Start.class, args);    }}
```

**2.`GetWay`路由消费者1**

**(1) 依赖包**

```xml
<!-- springboot 父节点区域 -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.2.0.RELEASE</version>
</parent>
<!--指定项目中公有的模块-->
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>Hoxton.SR1</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
</dependencies>
```

**(2) 在主启动目录`resources`资源文件夹中创建`application.properties`文件**

`application.properties`文件中内容：

```ini
server.port=8310
spring.application.name=client-test
eureka.client.healthcheck.enabled = true
eureka.client.serviceUrl.defaultZone=http://admin:123456@localhost:8300/eureka/
eureka.instance.instance-id=${spring.cloud.client.ip-address}:${server.port}
eureka.instance.prefer-ip-address=true
eureka.instance.lease-renewal-interval-in-seconds=30
eureka.instance.lease-expiration-duration-in-seconds=90
```

**(3) 创建`controller`包，并创建`controller.java`文件**

`controller.java`文件中内容：

```java
import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.CrossOrigin;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;

@CrossOrigin
@RestController
public class controller {

    @Value("${spring.cloud.client.ip-address}:${server.port}")
    private String instance_id;

    @RequestMapping(value = "/index",method = RequestMethod.GET)
    public String Index() {
        return instance_id;
    }

    @RequestMapping(value = "/gatewayFallback",method = RequestMethod.GET)
    public String Fallback() {
        return "接口熔断";
    }
}
```

**(4) 主启动目录中内容**

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;
import org.springframework.context.annotation.ComponentScan;

@EnableAutoConfiguration
@EnableEurekaClient
@ComponentScan("controller")
public class Start {
    public static void main(String[] args) {
        SpringApplication.run(Start.class, args);
    }
}
```

**3.`GetWay`路由消费者**

**(1) 依赖包**

```xml
<!-- springboot 父节点区域 -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.2.0.RELEASE</version>
</parent>
<!--指定项目中公有的模块-->
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>Hoxton.SR1</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-gateway</artifactId>
        <exclusions>
            <exclusion>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-starter-web</artifactId>
            </exclusion>
        </exclusions>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
        <version>2.2.3.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>io.projectreactor.netty</groupId>
        <artifactId>reactor-netty</artifactId>
        <version>0.9.4.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>fastjson</artifactId>
        <version>1.2.74</version>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-redis-reactive</artifactId>
        <version>2.2.0.RELEASE</version>
    </dependency>
</dependencies>
```

**(2) 在主启动目录`resources`资源文件夹中创建`application.properties`文件**

`application.properties`文件中内容：

```ini
server.port=8312
spring.main.allow-bean-definition-overriding=true
spring.application.name=startGateway
eureka.client.healthcheck.enabled = true
eureka.client.serviceUrl.defaultZone=http://admin:123456@localhost:8300/eureka/
eureka.instance.instance-id=gateway8312
eureka.instance.prefer-ip-address=true
eureka.instance.lease-renewal-interval-in-seconds=30
eureka.instance.lease-expiration-duration-in-seconds=90

spring.cloud.gateway.discovery.locator.enabled=false
#开启小写验证，默认feign根据服务名查找都是用的全大写
spring.cloud.gateway.discovery.locator.lowerCaseServiceId=true
spring.cloud.gateway.routes[0].id=client-test
spring.cloud.gateway.routes[0].uri=lb://client-test
#spring.cloud.gateway.routes[0].predicates[0]=Path=/api/**
spring.cloud.gateway.routes[0].predicates[0]=Path=/**
#使用内置的HystrixGatewayFilterFactory工厂类做熔断降级
spring.cloud.gateway.routes[0].filters[0].name=Hystrix
# Hystrix的bean名称
spring.cloud.gateway.routes[0].filters[0].args.name=testHystrix
spring.cloud.gateway.routes[0].filters[0].args.fallbackUri=forward:/gatewayFallback
spring.cloud.gateway.routes[0].filters[1]=StripPrefix=1

hystrix.command.default.execution.isolation.strategy=SEMAPHORE
#接口超时时间为2秒
hystrix.command.default.execution.isolation.thread.timeoutInMilliseconds=2000
```

**(3) 主启动目录中内容**

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;
import org.springframework.context.annotation.ComponentScan;

@EnableAutoConfiguration
@EnableEurekaClient
@ComponentScan("Configure")
public class Start {

    public static void main(String[] args) {
        SpringApplication.run(Start.class, args);
    }
}
```

> **注：**
>
> (1) 依次启动`Eureka`注册中心、`Getway`路由生产者1、`GetWay`路由消费者
>
> (2) 启动完所有服务，测试`Web`页面
>
> ```
> 地址：http://localhost:8312/client-test/index
> ```

***

##### <font size="4" color="red">08. Springboot热部署</font>

**(1) 依赖包**

```xml
<!-- springboot 父节点区域 -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.0.2.RELEASE</version>
</parent>
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-devtools</artifactId>
        <optional>true</optional>
    </dependency>
</dependencies>
```

**(2) 主启动目录中创建资源文件夹`resources`文件夹，并创建`application.properties`文件**

`application.properties`文件中内容：

```ini
server.port=8310
#热部署生效
spring.devtools.restart.enabled= true
#设置重启的目录
#spring.devtools.restart.additional-paths: src/main/java
#classpath目录下的WEB-INF文件夹内容修改不重启
spring.devtools.restart.exclude= WEB-INF/**
```

**(3) 创建`controller`包，并创建`controller.java`文件**

`controller.java`文件中内容：

```java
import org.springframework.web.bind.annotation.CrossOrigin;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;

@CrossOrigin
@RestController
public class controller {

    @RequestMapping(value = "/login",method = RequestMethod.GET)
    public String Login() {
        return "成功";
    }

    @RequestMapping(value = "/index",method = RequestMethod.GET)
    public String Index() {

        return "成功";
    }
}
```

**(4) 主启动目录中内容**

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.context.annotation.ComponentScan;

@EnableAutoConfiguration
@ComponentScan("controller")
public class Start {

    public static void main(String[] args) {
        SpringApplication.run(Start.class, args);
    }
}
```

***

##### <font size="4" color="red">09. Springboot(2.0)中集成Mybaits和Durid操作Mysql(普通配置)</font>

**(1) 依赖包**

```xml
<!-- springboot 父节点区域 -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.0.2.RELEASE</version>
</parent>
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
    </dependency>
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>druid</artifactId>
        <version>1.0.13</version>
    </dependency>
    <dependency>
        <groupId>dom4j</groupId>
        <artifactId>dom4j</artifactId>
        <version>1.6.1</version>
    </dependency>
    <!-- Spring Boot Mybatis 依赖 -->
    <dependency>
        <groupId>org.mybatis.spring.boot</groupId>
        <artifactId>mybatis-spring-boot-starter</artifactId>
        <version>2.1.1</version>
    </dependency>
</dependencies>
```

**(2) 主启动目录中创建资源文件夹`resources`文件夹，并创建`application.properties`文件**

`application.properties`文件中内容：

```ini
server.port=8310
spring.datasource.url=jdbc:mysql://192.168.0.19:3860/test?serverTimezone=UTC&useSSL=true
spring.datasource.username=mysql
spring.datasource.password=123456
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
spring.datasource.type=com.alibaba.druid.pool.DruidDataSource
spring.datasource.druid.initial-size=10
spring.datasource.druid.max-active=50
spring.datasource.druid.min-idle=10
spring.datasource.druid.max-wait=5000
spring.datasource.druid.pool-prepared-statements=true
spring.datasource.druid.max-pool-prepared-statement-per-connection-size=20
spring.datasource.druid.validation-query=SELECT 1 FROM DUAL
spring.datasource.druid.validation-query-timeout=20000
spring.datasource.druid.test-on-borrow=false
spring.datasource.druid.test-on-return=false
spring.datasource.druid.test-while-idle=true
spring.datasource.druid.time-between-eviction-runs-millis=60000

mybatis.type-aliases-package=Entity
mybatis.mapperLocations=classpath:mapper/*.xml
```

**(3) 在资源文件夹`resources`文件夹中创建`mapper`文件夹，并创建`userMapper.xml`文件**

`userMapper.xml`文件中内容：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
<mapper namespace="Dao.UserDao">
    <resultMap id="BaseResultMap" type="Entity.User">
        <result column="id" property="id"/>
        <result column="name" property="name"/>
        <result column="age" property="age"/>
        <result column="address" property="address"/>
        <result column="create_time" jdbcType="TIMESTAMP" property="createTime" />
        <result column="update_time" jdbcType="TIMESTAMP" property="updateTime" />
    </resultMap>
    <sql id="Base_Column_List">
        id, name, age,address, create_time, update_time
    </sql>
    <select id="selectAll"  resultMap="BaseResultMap">
        select
        <include refid="Base_Column_List" />
        from user order by age
    </select>
</mapper>
```

**(4) 创建`Entity`包，并创建`User.java`文件**

`User.java`文件中内容：

```java
import java.util.Date;

public class User {

    private int id; 
    private String name;
    private int age;
    private String address;
    private Date createTime;
    private Date updateTime;

    public User() {

    }

    public User(int id, String name, int age, String address) {
        this.id = id;
        this.name = name;
        this.age = age;
        this.address = address;
    }

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public String getAddress() {
        return address;
    }

    public void setAddress(String address) {
        this.address = address;
    }

    public Date getCreateTime() {
        return createTime;
    }

    public void setCreateTime(Date createTime) {
        this.createTime = createTime;
    }

    public Date getUpdateTime() {
        return updateTime;
    }

    public void setUpdateTime(Date updateTime) {
        this.updateTime = updateTime;
    }
}
```

**(5) 创建`Dao`包，并创建`UserDao.java`文件**

`UserDao.java`文件中内容：

```java
import java.util.List;
import org.apache.ibatis.annotations.Mapper;
import Entity.*;

@Mapper
public interface UserDao {
    List<User> selectAll();
}
```

**(6) 创建`controller`包，并创建`controller.java`文件**

`controller.java`文件中内容：

```java
import java.util.List;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.CrossOrigin;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.ResponseBody;
import Dao.*;
import Entity.*;

@CrossOrigin
@Controller
public class controller {

    @Autowired
    private UserDao userDao;

    @RequestMapping(value = "/index",method = RequestMethod.GET)
    @ResponseBody
    public List<User> Index() throws Exception {
        List<User> users = userDao.selectAll();
        return users;
    }

}
```

**(7) 主启动目录中内容**

```java
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.context.annotation.ComponentScan;

@EnableAutoConfiguration
@ComponentScan("Configure,controller")
@MapperScan("Dao")
public class Start {

    public static void main(String[] args) {
        SpringApplication.run(Start.class, args);
    }
}
```

***

##### <font size="4" color="red">10. Springboot(2.0)集成Zuul路由Ribbon负载均衡</font>

**1.`Eureka`服务器端**

**(1) 依赖包**

```xml
<!-- springboot 父节点区域 -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.0.2.RELEASE</version>
</parent>
<!--指定项目中公有的模块-->
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>Finchley.SR2</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
        <version>2.0.2.RELEASE</version>
    </dependency>
</dependencies>
```

**(2) 主启动目录中创建资源文件夹`resources`文件夹，并创建`application.properties`文件**

`application.properties`文件中内容：

```ini
server.port=8310
spring.application.name=eureka-server
eureka.client.fetch-registry=false
eureka.client.register-with-eureka=false
eureka.instance.hostname = localhost
eureka.client.serviceUrl.defaultZone=http://localhost:${server.port}/eureka/
spring.security.user.name=admin
spring.security.user.password=123456
```

**(3) 主启动目录中内容**

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;

@SpringBootApplication
@EnableEurekaServer
public class Start {

    public static void main(String[] args) {
        SpringApplication.run(Start.class, args);
    }
}
```

**2.路由消费者**

**(1) 依赖包**

```xml
<!-- springboot 父节点区域 -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.0.2.RELEASE</version>
</parent>
<!--指定项目中公有的模块-->
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>Finchley.SR2</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
<dependencies>
    <!-- springboot依赖包 -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-zuul</artifactId>
    </dependency>
</dependencies>
```

**(2) 主启动目录中创建资源文件夹`resources`文件夹，并创建`application.properties`文件**

`application.properties`文件中内容：

```ini
server.port=8312
spring.application.name=startGateway
eureka.client.serviceUrl.defaultZone=http://admin:123456@localhost:8310/eureka/
zuul.routes.api.path=/api/**
zuul.routes.api.service-id=serviceA
zuul.routes.api.stripPrefix=false
eureka.client.healthcheck.enabled=true
```

**(3) 主启动目录中内容**

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.cloud.client.loadbalancer.LoadBalanced;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;
import org.springframework.cloud.netflix.zuul.EnableZuulProxy;
import org.springframework.context.annotation.Bean;
import org.springframework.web.client.RestTemplate;

@EnableAutoConfiguration
@EnableEurekaClient
@EnableZuulProxy
public class Start {

    @Bean
    @LoadBalanced   //调用方法时启用负载均衡
    public RestTemplate getRestTemplate(){
        return new RestTemplate();
    }

    public static void main(String[] args) {
        SpringApplication.run(Start.class, args);
    }

}
```

**3.路由生产者01**

**(1) 依赖包**

```xml
<!-- springboot 父节点区域 -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.0.2.RELEASE</version>
</parent>
<!--指定项目中公有的模块-->
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>Finchley.SR2</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
    </dependency>
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>druid</artifactId>
        <version>1.0.13</version>
    </dependency>
    <dependency>
        <groupId>dom4j</groupId>
        <artifactId>dom4j</artifactId>
        <version>1.6.1</version>
    </dependency>
    <dependency>
        <groupId>org.apache.shardingsphere</groupId>
        <artifactId>sharding-jdbc-spring-boot-starter</artifactId>
        <version>${sharding-sphere.version}</version>
    </dependency>
    <!-- Spring Boot Mybatis 依赖 -->
    <dependency>
        <groupId>org.mybatis.spring.boot</groupId>
        <artifactId>mybatis-spring-boot-starter</artifactId>
        <version>2.1.1</version>
    </dependency>
</dependencies>
```

**(2) 主启动目录中创建资源文件夹`resources`文件夹，并创建`application.properties`文件**

`application.properties`文件中内容：

```ini
server.port=8313
spring.application.name=serviceA
eureka.client.serviceUrl.defaultZone=http://admin:123456@localhost:8310/eureka/
eureka.client.healthcheck.enabled = true
```

**(3) 创建`controller`包，并创建`controller.java`文件**

`controller.java`文件中内容：

```java
import org.springframework.web.bind.annotation.CrossOrigin;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;

@CrossOrigin
@RestController
public class controller {

    @GetMapping("/api/{name}")
    public String Api(@PathVariable("name") String name) {
        return "hello " + name + "  this is ZuulTestFive";
    }

}
```

**(4) 主启动目录中内容**

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;
import org.springframework.context.annotation.ComponentScan;

@EnableAutoConfiguration
@EnableEurekaClient
@ComponentScan("controller")
public class Start {

    public static void main(String[] args) {
        SpringApplication.run(Start.class, args);
    }

}
```

**4.路由生产者02**

**(1) 依赖包**

```xml
<!-- springboot 父节点区域 -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.0.2.RELEASE</version>
</parent>
<!--指定项目中公有的模块-->
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>Finchley.SR2</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
    </dependency>
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>druid</artifactId>
        <version>1.0.13</version>
    </dependency>
    <dependency>
        <groupId>dom4j</groupId>
        <artifactId>dom4j</artifactId>
        <version>1.6.1</version>
    </dependency>
    <dependency>
        <groupId>org.apache.shardingsphere</groupId>
        <artifactId>sharding-jdbc-spring-boot-starter</artifactId>
        <version>${sharding-sphere.version}</version>
    </dependency>
    <!-- Spring Boot Mybatis 依赖 -->
    <dependency>
        <groupId>org.mybatis.spring.boot</groupId>
        <artifactId>mybatis-spring-boot-starter</artifactId>
        <version>2.1.1</version>
    </dependency>
</dependencies>
```

**(2) 主启动目录中创建资源文件夹`resources`文件夹，并创建`application.properties`文件**

`application.properties`文件中内容：

```ini
server.port=8314
spring.application.name=serviceA
eureka.client.serviceUrl.defaultZone=http://admin:123456@localhost:8310/eureka/
eureka.client.healthcheck.enabled = true
```

**(3) 创建`controller`包，并创建`controller.java`文件**

`controller.java`文件中内容：

```java
import org.springframework.web.bind.annotation.CrossOrigin;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;

@CrossOrigin
@RestController
public class controller {

    @GetMapping("/api/{name}")
    public String Api(@PathVariable("name") String name) {
        return "hello " + name + "  this is ZuulTestFive";
    }

}
```

**(4) 主启动目录中内容**

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;
import org.springframework.context.annotation.ComponentScan;

@EnableAutoConfiguration
@EnableEurekaClient
@ComponentScan("controller")
public class Start {

    public static void main(String[] args) {
        SpringApplication.run(Start.class, args);
    }
}
```

> **注：**
>
> ```
> (1) 启动eureka服务器端服务
> (2) 启动zuul生产者服务
> (3) 启动zuul消费者01服务
> (4) 启动zuul消费者02服务
> (5) 测试访问地址：http://localhost:8312/api/jacks
> ```

***

##### <font size="4" color="red">11. Springboot(2.0)接口重用</font>

**(1) 依赖包**

```xml
<!-- springboot 父节点区域 -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.0.2.RELEASE</version>
</parent>
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```

**(2) 创建`service`包，并创建`UserService.java`文件**

`UserService.java`文件中内容：

```java
import org.springframework.stereotype.Component;

@Component
public interface UserService {
    void printShow();
}
```

**(3) 在`service`包中创建`UserService1Impl.java`文件**

`UserService1Impl.java`文件中内容：

```java
import org.springframework.stereotype.Service;

@Service(value="userService1")
public class UserService1Impl implements UserService {

    @Override
    public void printShow() {
        System.out.println("service1输出");
    }
}
```

**(4) 在`service`包中创建`UserService2Impl.java`文件**

`UserService2Impl.java`文件中内容：

```java
import org.springframework.stereotype.Service;

@Service(value="userService2")
public class UserService2Impl implements UserService {

    @Override
    public void printShow() {
        System.out.println("service2输出");
    }
}
```

**(5) 创建`controller`包，并创建`controller.java`文件**

`controller.java`文件中内容：

```java
import org.springframework.web.bind.annotation.CrossOrigin;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;
import service.*;

@CrossOrigin
@RestController
public class controller {

    @Resource(name="userService1")
    private UserService userService1;

    @Resource(name="userService1")
    private UserService userService2;

    @RequestMapping(value = "/index",method = RequestMethod.GET)
    public String Index() {
        userService1.printShow();
         userService2.printShow();
        return "成功";
    }

}
```

**(6) 主启动目录中内容**

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.context.annotation.ComponentScan;

@EnableAutoConfiguration
@ComponentScan("controller,service")
public class Start {

    public static void main(String[] args) {
        SpringApplication.run(Start.class, args);
    }
}
```

***

##### <font size="4" color="red">12. Springboot(2.0)集成Springdata操作Redis(多数据源)(2)</font>

**(1) 依赖包**

```xml
<!-- springboot 父节点区域 -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.0.2.RELEASE</version>
</parent>
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-redis</artifactId>
    </dependency>
    <dependency>
        <groupId>redis.clients</groupId>
        <artifactId>jedis</artifactId>
        <version>2.9.0</version>
    </dependency>
</dependencies>
```

**(2) 主启动目录中创建资源文件夹`resources`文件夹，并创建`application.properties`文件**

`application.properties`文件中内容：

```ini
server.port=8310
spring.data.redis.database = 0
spring.data.redis.host = 192.168.0.19
spring.data.redis.port = 8302
spring.data.redis.password = 
spring.data.redis.timeout = 2000
spring.data.redis.max-active = 30
spring.data.redis.max-idle = 30
spring.data.redis.max-waite = 30
spring.data.redis.min-idle = 30
spring.data.redis.max-wait = 6000
```

**(3) 创建`Configure`包，并创建`Configure.java`文件**

`Configure.java`文件中内容：

```java
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.RedisStandaloneConfiguration;
import org.springframework.data.redis.connection.jedis.JedisClientConfiguration;
import org.springframework.data.redis.connection.jedis.JedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.Jackson2JsonRedisSerializer;
import org.springframework.data.redis.serializer.StringRedisSerializer;

import com.fasterxml.jackson.annotation.JsonAutoDetect;
import com.fasterxml.jackson.annotation.PropertyAccessor;
import com.fasterxml.jackson.databind.ObjectMapper;

import redis.clients.jedis.JedisPoolConfig;

@Configuration
public class Configure {

    @Value("${spring.data.redis.database}")
    private int dbIndex;

    @Value("${spring.data.redis.host}")
    private String host;

    @Value("${spring.data.redis.port}")
    private int port;

    @Value("${spring.data.redis.password}")
    private String password;

    @Value("${spring.data.redis.timeout}")
    private int timeout;

    @Value("${spring.data.redis.max-active}")
    private int redisPoolMaxActive;

    @Value("${spring.data.redis.max-wait}")
    private int redisPoolMaxWait;

    @Value("${spring.data.redis.max-idle}")
    private int redisPoolMaxIdle;

    @Value("${spring.data.redis.min-idle}")
    private int redisPoolMinIdle;

    @Bean
    public JedisConnectionFactory redisConnectionFactory() {
        RedisStandaloneConfiguration redisStandaloneConfiguration = new RedisStandaloneConfiguration();
        redisStandaloneConfiguration.setHostName(host);
        redisStandaloneConfiguration.setPort(port);
        redisStandaloneConfiguration.setDatabase(dbIndex);
        //redisStandaloneConfiguration.setPassword(password);
        JedisPoolConfig poolConfig = new JedisPoolConfig();
        poolConfig.setMaxIdle(redisPoolMaxIdle);
        poolConfig.setMinIdle(redisPoolMinIdle);
        poolConfig.setMaxTotal(redisPoolMaxActive);
        poolConfig.setMaxWaitMillis(redisPoolMaxWait);
        poolConfig.setTestOnBorrow(true);
        JedisClientConfiguration.JedisClientConfigurationBuilder jedisClientConfigurationBuilder = JedisClientConfiguration.builder();
        JedisClientConfiguration jedisClientConfiguration = jedisClientConfigurationBuilder.usePooling().poolConfig(poolConfig).build();
        JedisConnectionFactory jedisConnectionFactory = new JedisConnectionFactory(redisStandaloneConfiguration,jedisClientConfiguration);
        return jedisConnectionFactory;
    }

    @Bean(name="basicRedisTemplate")
    public RedisTemplate<Object,Object> defaultRedisTemplate() {
        RedisTemplate<Object,Object> template = new RedisTemplate<Object,Object>();
        template.setConnectionFactory(redisConnectionFactory());
        Jackson2JsonRedisSerializer<Object> jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer<Object>(Object.class);

        ObjectMapper objectMapper = new ObjectMapper();
        objectMapper.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        objectMapper.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);

        jackson2JsonRedisSerializer.setObjectMapper(objectMapper);
        // 设置value的序列化规则和 key的序列化规则
        template.setKeySerializer(new StringRedisSerializer());
        template.setValueSerializer(jackson2JsonRedisSerializer);
        template.setHashKeySerializer(new StringRedisSerializer());
        template.setHashValueSerializer(jackson2JsonRedisSerializer);
        template.afterPropertiesSet();
        return template;
    }
}
```

**(4) 创建`controller`包，并创建`controller.java`文件**

`controller.java`文件中内容：

```java
import javax.annotation.Resource;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.CrossOrigin;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.ResponseBody;

@CrossOrigin
@Controller
public class controller {

    @Resource(name="basicRedisTemplate")
    RedisTemplate<String,String> template;

    @RequestMapping(value = "/index",method = RequestMethod.GET)
    @ResponseBody
    public String Index() {
        if(!template.hasKey("key")){
            //写入内容
            template.opsForValue().set("key","字符串");
        }
        else{
            //删除数据
            template.delete("key");
        }
        return "成功";
    }

}
```

**(5) 主启动目录中内容**

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.context.annotation.ComponentScan;

@EnableAutoConfiguration
@ComponentScan("Configure,controller")
public class Start {

    public static void main(String[] args) {
        SpringApplication.run(Start.class, args);
    }
}
```

***

##### <font size="4" color="red">13. Springboot(2.0)集成Springdata操作Redis(缓存)</font>

**(1) 依赖包**

```xml
<!-- springboot 父节点区域 -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.0.2.RELEASE</version>
</parent>
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-redis</artifactId>
    </dependency>
    <dependency>
        <groupId>redis.clients</groupId>
        <artifactId>jedis</artifactId>
        <version>2.9.0</version>
    </dependency>
</dependencies>
```

**(2) 主启动目录中创建资源文件夹`resources`文件夹，并创建`application.properties`文件**

`application.properties`文件中内容：

```ini
server.port=8310

spring.redis.database=0
spring.redis.host=192.168.0.19
spring.redis.port=8302
spring.redis.timeout=2000
spring.redis.jedis.pool.max-idle=8
spring.redis.jedis.pool.max-wait=-1ms
spring.redis.jedis.pool.min-idle=0
spring.redis.jedis.pool.max-active=8
```

**(3) 创建`Configure`包，并创建`Configure.java`文件**

`Configure.java`文件中内容：

```java
import java.time.Duration;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.cache.CacheManager;
import org.springframework.cache.annotation.EnableCaching;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.cache.RedisCacheConfiguration;
import org.springframework.data.redis.cache.RedisCacheManager;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.Jackson2JsonRedisSerializer;
import org.springframework.data.redis.serializer.RedisSerializationContext;
import org.springframework.data.redis.serializer.RedisSerializer;
import org.springframework.data.redis.serializer.StringRedisSerializer;
import com.fasterxml.jackson.annotation.PropertyAccessor;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.annotation.JsonAutoDetect;

@Configuration
@EnableCaching
public class Configure {

    @Autowired
    RedisConnectionFactory redisConnectionFactory;

    @Bean
    public CacheManager cacheManager(RedisConnectionFactory factory) {
        RedisSerializer<String> redisSerializer = new StringRedisSerializer();
        Jackson2JsonRedisSerializer<Object> jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer<Object>(Object.class);

        //解决查询缓存转换异常的问题
        ObjectMapper om = new ObjectMapper();
        om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        om.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
        jackson2JsonRedisSerializer.setObjectMapper(om);
        // 配置序列化（解决乱码的问题）,过期时间30秒
        RedisCacheConfiguration config = RedisCacheConfiguration.defaultCacheConfig()
            .entryTtl(Duration.ofSeconds(1800000))
            .serializeKeysWith(RedisSerializationContext.SerializationPair.fromSerializer(redisSerializer))
            .serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(jackson2JsonRedisSerializer))
            .disableCachingNullValues();
        RedisCacheManager cacheManager = RedisCacheManager.builder(factory)
            .cacheDefaults(config)
            .build();
        return cacheManager;
    }

    @Bean
    public RedisTemplate<Object,Object> defaultRedisTemplate() {
        RedisTemplate<Object,Object> template = new RedisTemplate<Object,Object>();
        template.setConnectionFactory(redisConnectionFactory);
        Jackson2JsonRedisSerializer<Object> jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer<Object>(Object.class);
        ObjectMapper objectMapper = new ObjectMapper();
        objectMapper.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        objectMapper.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
        jackson2JsonRedisSerializer.setObjectMapper(objectMapper);
        // 设置value的序列化规则和 key的序列化规则
        template.setKeySerializer(new StringRedisSerializer());
        template.setValueSerializer(jackson2JsonRedisSerializer);
        template.setHashKeySerializer(new StringRedisSerializer());
        template.setHashValueSerializer(jackson2JsonRedisSerializer);
        template.afterPropertiesSet();
        return template;
    }
}
```

**(4) 创建`controller`包，并创建`controller.java`文件**

`controller.java`文件中内容：

```java
import org.springframework.cache.annotation.Cacheable;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.CrossOrigin;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.ResponseBody;

@CrossOrigin
@Controller
public class controller {

    @RequestMapping(value = "/index",method = RequestMethod.GET)
    @ResponseBody
    @Cacheable(value = "test", key = "#id")
    public String Index(@RequestParam("id") String  id) {
        return "成功";
    }
}
```

**(5) 主启动目录中内容**

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.context.annotation.ComponentScan;

@EnableAutoConfiguration
@ComponentScan("Configure,controller")
public class Start {

    public static void main(String[] args) {
        SpringApplication.run(Start.class, args);
    }
}
```

>**注：**删除缓存方法
>
>(1) 删除指定`key`值
>
>```java
>@Caching(evict = {
>   @CacheEvict(value = "MyRedis",key="redis")
>})
>```
>
>(2) 删除指定文件下`value`值所有`key`值
>
>```java
>@Caching(evict = {
>@CacheEvict(value = "MyRedis",allEntries=true/*表示删除MyRedis文件下所有的缓存*/)
>})
>```

****

##### <font size="4" color="red">14. Springboot(2.0)集成Springdata操作Redis(common-pool连接池)</font>

**(1) 依赖包**

```xml
<!-- springboot 父节点区域 -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.0.2.RELEASE</version>
</parent>
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-redis</artifactId>
    </dependency>
    <dependency>
        <groupId>org.apache.commons</groupId>
        <artifactId>commons-pool2</artifactId>
        <version>2.9.0</version>
    </dependency>
</dependencies>
```

**(2) 主启动目录中创建资源文件夹`resources`文件夹，并创建`application.properties`文件**

`application.properties`文件中内容：

```ini
server.port=8310
spring.redis.database=0
spring.redis.host=192.168.0.19
spring.redis.port=8302
#Redis服务器连接密码（默认为空）
#spring.redis.password=redis123
#连接池最大连接数（使用负值表示没有限制）
spring.redis.lettuce.pool.max-active=8
#连接池最大阻塞等待时间（使用负值表示没有限制）
spring.redis.lettuce.pool.max-wait=-1
#连接池中的最大空闲连接
spring.redis.lettuce.pool.max-idle=8
#连接池中的最小空闲连接
spring.redis.lettuce.pool.min-idle=0
#连接超时时间（毫秒）
spring.redis.timeout=30000
```

**(3) 创建`Configure`包，并创建`Configure.java`文件**

`Configure.java`文件中内容：

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.Jackson2JsonRedisSerializer;
import org.springframework.data.redis.serializer.StringRedisSerializer;
import com.fasterxml.jackson.annotation.PropertyAccessor;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.annotation.JsonAutoDetect;

@Configuration
public class Configure {

    @Autowired
    RedisConnectionFactory redisConnectionFactory;

    @Bean
    public RedisTemplate<Object,Object> defaultRedisTemplate() {
        RedisTemplate<Object,Object> template = new RedisTemplate<Object,Object>();
        template.setConnectionFactory(redisConnectionFactory);
        Jackson2JsonRedisSerializer<Object> jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer<Object>(Object.class);
        ObjectMapper objectMapper = new ObjectMapper();
        objectMapper.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        objectMapper.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
        jackson2JsonRedisSerializer.setObjectMapper(objectMapper);
        // 设置value的序列化规则和 key的序列化规则
        template.setKeySerializer(new StringRedisSerializer());
        template.setValueSerializer(jackson2JsonRedisSerializer);
        template.setHashKeySerializer(new StringRedisSerializer());
        template.setHashValueSerializer(jackson2JsonRedisSerializer);
        template.afterPropertiesSet();
        return template;
    }
}
```

**(4) 创建`controller`包，并创建`controller.java`文件**

`controller.java`文件中内容：

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.CrossOrigin;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.ResponseBody;

@CrossOrigin
@Controller
public class controller {

    @Autowired
    RedisTemplate<String,String> template;

    @RequestMapping(value = "/index",method = RequestMethod.GET)
    @ResponseBody
    public String Index() {
        //写入内容
        template.opsForValue().set("key","字符串");
        return "成功";
    }
}
```

**(5) 主启动目录中内容**

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.context.annotation.ComponentScan;

@EnableAutoConfiguration
@ComponentScan("Configure,controller")
public class Start {

    public static void main(String[] args) {
        SpringApplication.run(Start.class, args);
    }

}
```

***

##### <font size="4" color="red">15. Springboot中集成jasypt配置文件加密(自定义加密器)</font>

**(1) 依赖包**

```xml
<!-- springboot 父节点区域 -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.0.2.RELEASE</version>
</parent>
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>com.github.ulisesbocchio</groupId>
        <artifactId>jasypt-spring-boot-starter</artifactId>
        <version>3.0.2</version>
    </dependency>
</dependencies>
```

**(2) 主启动目录中创建资源文件`resources`，并创建`application.properties`文件**

`application.properties`文件中内容：

```ini
server.port=8310
jasypt.encryptor.bean=codeSheepEncryptorBean
spring.datasource.password=123456
jasypt.encryptor.property.prefix=CodeSheep(
jasypt.encryptor.property.suffix=)
redis.password=CodeSheep(jkKPn/BDQMyplqDQ1Nxp2JB3VlpJZGbsNe2UCTzfI38CsZ/+nV0MbSxZ/EKFr4QP)
```

**(3) 创建`Configure`包，并创建`Configure.java`文件**

`Configure.java`文件中内容：

```java
import org.jasypt.encryption.StringEncryptor;
import org.jasypt.encryption.pbe.PooledPBEStringEncryptor;
import org.jasypt.encryption.pbe.config.SimpleStringPBEConfig;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.env.Environment;

@Configuration
public class Configure {


    @Autowired
    private  ApplicationContext appCtx;

    @Autowired
    private StringEncryptor codeSheepEncryptorBean;

    @Bean(name="codeSheepEncryptorBean" )
    public StringEncryptor codesheepStringEncryptor() {
        PooledPBEStringEncryptor encryptor = new PooledPBEStringEncryptor();
        SimpleStringPBEConfig config = new SimpleStringPBEConfig();
        config.setPassword("CodeSheep");
        config.setAlgorithm("PBEWITHHMACSHA512ANDAES_256");
        config.setKeyObtentionIterations("1000");
        config.setPoolSize("1");
        config.setProviderName("SunJCE");
        config.setSaltGeneratorClassName("org.jasypt.salt.RandomSaltGenerator");
        config.setIvGeneratorClassName("org.jasypt.iv.RandomIvGenerator");
        config.setStringOutputType("base64");
        encryptor.setConfig(config);

        return encryptor;
    }

    @Bean
    public String init() {
        Environment environment = appCtx.getBean(Environment.class);
        String redisOriginPswd = environment.getProperty("redis.password");
        System.out.println(redisOriginPswd);
        //此处将原始密码加密,然后在配置文件中加ENC(<加密字符串>)
        //String redisEncryptedPswd = decrypt( redisOriginPswd);
        //System.out.println(redisEncryptedPswd);

        return "sd";
    }

    private String encrypt( String originPassord ) {
        String encryptStr = codeSheepEncryptorBean.encrypt( originPassord );
        return encryptStr;
    }

    private String decrypt( String encryptedPassword ) {
        String decryptStr = codeSheepEncryptorBean.decrypt( encryptedPassword );
        return decryptStr;
    }
}
```

**(4) 主启动目录中内容**

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.context.annotation.ComponentScan;

@EnableAutoConfiguration
@ComponentScan("Configure,controller")
public class Start {

    public static void main(String[] args) {
        SpringApplication.run(Start.class, args);
    }
}
```

> **注：**
>
> (1) 启动参数启动
>
> ```shell
> java -jar yourproject.jar --jasypt.encryptor.password=CodeSheep
> ```

***

##### <font size="4" color="red">16. Springboot(2.0)中集成mybatis(树形结构查询)</font>

**(1) 依赖包**

```xml
<!-- springboot 父节点区域 -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.0.2.RELEASE</version>
</parent>
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>druid</artifactId>
        <version>1.0.13</version>
    </dependency>
    <dependency>
        <groupId>dom4j</groupId>
        <artifactId>dom4j</artifactId>
        <version>1.6.1</version>
    </dependency>
    <!-- Spring Boot Mybatis 依赖 -->
    <dependency>
        <groupId>org.mybatis.spring.boot</groupId>
        <artifactId>mybatis-spring-boot-starter</artifactId>
        <version>2.1.1</version>
    </dependency>
</dependencies>
```

**(2) 主启动目录中创建资源文件夹`resources`，并创建`application.properties`文件**

`application.properties`文件中内容：

```ini
server.port=8310
spring.mysql.url=jdbc:mysql://127.0.0.1:3358/test?serverTimezone=UTC&useSSL=true
spring.mysql.username=mysql
spring.mysql.password=123456
spring.mysql.driver-class=com.mysql.jdbc.Driver
```

**(3) 在资源文件夹`resources`中创建`mapper`文件夹，并在`mapper`文件夹中创建`personMapper.xml`文件**

`personMapper.xml`文件中内容：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
<mapper namespace="Dao.PersonDao">
    <resultMap id="BaseTreeResultMap" type="Entity.Person">
        <result column="id" property="id"/>
        <result column="parent_id" property="parentId"/>
        <result column="name" property="name"/>
        <collection column="id" property="child" javaType="java.util.ArrayList"
                    ofType="Entity.Person" select="getNextPersonTree"/>
    </resultMap>
    <resultMap id="NextTreeResultMap" type="Entity.Person">
        <result column="id" property="id"/>
        <result column="name" property="name"/>
        <result column="parent_id" property="parentId"/>
        <collection column="id" property="child" javaType="java.util.ArrayList"
                    ofType="Entity.Person" select="getNextPersonTree"/>
    </resultMap>
    <sql id="Base_Column_List">
        id, name,parent_id
    </sql>
    <select id="getNextPersonTree" resultMap="NextTreeResultMap">
        SELECT id,name
        FROM person
        WHERE parent_id = #{id}
    </select>
    <select id="getNodeTree" resultMap="BaseTreeResultMap">
        SELECT
        <include refid="Base_Column_List"/>
        FROM person
        WHERE parent_id = 23
    </select>
</mapper>
```

**(4) 创建`Configure`包，并创建`Configure.java`文件**

`Configure.java`文件中内容：

```java
import javax.sql.DataSource;
import org.apache.ibatis.session.SqlSessionFactory;
import org.mybatis.spring.SqlSessionFactoryBean;
import org.mybatis.spring.SqlSessionTemplate;
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;
import org.springframework.core.io.support.PathMatchingResourcePatternResolver;
import com.alibaba.druid.pool.DruidDataSource;

@Configuration
@MapperScan(basePackages ="Dao")
public class Configure {


    @Value("${spring.mysql.url}")
    private String url;

    @Value("${spring.mysql.username}")
    private String username;

    @Value("${spring.mysql.password}")
    private String password;

    @Value("${spring.mysql.driver-class}")
    private String driverName;

    @Bean
    @Primary
    public DataSource basicDataSource() throws Exception {
        DruidDataSource dataSource = new DruidDataSource();
        dataSource.setUrl(url);
        dataSource.setUsername(username);
        dataSource.setPassword(password);
        dataSource.setDriverClassName(driverName);

        //连接池配置
        dataSource.setMaxActive(3000);
        dataSource.setMinIdle(5);
        dataSource.setInitialSize(5);
        dataSource.setMaxWait(60000);
        dataSource.setTimeBetweenEvictionRunsMillis(60000);
        dataSource.setMinEvictableIdleTimeMillis(300000);
        dataSource.setTestWhileIdle(true);
        dataSource.setTestOnBorrow(true);
        dataSource.setTestOnReturn(true);
        dataSource.setValidationQuery("SELECT 'x'");

        dataSource.setPoolPreparedStatements(true);
        dataSource.setMaxPoolPreparedStatementPerConnectionSize(20);
        return dataSource;
    }

    @Bean(name = "testSqlSessionFactory")
    public SqlSessionFactory testSqlSessionFactory() throws Exception {
        SqlSessionFactoryBean factoryBean = new SqlSessionFactoryBean();
        factoryBean.setDataSource(basicDataSource());
        PathMatchingResourcePatternResolver pathMatch = new PathMatchingResourcePatternResolver();
        factoryBean.setMapperLocations(pathMatch.getResources("classpath:mapper/*.xml"));
        return factoryBean.getObject();
    }

    @Bean(name = "testSqlSessionTemplate")
    public SqlSessionTemplate testSqlSessionTemplate(@Qualifier("testSqlSessionFactory") SqlSessionFactory sqlSessionFactory) {
        return new SqlSessionTemplate(sqlSessionFactory);
    }
}
```

**(5) 创建`Entity`包，并创建`Person.java`文件**

`Person.java`文件中内容：

```java
import java.util.List;

public class Person {

    private int id;
    private int parentId;
    private String name;
    //子节点
    private List<Person> child;

    public Person() {

    }

    public Person(int id, int parentId, String name, List<Person> child) {
        this.id = id;
        this.parentId = parentId;
        this.name = name;
        this.child = child;
    }

    public void setId(int id) {
        this.id = id;
    }

    public int getId() {
        return id;
    }

    public void setParentId(int parentId) {
        this.parentId = parentId;
    }

    public int getParentId() {
        return parentId;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }

    public void setChild(List<Person> child) {
        this.child = child;
    }

    public List<Person> getChild() {
        return child;
    }
}
```

**(6) 创建`Dao`包，并创建`PersonDao.java`文件**

`PersonDao.java`文件中内容：

```java
import java.util.List;
import org.apache.ibatis.annotations.Mapper;
import org.apache.ibatis.annotations.Select;
import Entity.*;

@Mapper
public interface PersonDao {

    @Select("select * from person where id = #{id}")
    Person getPerson(int id);

    //查询所有节点
    List<Person> getNodeTree();
    //查询节点
    List<Person> getNextPersonTree(int parentId);
}
```

**(7) 创建`controller`包，并创建`controller.java`文件**

`controller.java`文件中内容：

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.CrossOrigin;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.ResponseBody;
import Dao.*;

@CrossOrigin
@Controller
public class controller {

    @Autowired
    private PersonDao personDao;

    @RequestMapping(value = "/index",method = RequestMethod.GET)
    @ResponseBody
    public List<Person> Index() throws Exception {
        List<Person> list =personDao.getNextPersonTree(23);
        return list;
    }
}
```

**(8) 主启动目录中内容**

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.context.annotation.ComponentScan;

@EnableAutoConfiguration
@ComponentScan("Configure,controller")
public class Start {

	public static void main(String[] args) {
		SpringApplication.run(Start.class, args);
	}
}
```

***

##### <font size="4" color="red">17. Springboot集成mybait plus(增强型mybaits)</font>

**(1) 依赖包**

```xml
<!-- springboot 父节点区域 -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.0.2.RELEASE</version>
</parent>
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>druid</artifactId>
        <version>1.0.13</version>
    </dependency>
    <dependency>
        <groupId>dom4j</groupId>
        <artifactId>dom4j</artifactId>
        <version>1.6.1</version>
    </dependency>
    <dependency>
        <groupId>com.baomidou</groupId>
        <artifactId>mybatis-plus-boot-starter</artifactId>
        <version>3.0.4</version>
    </dependency>
</dependencies>
```

**(2) 主启动目录中创建资源文件夹`resources`，并创建`application.properties`文件**

`application.properties`文件中内容：

```ini
server.port=8310
spring.mysql.url=jdbc:mysql://127.0.0.1:3358/test?serverTimezone=UTC&useSSL=true
spring.mysql.username=mysql
spring.mysql.password=123456
spring.mysql.driver-class=com.mysql.jdbc.Driver
# 初始化大小，最小，最大
spring.mysql.initialSize=5
spring.mysql.minIdle=5
spring.mysql.maxActive=20
# 配置获取连接等待超时的时间
spring.mysql.maxWait=60000
# 配置间隔多久才进行一次检测，检测需要关闭的空闲连接，单位是毫秒
spring.mysql.timeBetweenEvictionRunsMillis=60000
# 配置一个连接在池中最小生存的时间，单位是毫秒
spring.mysql.minEvictableIdleTimeMillis=300000
# 测试连接
spring.mysql.testWhileIdle=true
spring.mysql.testOnBorrow=false
spring.mysql.testOnReturn=false
# 打开PSCache，并且指定每个连接上PSCache的大小
spring.mysql.poolPreparedStatements=true
spring.mysql.maxPoolPreparedStatementPerConnectionSize=20
# 配置监控统计拦截的filters
spring.mysql.filters=stat
# asyncInit是1.1.4中新增加的配置，如果有initialSize数量较多时，打开会加快应用启动时间
spring.mysql.asyncInit=true
```

**(3) 资源文件夹resources中创建mapper文件夹，并在mapper文件夹中创建personMapper.xml文件**

personMapper.xml文件中内容：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
<mapper namespace="Dao.PersonDao">
    <resultMap id="BaseTreeResultMap" type="Entity.Person">
        <result column="id" property="id"/>
        <result column="parent_id" property="parentId"/>
        <result column="name" property="name"/>
        <collection column="id" property="child" javaType="java.util.ArrayList"
                    ofType="Entity.Person" select="getNextPersonTree"/>
    </resultMap>
    <resultMap id="NextTreeResultMap" type="Entity.Person">
        <result column="id" property="id"/>
        <result column="name" property="name"/>
        <result column="parent_id" property="parentId"/>
        <collection column="id" property="child" javaType="java.util.ArrayList"
                    ofType="Entity.Person" select="getNextPersonTree"/>
    </resultMap>
    <sql id="Base_Column_List">
        id, name,parent_id
    </sql>
    <select id="getNextPersonTree" resultMap="NextTreeResultMap">
        SELECT id,name
        FROM person
        WHERE parent_id = #{id}
    </select>
    <select id="getNodeTree" resultMap="BaseTreeResultMap">
        SELECT
        <include refid="Base_Column_List"/>
        FROM person
        WHERE parent_id = 23
    </select>
</mapper>
```

**(4) 创建`Configure`包，并创建`Configure.java`文件**

`Configure.java`文件中内容：

```java
import javax.sql.DataSource;
import org.apache.ibatis.session.SqlSessionFactory;
import org.mybatis.spring.SqlSessionFactoryBean;
import org.mybatis.spring.SqlSessionTemplate;
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;
import org.springframework.core.io.support.PathMatchingResourcePatternResolver;
import com.alibaba.druid.pool.DruidDataSource;

@Configuration
@MapperScan(basePackages ="Dao")
public class Configure {


    @Value("${spring.mysql.url}")
    private String url;

    @Value("${spring.mysql.username}")
    private String username;

    @Value("${spring.mysql.password}")
    private String password;

    @Value("${spring.mysql.driver-class}")
    private String driverName;

    @Bean
    @Primary
    public DataSource basicDataSource() throws Exception {
        DruidDataSource dataSource = new DruidDataSource();
        dataSource.setUrl(url);
        dataSource.setUsername(username);
        dataSource.setPassword(password);
        dataSource.setDriverClassName(driverName);

        //连接池配置
        dataSource.setMaxActive(3000);
        dataSource.setMinIdle(5);
        dataSource.setInitialSize(5);
        dataSource.setMaxWait(60000);
        dataSource.setTimeBetweenEvictionRunsMillis(60000);
        dataSource.setMinEvictableIdleTimeMillis(300000);
        dataSource.setTestWhileIdle(true);
        dataSource.setTestOnBorrow(true);
        dataSource.setTestOnReturn(true);
        dataSource.setValidationQuery("SELECT 'x'");
        dataSource.setPoolPreparedStatements(true);
        dataSource.setMaxPoolPreparedStatementPerConnectionSize(20);
        return dataSource;
    }

    @Bean(name = "testSqlSessionFactory")
    public SqlSessionFactory testSqlSessionFactory() throws Exception {
        SqlSessionFactoryBean factoryBean = new SqlSessionFactoryBean();
        factoryBean.setDataSource(basicDataSource());
        PathMatchingResourcePatternResolver pathMatch = new PathMatchingResourcePatternResolver();
        factoryBean.setMapperLocations(pathMatch.getResources("classpath:mapper/*.xml"));
        return factoryBean.getObject();
    }

    @Bean(name = "testSqlSessionTemplate")
    public SqlSessionTemplate testSqlSessionTemplate(@Qualifier("testSqlSessionFactory") SqlSessionFactory sqlSessionFactory) {
        return new SqlSessionTemplate(sqlSessionFactory);
    }

}
```

**(5) 创建`Entity`包，并创建`Person.java`文件**

`Person.java`文件中内容：

```java
import java.util.List;

public class Person {

    private int id;
    private int parentId;
    private String name;
    //子节点
    private List<Person> child;

    public Person() {

    }

    public Person(int id, int parentId, String name, List<Person> child) {
        this.id = id;
        this.parentId = parentId;
        this.name = name;
        this.child = child;
    }

    public void setId(int id) {
        this.id = id;
    }

    public int getId() {
        return id;
    }

    public void setParentId(int parentId) {
        this.parentId = parentId;
    }

    public int getParentId() {
        return parentId;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }

    public void setChild(List<Person> child) {
        this.child = child;
    }

    public List<Person> getChild() {
        return child;
    }
}
```

**(6) 创建`Dao`包，并创建`PersoDao.java`文件**

`PersonDao.java`文件中内容：

```java
import java.util.List;
import org.apache.ibatis.annotations.Mapper;
import org.apache.ibatis.annotations.Select;
import Entity.*;

@Mapper
public interface PersonDao {

    @Select("select * from person where id = #{id}")
    Person getPerson(int id);

    //查询所有节点
    List<Person> getNodeTree();

    //查询节点
    List<Person> getNextPersonTree(int parentId);
}
```

**(7) 创建`controller`包，并创建`controller.java`文件**

`controller.java`文件中内容：

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.CrossOrigin;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.ResponseBody;
import Dao.*;

@CrossOrigin
@Controller
public class controller {

    @Autowired
    private PersonDao personDao;

    @RequestMapping(value = "/index",method = RequestMethod.GET)
    @ResponseBody
    public List<Person> Index() throws Exception {
        List<Person> list =personDao.getNextPersonTree(23);
        return list;
    }
}
```

**(8) 主启动目录中内容**

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.context.annotation.ComponentScan;

@EnableAutoConfiguration
@ComponentScan("Configure,controller")
public class Start {

	public static void main(String[] args) {
		SpringApplication.run(Start.class, args);
	}
}
```

***

##### <font size="4" color="red">18. Springboot集成Easyexcel(操作Excel)</font>

**(1) 依赖包**

```xml
<!-- springboot 父节点区域 -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.0.2.RELEASE</version>
</parent>
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>easyexcel</artifactId>
        <version>2.1.4</version>
    </dependency>
</dependencies>
```

**(2) 创建`Entity`包，并创建`Users.java`文件**

`Users.java`文件中内容：

```java
import com.alibaba.excel.annotation.ExcelProperty;
import com.alibaba.excel.metadata.BaseRowModel;

public class Users extends BaseRowModel {

    @ExcelProperty(value = "姓名",index = 0)
    private String name;
    @ExcelProperty(value = "地址",index = 1)
    private String address;

    public Users() {

    }

    public Users(String name, String address) {
        this.name = name;
        this.address = address;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getAddress() {
        return address;
    }

    public void setAddress(String address) {
        this.address = address;
    }
}
```

**(3) 创建`controller`包，并创建`controller.java`文件**

`controller.java`文件中内容：

```java
import java.io.IOException;
import java.util.ArrayList;
import java.util.List;

import javax.servlet.ServletOutputStream;
import javax.servlet.http.HttpServletResponse;
import org.apache.poi.util.IOUtils;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;
import com.alibaba.excel.ExcelWriter;
import com.alibaba.excel.metadata.Sheet;
import com.alibaba.excel.support.ExcelTypeEnum;
import Entity.*;

@RestController
public class controller {

    @RequestMapping(value="/index",method={RequestMethod.GET,RequestMethod.POST})
    public ResultJson Index(){
        return ResultJson.Success();
    }

    @GetMapping("/download")
    public void DownloadExcel(HttpServletResponse response) throws IOException {
        Users u1 = new Users("张三","广东");
        Users u2 = new Users("李四","四川");
        List<Users> list = new ArrayList<Users>();
        list.add(u1);
        list.add(u2);
        ServletOutputStream out = response.getOutputStream();
        @SuppressWarnings("deprecation")
        ExcelWriter writer = new ExcelWriter(out, ExcelTypeEnum.XLSX, true);
        String fileName = "测试exportExcel";
        Sheet sheet = new Sheet(1, 0,Users.class);
        //设置自适应宽度
        sheet.setAutoWidth(Boolean.TRUE);
        // 第一个 sheet 名称
        sheet.setSheetName("第一个sheet");
        writer.write(list, sheet);
        //通知浏览器以附件的形式下载处理，设置返回头要注意文件名有中文
        response.setHeader("Content-disposition", "attachment;filename=" + new String( fileName.getBytes("gb2312"), "ISO8859-1" ) + ".xlsx");
        writer.finish();
        response.setContentType("application/vnd.ms-excel");
        response.setCharacterEncoding("utf-8");
        out.flush();
        IOUtils.closeQuietly(out);
    }
}
```

**(4) 主启目录中内容**

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.context.annotation.ComponentScan;

@EnableAutoConfiguration
@ComponentScan("controller")
public class Start {

    public static void main(String[] args) {
        SpringApplication.run(Start.class, args);
    }

}
```

***

##### <font size="4" color="red">19. Springboot集成EasyPoi(操作Excel)</font>

**(1) 依赖包**

```xml
<!-- springboot 父节点区域 -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.0.2.RELEASE</version>
</parent>
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>cn.afterturn</groupId>
        <artifactId>easypoi-spring-boot-starter</artifactId>
        <version>4.1.0</version>
    </dependency>
</dependencies>
```

或

```xml
<!-- springboot 父节点区域 -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.0.2.RELEASE</version>
</parent>
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <!-- easy-poi -->
    <dependency>
        <groupId>cn.afterturn</groupId>
        <artifactId>easypoi-base</artifactId>
        <version>4.1.0</version>
    </dependency>
    <dependency>
        <groupId>cn.afterturn</groupId>
        <artifactId>easypoi-web</artifactId>
        <version>4.1.0</version>
    </dependency>
    <dependency>
        <groupId>cn.afterturn</groupId>
        <artifactId>easypoi-annotation</artifactId>
        <version>4.1.0</version>
    </dependency>
</dependencies>
```

**(2) 创建`Entity`包，并创建`Users.java`文件**

`Uers.java`文件中内容：

```java
import cn.afterturn.easypoi.excel.annotation.Excel;

public class Users {

    @Excel(name = "姓名", orderNum = "0", width = 15)
    private String name;
    @Excel(name = "地址", orderNum = "1", width = 15)
    private String address;

    public Users() {

    }

    public Users(String name, String address) {
        this.name = name;
        this.address = address;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getAddress() {
        return address;
    }

    public void setAddress(String address) {
        this.address = address;
    }
}
```

**(3) 创建`controller`包，并创建`ExcelUtils.java`文件**

`ExcelUtils.java`文件中内容：

```java
import java.io.File;
import java.io.IOException;
import java.io.InputStream;
import java.net.URLEncoder;
import java.util.List;
import java.util.Map;
import java.util.NoSuchElementException;

import javax.servlet.http.HttpServletResponse;

import org.apache.commons.lang3.StringUtils;
import org.apache.poi.ss.usermodel.Workbook;
import org.springframework.web.multipart.MultipartFile;

import cn.afterturn.easypoi.excel.ExcelExportUtil;
import cn.afterturn.easypoi.excel.ExcelImportUtil;
import cn.afterturn.easypoi.excel.entity.ExportParams;
import cn.afterturn.easypoi.excel.entity.ImportParams;
import cn.afterturn.easypoi.excel.entity.enmus.ExcelType;

public class ExcelUtils {

    /**
         * excel 导出
         *
         * @param list           数据
         * @param title          标题
         * @param sheetName      sheet名称
         * @param pojoClass      pojo类型
         * @param fileName       文件名称
         * @param isCreateHeader 是否创建表头
         * @param response
         */
    public static void exportExcel(List<?> list, String title, String sheetName, Class<?> pojoClass, String fileName,
                                   boolean isCreateHeader, HttpServletResponse response) throws IOException {
        ExportParams exportParams = new ExportParams(title, sheetName, ExcelType.XSSF);
        exportParams.setCreateHeadRows(isCreateHeader);
        defaultExport(list, pojoClass, fileName, response, exportParams);
    }

    /**
         * excel 导出
         *
         * @param list      数据
         * @param title     标题
         * @param sheetName sheet名称
         * @param pojoClass pojo类型
         * @param fileName  文件名称
         * @param response
         */
    public static void exportExcel(List<?> list, String title, String sheetName, Class<?> pojoClass, String fileName,
                                   HttpServletResponse response) throws IOException {
        defaultExport(list, pojoClass, fileName, response, new ExportParams(title, sheetName, ExcelType.XSSF));
    }

    /**
         * excel 导出
         *
         * @param list         数据
         * @param pojoClass    pojo类型
         * @param fileName     文件名称
         * @param response
         * @param exportParams 导出参数
         */
    public static void exportExcel(List<?> list, Class<?> pojoClass, String fileName, ExportParams exportParams,
                                   HttpServletResponse response) throws IOException {
        defaultExport(list, pojoClass, fileName, response, exportParams);
    }

    /**
         * excel 导出
         *
         * @param list     数据
         * @param fileName 文件名称
         * @param response
         */
    public static void exportExcel(List<Map<String, Object>> list, String fileName, HttpServletResponse response)
        throws IOException {
        defaultExport(list, fileName, response);
    }

    /**
         * 默认的 excel 导出
         *
         * @param list         数据
         * @param pojoClass    pojo类型
         * @param fileName     文件名称
         * @param response
         * @param exportParams 导出参数
         */
    private static void defaultExport(List<?> list, Class<?> pojoClass, String fileName, HttpServletResponse response,
                                      ExportParams exportParams) throws IOException {
        Workbook workbook = ExcelExportUtil.exportExcel(exportParams, pojoClass, list);
        downLoadExcel(fileName, response, workbook);
    }

    /**
         * 默认的 excel 导出
         *
         * @param list     数据
         * @param fileName 文件名称
         * @param response
         */
    private static void defaultExport(List<Map<String, Object>> list, String fileName, HttpServletResponse response)
        throws IOException {
        Workbook workbook = ExcelExportUtil.exportExcel(list, ExcelType.HSSF);
        downLoadExcel(fileName, response, workbook);
    }

    /**
         * 下载
         *
         * @param fileName 文件名称
         * @param response
         * @param workbook excel数据
         */
    private static void downLoadExcel(String fileName, HttpServletResponse response, Workbook workbook)
        throws IOException {
        try {
            response.setCharacterEncoding("UTF-8");
            response.setHeader("content-Type", "application/vnd.ms-excel");
            response.setHeader("Content-Disposition", "attachment;filename="
                               + URLEncoder.encode(fileName + "." + ExcelTypeEnum.XLSX.getValue(), "UTF-8"));
            workbook.write(response.getOutputStream());
        } catch (Exception e) {
            throw new IOException(e.getMessage());
        }
    }

    /**
         * excel 导入
         *
         * @param filePath   excel文件路径
         * @param titleRows  标题行
         * @param headerRows 表头行
         * @param pojoClass  pojo类型
         * @param <T>
         * @return
         */
    public static <T> List<T> importExcel(String filePath, Integer titleRows, Integer headerRows, Class<T> pojoClass)
        throws IOException {
        if (StringUtils.isBlank(filePath)) {
            return null;
        }
        ImportParams params = new ImportParams();
        params.setTitleRows(titleRows);
        params.setHeadRows(headerRows);
        params.setNeedSave(true);
        params.setSaveUrl("/excel/");
        try {
            return ExcelImportUtil.importExcel(new File(filePath), pojoClass, params);
        } catch (NoSuchElementException e) {
            throw new IOException("模板不能为空");
        } catch (Exception e) {
            throw new IOException(e.getMessage());
        }
    }

    /**
         * excel 导入
         *
         * @param file      excel文件
         * @param pojoClass pojo类型
         * @param <T>
         * @return
         */
    public static <T> List<T> importExcel(MultipartFile file, Class<T> pojoClass) throws IOException {
        return importExcel(file, 1, 1, pojoClass);
    }

    /**
         * excel 导入
         *
         * @param file       excel文件
         * @param titleRows  标题行
         * @param headerRows 表头行
         * @param pojoClass  pojo类型
         * @param <T>
         * @return
         */
    public static <T> List<T> importExcel(MultipartFile file, Integer titleRows, Integer headerRows, Class<T> pojoClass)
        throws IOException {
        return importExcel(file, titleRows, headerRows, false, pojoClass);
    }

    /**
         * excel 导入
         *
         * @param file       上传的文件
         * @param titleRows  标题行
         * @param headerRows 表头行
         * @param needVerfiy 是否检验excel内容
         * @param pojoClass  pojo类型
         * @param <T>
         * @return
         */
    public static <T> List<T> importExcel(MultipartFile file, Integer titleRows, Integer headerRows, boolean needVerfiy,
                                          Class<T> pojoClass) throws IOException {
        if (file == null) {
            return null;
        }
        try {
            return importExcel(file.getInputStream(), titleRows, headerRows, needVerfiy, pojoClass);
        } catch (Exception e) {
            throw new IOException(e.getMessage());
        }
    }

    /**
         * excel 导入
         *
         * @param inputStream 文件输入流
         * @param titleRows   标题行
         * @param headerRows  表头行
         * @param needVerfiy  是否检验excel内容
         * @param pojoClass   pojo类型
         * @param <T>
         * @return
         */
    public static <T> List<T> importExcel(InputStream inputStream, Integer titleRows, Integer headerRows,
                                          boolean needVerify, Class<T> pojoClass) throws IOException {
        if (inputStream == null) {
            return null;
        }
        ImportParams params = new ImportParams();
        params.setTitleRows(titleRows);
        params.setHeadRows(headerRows);
        params.setSaveUrl("/excel/");
        params.setNeedSave(true);
        params.setNeedVerify(needVerify);
        try {
            return ExcelImportUtil.importExcel(inputStream, pojoClass, params);
        } catch (NoSuchElementException e) {
            throw new IOException("excel文件不能为空");
        } catch (Exception e) {
            throw new IOException(e.getMessage());
        }
    }

    /**
     * Excel 类型枚举
     */
    enum ExcelTypeEnum {
        XLS("xls"), XLSX("xlsx");
        private String value;

        ExcelTypeEnum(String value) {
            this.value = value;
        }

        public String getValue() {
            return value;
        }

        public void setValue(String value) {
            this.value = value;
        }
    }

}
```

**(4) 在`controller`包中创建`controller.java`文件**

`controller.java`文件中内容：

```java
import java.io.IOException;
import java.util.ArrayList;
import java.util.List;
import javax.servlet.http.HttpServletResponse;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.multipart.MultipartFile;
import Entity.*;

@RestController
public class controller {

    @RequestMapping(value="/index",method={RequestMethod.GET,RequestMethod.POST})
    public ResultJson Index(){
        return ResultJson.Success();
    }

    @GetMapping("/export")
    public void ExportExcel(HttpServletResponse response) throws IOException {
        Users u1 = new Users("张三","广东");
        Users u2 = new Users("李四","四川");
        List<Users> list = new ArrayList<Users>();
        list.add(u1);
        list.add(u2);
        ExcelUtils.exportExcel(list, "员工信息", "员工信息sheet", Users.class, "员工信息表", response);
    }

    @PostMapping("/import")
    public List<Users> ImportExcel(@RequestParam("file") MultipartFile file) throws IOException {
        List<Users> list = ExcelUtils.importExcel(file, Users.class);
        return list;
    }
}
```

**(5) 主启动目录中内容**

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.context.annotation.ComponentScan;

@EnableAutoConfiguration
@ComponentScan("controller")
public class Start {

    public static void main(String[] args) {
        SpringApplication.run(Start.class, args);
    }

}
```

***

##### <font size="4" color="red">20. Springboot集成Minio文件系统</font>

**(1) 依赖包**

```xml
<!-- springboot 父节点区域 -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.0.2.RELEASE</version>
</parent>
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>io.minio</groupId>
        <artifactId>minio</artifactId>
        <version>7.0.2</version>
    </dependency>
</dependencies>
```

**(2) 主启动目录中创建资源文件夹`resources`，并创建application.properties文件**

application.properties文件中内容：

```ini
server.port=8310
spring.minio.url = http://127.0.0.1:3580
spring.minio.accessKey = minioadmin
spring.minio.secretKey = minioadmin
```

**(3) 创建`Configure`包，并创建`Configure.java`文件**

`Configure.java`文件中内容：

```java
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import io.minio.MinioClient;

@Configuration
public class Configure {


    @Value("${spring.minio.url}")
    private String url;
    @Value("${spring.minio.accessKey}")
    private String accessKey;
    @Value("${spring.minio.secretKey}")
    private String secretKey;

    @Bean
    public MinioClient  minioClient() {
        //初始化minio客户端
        MinioClient minioClient = null;
        try {
            minioClient = new MinioClient(url, accessKey, secretKey); 
        }
        catch(Exception e) {
            minioClient = null;
        }
        return minioClient;
    }
}
```

**(4) 创建`controller`包，并创建`controller.java`文件**

`controller.java`文件中内容：

```java
import java.io.InputStream;
import java.net.URLEncoder;
import javax.servlet.http.HttpServletResponse;
import org.apache.tomcat.util.http.fileupload.IOUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.multipart.MultipartFile;
import io.minio.MinioClient;
import io.minio.ObjectStat;
import io.minio.PutObjectOptions;

@RestController
public class controller {


    @Autowired
    private MinioClient minioClient;

    @RequestMapping(value="/upload",method={RequestMethod.GET,RequestMethod.POST})
    public String Upload(@RequestParam("file") MultipartFile file) throws Exception {
        InputStream is = file.getInputStream();
        // 文件名
        final String fileName = file.getOriginalFilename();
        // 把文件放到minio的test桶里面
        minioClient.putObject("test", fileName, is, new PutObjectOptions(is.available(), -1));
        return "成功";
    }


    @RequestMapping(value="/download",method={RequestMethod.GET,RequestMethod.POST})
    public void Download(@RequestParam("fileName") String fileName, HttpServletResponse response) throws Exception {
        InputStream in = null;
        ObjectStat stat = minioClient.statObject("test", fileName);
        response.setContentType(stat.contentType());
        response.setHeader("Content-Disposition", "attachment;filename=" + URLEncoder.encode(fileName, "UTF-8"));
        in = minioClient.getObject("test", fileName);
        IOUtils.copy(in, response.getOutputStream());
        in.close();
    }


    @RequestMapping(value="/delete",method={RequestMethod.GET,RequestMethod.POST})
    public String Delete(@RequestParam("fileName") String fileName) throws Exception {
        minioClient.removeObject("test", fileName);
        return "成功";
    }
}
```

**(5) 主启动目录中内容**

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.context.annotation.ComponentScan;

@EnableAutoConfiguration
@ComponentScan("Configure,controller")
public class Start {

    public static void main(String[] args) {
        SpringApplication.run(Start.class, args);
    }
}
```

***

##### <font size="4" color="red">21. Springboot集成限流访问(2)</font>

**(1) 依赖包**

```xml
<!-- springboot 父节点区域 -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.3.4.RELEASE</version>
</parent>
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-cache</artifactId>
    </dependency>
    <dependency>
        <groupId>com.google.guava</groupId>
        <artifactId>guava</artifactId>
        <version>19.0</version>
    </dependency>	
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-aspects</artifactId>
    </dependency>
</dependencies>
```

**(2) 创建`aop`包，并创建`Limit.java`文件**

`Limit.java`文件中内容：

```java
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Limit {

    //资源名称
    String name() default "";

    // 限制每秒访问次数，默认最大即不限制
    double perSecond() default Double.MAX_VALUE;

}
```

**(3) 在`aop`包中创建`LimitAspect.java`文件**

`LimitAspect.java`文件中内容：

```java
import java.lang.reflect.Method;
import java.util.concurrent.TimeUnit;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Pointcut;
import org.aspectj.lang.reflect.MethodSignature;
import org.springframework.stereotype.Component;
import com.google.common.util.concurrent.RateLimiter;

@Aspect
@Component
public class LimitAspect {

    RateLimiter rateLimiter = RateLimiter.create(Double.MAX_VALUE);

    @Pointcut("@annotation(aop.Limit)") //包名和限制标记
    public void pointcut() {
    }

    @Around("pointcut()")
    public Object around(ProceedingJoinPoint point) throws Throwable {
        //获取目标方法
        MethodSignature signature = (MethodSignature) point.getSignature();
        Method method = signature.getMethod();
        Limit limit = method.getAnnotation(Limit.class);
        rateLimiter.setRate(limit.perSecond());
        // 获取令牌桶中的一个令牌，最多等待1秒
        if (rateLimiter.tryAcquire(1, 1, TimeUnit.SECONDS)) {
            return point.proceed();
        } else {
            throw new LimitAccessException("网络异常，请稍后重试！");
        }
    }
}
```

**(4) 在`aop`包中创建`LimitAccessException.java`文件**

`LimitAccessException.java`文件中内容：

```java
public class LimitAccessException extends RuntimeException  {

    private static final long serialVersionUID = -3608667856397125671L;

    public LimitAccessException(String message) {
        super(message);
    }
}
```

**(5) 创建`Configure`包，并创建`Configure.java`文件**

`Configure.java`文件中内容：

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.EnableAspectJAutoProxy;
import aop.*;

@Configuration
@EnableAspectJAutoProxy
@ComponentScan(basePackages = {"aop"})
public class Configure {

    @Bean
    public LimitAspect requestLimitAop() {
        return new LimitAspect();
    }

}
```

**(6) 创建`controller`包，并创建`controller.java`文件**

`controller.java`文件中内容：

```java
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;
import aop.*;

@RestController
public class controller {

    @RequestMapping(value="/index",method={RequestMethod.GET,RequestMethod.POST})
    @Limit(name = "测试限流每秒3个请求", perSecond = 1)
    public String Index() throws Exception {
        return "ok";
    }
}
```

**(7) 创建`Exceptions`包，并创建`MyExceptionHandler.java`文件**

`MyExceptionHandler.java`文件中内容：

```java
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;
import aop.*;

@ControllerAdvice
public class MyExceptionHandler {

    @ExceptionHandler(value = LimitAccessException.class)//指定拦截的异常
    public void errorHandler(HttpServletRequest request, HttpServletResponse response,Exception e) throws Exception{

        response.getWriter().write("xsfjslkjskjfsd");
        response.getWriter().flush();
    }

}
```

**(8) 主启动目录中内容**

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.context.annotation.ComponentScan;

@EnableAutoConfiguration
@ComponentScan("Configure,controller,Exceptions")
public class Start {

    public static void main(String[] args) {
        SpringApplication.run(Start.class, args);
    }
}
```

***

##### <font size="4" color="red">22. Springboot(2.0)中集成Netty(服务器端)(2)</font>

**(1) 依赖包**

```xml
<!-- springboot 父节点区域 -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.0.2.RELEASE</version>
</parent>
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>io.netty</groupId>
        <artifactId>netty-all</artifactId>
        <version>4.1.30.Final</version>
    </dependency>
</dependencies>
```

**(2) 主启动目录中创建资源文件夹`resources`，并创建`application.properties`文件**

`application.properties`文件中内容：

```ini
server.port=8310
netty.port = 8080
```

**(3) 创建`netty`包，并创建`NettyServer.java`文件**

`NettyServer.java`文件中内容：

```java
import javax.annotation.PostConstruct;
import javax.annotation.PreDestroy;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

import io.netty.bootstrap.ServerBootstrap;
import io.netty.channel.ChannelFuture;
import io.netty.channel.ChannelOption;
import io.netty.channel.EventLoopGroup;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.nio.NioServerSocketChannel;

@Component
public class NettyServer {
	
	@Value("${netty.port}")
	private int nettyPort;
	

    private EventLoopGroup bossGroup = new NioEventLoopGroup();
    private EventLoopGroup workGroup = new NioEventLoopGroup();
    /**
     * 启动netty服务
     * @throws InterruptedException
     */
    @PostConstruct
    public void start() throws InterruptedException {
        ServerBootstrap b=new ServerBootstrap();
        b.group(bossGroup,workGroup)
                .channel(NioServerSocketChannel.class)
                .option(ChannelOption.SO_BACKLOG,128)
                .childHandler(new NettyChannel());
        ChannelFuture future = b.bind(nettyPort).sync();
        if (future.isSuccess()) {
            System.out.println("启动 Netty 成功");
        }
    }
    /**
     * 销毁
     */
    @PreDestroy
    public void destroy() {
        bossGroup.shutdownGracefully().syncUninterruptibly();
        workGroup.shutdownGracefully().syncUninterruptibly();
        System.out.println("关闭 Netty 成功");
    }
}
```

> **注：**
>
> (1) `@PostConstruct`修饰的方法会在服务器加载Servlet的时候运行，并且只会被服务器调用一次，类似于`Servlet`的`inti()`方法。
>
> (2) `@PreDestroy`修饰的方法会在服务器卸载`Servlet`的时候运行，并且只会被服务器调用一次，类似于`Servlet`的`destroy()`方法。

**(4) 在`netty`包中创建`NettyChannel.java`文件**

`NettyChannel.java`文件中内容：

```java
import io.netty.channel.ChannelInitializer;
import io.netty.channel.socket.SocketChannel;

public class NettyChannel extends ChannelInitializer<SocketChannel>{

    @Override
    protected void initChannel(SocketChannel ch) throws Exception {
        ch.pipeline().addLast(new NettyHandler());
    }
}
```

**(4) 在`netty`包中创建`NettyHandler.java`文件**

`NettyHandler.java`文件中内容：

```java
import io.netty.buffer.ByteBuf;
import io.netty.buffer.ByteBufUtil;
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.SimpleChannelInboundHandler;

public class NettyHandler  extends SimpleChannelInboundHandler<ByteBuf> {

	@Override
	protected void channelRead0(ChannelHandlerContext ctx, ByteBuf buf) throws Exception {
         //转十六进制
		String str = ByteBufUtil.hexDump(buf).toLowerCase();
		System.out.println(str);
	}
}
```

**(5) 主启动目录中内容**

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.context.annotation.ComponentScan;

@EnableAutoConfiguration
@ComponentScan("netty")
public class Start {

    public static void main(String[] args) {
        SpringApplication.run(Start.class, args);
    }
}
```

***

##### <font size="4" color="red">23. Springboot(2.0)中集成Netty(服务器端)(3)</font>

**(1) 依赖包**

```xml
<!-- springboot 父节点区域 -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.4.1</version>
</parent>
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>io.netty</groupId>
        <artifactId>netty-all</artifactId>
        <version>4.1.30.Final</version>
    </dependency>
</dependencies>
```

**(2) 主启动目录中创建资源文件夹`resources`，并创建`application.properties`文件**

`application.properties`文件中内容：

```ini
server.port=8310
netty.port = 8080
```

**(3) 创建`netty`包，并创建`NettyServer.java`文件**

`NettyServer.java`文件中内容：

```java
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;
import io.netty.bootstrap.ServerBootstrap;
import io.netty.channel.ChannelFuture;
import io.netty.channel.ChannelOption;
import io.netty.channel.EventLoopGroup;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.nio.NioServerSocketChannel;

@Component
public class NettyServer {

    @Value("${netty.port}")
    private int nettyPort;


    private EventLoopGroup bossGroup = new NioEventLoopGroup();
    private EventLoopGroup workGroup = new NioEventLoopGroup();

    public void start() {
        ServerBootstrap b=new ServerBootstrap();
        try {
            b.group(bossGroup,workGroup)
                .channel(NioServerSocketChannel.class)
                .option(ChannelOption.SO_BACKLOG,128)
                .childHandler(new NettyChannel());
            ChannelFuture future = b.bind(nettyPort).sync();
            /*等待端口关闭*/
            future.channel().closeFuture().sync();

        }
        catch(Exception e) {
            e.printStackTrace();
        }
        finally{
            bossGroup.shutdownGracefully();
            workGroup.shutdownGracefully();	
        }
    }
}
```

**(4) 在`netty`包中创建`NettyChannel.java`文件**

`NettyChannel.java`文件中内容：

```java
import org.springframework.stereotype.Component;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.socket.SocketChannel;

@Component
public class NettyChannel extends ChannelInitializer<SocketChannel>{

    @Override
    protected void initChannel(SocketChannel ch) throws Exception {
        ch.pipeline().addLast(new NettyHandler());
    }
}
```

**(4) 在`netty`包中创建`NettyHandler.java`文件**

`NettyHandler.java`文件中内容：

```java
import io.netty.buffer.ByteBuf;
import io.netty.buffer.ByteBufUtil;
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.SimpleChannelInboundHandler;

public class NettyHandler  extends SimpleChannelInboundHandler<ByteBuf> {

	@Override
	protected void channelRead0(ChannelHandlerContext ctx, ByteBuf buf) throws Exception {
        //转十六进制
		String str = ByteBufUtil.hexDump(buf).toLowerCase();
		System.out.println(str);
	}
}
```

**(5) 主启动目录中内容**

```java
import javax.annotation.Resource;
import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.context.annotation.ComponentScan;
import netty.*;

@EnableAutoConfiguration
@ComponentScan("netty")

public class Start implements CommandLineRunner{

    @Resource
    private NettyServer nettyServer;

    public static void main(String[] args) {
        SpringApplication.run(Start.class, args);
    }
    @Override
    public void run(String... args) throws Exception {
        // 开启服务
        nettyServer.start();
    }
}
```

>**注：**`@Resource`对象只初始化一次

****

##### <font size="4" color="red">24. Springboot(2.0)中集成Netty(消息服务器)</font>

**1.服务器端**

**(1) 依赖包**

```xml
<!-- springboot 父节点区域 -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.4.1</version>
</parent>
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>io.netty</groupId>
        <artifactId>netty-all</artifactId>
        <version>4.1.30.Final</version>
    </dependency>
</dependencies>
```

**(2) 主启动目录中创建资源文件夹`resources`，并创建`application.properties`文件**

`application.properties`文件中内容：

```ini
server.port=8310
netty.port = 8080
```

**(3) 创建`netty`包，并创建`NettyServer.java`文件**

`NettyServer.java`文件中内容：

```java
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;
import io.netty.bootstrap.ServerBootstrap;
import io.netty.channel.ChannelFuture;
import io.netty.channel.ChannelOption;
import io.netty.channel.EventLoopGroup;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.nio.NioServerSocketChannel;

@Component
public class NettyServer {

    @Value("${netty.port}")
    private int nettyPort;

    private EventLoopGroup bossGroup = new NioEventLoopGroup();
    private EventLoopGroup workGroup = new NioEventLoopGroup();

    public void start() {
        ServerBootstrap b=new ServerBootstrap();
        try {
            b.group(bossGroup,workGroup)
                .channel(NioServerSocketChannel.class)
                .option(ChannelOption.SO_BACKLOG,128)
                .childHandler(new NettyChannel());
            ChannelFuture future = b.bind(nettyPort).sync();
            System.out.println("启动");
            /*等待端口关闭*/
            future.channel().closeFuture().sync();

        }
        catch(Exception e) {
            e.printStackTrace();
        }
        finally{
            bossGroup.shutdownGracefully();
            workGroup.shutdownGracefully();	
        }
    }
}
```

**(4) 在`netty`包中创建`NettyChannel.java`文件**

`NettyChannel.java`文件中内容：

```java
import io.netty.channel.ChannelInitializer;
import io.netty.channel.socket.SocketChannel;
import io.netty.handler.codec.string.StringDecoder;
import io.netty.handler.codec.string.StringEncoder;
import io.netty.util.CharsetUtil;

public class NettyChannel extends ChannelInitializer<SocketChannel>{

    @Override
    protected void initChannel(SocketChannel ch) throws Exception {
        ch.pipeline().addLast(new StringDecoder(CharsetUtil.UTF_8))
                     .addLast(new StringEncoder(CharsetUtil.UTF_8))
                     .addLast(new NettyHandler());
    }

}
```

**(5) 在`netty`包中创建`NettyHandler.java`文件**

`NettyHandler.java`文件中内容：

```java
import io.netty.buffer.ByteBuf;
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.SimpleChannelInboundHandler;

public class NettyHandler  extends SimpleChannelInboundHandler<ByteBuf> {

    @Override
    protected void channelRead0(ChannelHandlerContext ctx, ByteBuf msg) throws Exception {
        ByteBuf buf = (ByteBuf) msg;
        byte[] req = new byte[buf.readableBytes()];
        buf.readBytes(req);
        String body = new String(req,"UTF-8");
        System.out.println(body);
    }

    public void exceptionCaught(ChannelHandlerContext ctx,Throwable cause) throws Exception{
        ctx.close();
    }

}
```

**(6) 主启动目录中内容**

```java
import javax.annotation.Resource;

import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.context.annotation.ComponentScan;
import netty.*;

@EnableAutoConfiguration
@ComponentScan("netty")

public class Start implements CommandLineRunner{

    @Resource
    private NettyServer nettyServer;

    public static void main(String[] args) {
        SpringApplication.run(Start.class, args);
    }

    @Override
    public void run(String... args) throws Exception {
        // 开启服务
        nettyServer.start();
    }

}
```

**2.客户端**

**(1) 依赖包**

```xml
<!-- springboot 父节点区域 -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.4.1</version>
</parent>
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>io.netty</groupId>
        <artifactId>netty-all</artifactId>
        <version>4.1.30.Final</version>
    </dependency>
</dependencies>
```

**(2) 主启动目录中创建资源文件夹`resources`，并创建`application.properties`文件**

`application.properties`文件中内容：

```ini
server.port=8310
netty.host=127.0.0.1
netty.port = 8080
```

**(3) 创建`netty`包，并创建`NettyClient.java`文件**

`NettyClient.java`文件中内容：

```java
import javax.annotation.PostConstruct;
import javax.annotation.PreDestroy;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;
import io.netty.bootstrap.Bootstrap;
import io.netty.channel.ChannelFuture;
import io.netty.channel.ChannelOption;
import io.netty.channel.EventLoopGroup;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.nio.NioSocketChannel;

@Component
public class NettyClient {

    @Value("${netty.host}")
    private String nettyHost;

    @Value("${netty.port}")
    private int nettyPort;


    private EventLoopGroup workGroup = new NioEventLoopGroup();

    @PostConstruct
    public void start() throws InterruptedException {
        Bootstrap b = new Bootstrap();
        b.group(workGroup)
            .channel(NioSocketChannel.class)
            .option(ChannelOption.TCP_NODELAY,true)
            .option(ChannelOption.SO_BACKLOG,128)
            .handler(new NettyChannel());
        ChannelFuture f = b.connect(nettyHost,nettyPort).sync();
        System.out.println("启动");
        /*等待端口关闭*/
        f.channel().closeFuture().sync();
        if (f.isSuccess()) {
            System.out.println("启动 Netty 成功");
        }    
    }

    @PreDestroy
    public void destroy() {
        workGroup.shutdownGracefully().syncUninterruptibly();
        System.out.println("关闭 Netty 成功");
    }
}
```

**(4) 在`netty`包中创建`NettyChannel.java`文件**

`NettyChannel.java`文件中内容：

```java
import io.netty.channel.ChannelInitializer;
import io.netty.channel.socket.SocketChannel;
import io.netty.handler.codec.string.StringDecoder;
import io.netty.handler.codec.string.StringEncoder;
import io.netty.util.CharsetUtil;

public class NettyChannel extends ChannelInitializer<SocketChannel>{

    @Override
    protected void initChannel(SocketChannel ch) throws Exception {
        ch.pipeline().addLast(new StringDecoder(CharsetUtil.UTF_8))
            .addLast(new StringEncoder(CharsetUtil.UTF_8))
            .addLast(new NettyHandler());
    }
}
```

**(5) 在`netty`包中创建`NettyHandler.java`文件**

`NettyHandler.java`文件中内容：

```java
import io.netty.buffer.ByteBuf;
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.SimpleChannelInboundHandler;

public class NettyHandler  extends SimpleChannelInboundHandler<ByteBuf> {

    @Override
    protected void channelRead0(ChannelHandlerContext ctx, ByteBuf msg) throws Exception {
        ByteBuf buf = (ByteBuf) msg;
        byte[] req = new byte[buf.readableBytes()];
        buf.readBytes(req);
        String body = new String(req,"UTF-8");
        System.out.println(body);
    }

    public void exceptionCaught(ChannelHandlerContext ctx,Throwable cause) throws Exception{
        ctx.close();
    }

}
```

**(6) 主启动目录中内容**

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.context.annotation.ComponentScan;

@EnableAutoConfiguration
@ComponentScan("netty")
public class Start {
    public static void main(String[] args) {
        SpringApplication.run(Start.class, args);
    }
}
```

***

##### <font size="4" color="red">25. Springboot(2.0)中集成Netty中Websocket服务</font>

**(1) 依赖包**

```xml
<!-- springboot 父节点区域 -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.4.1</version>
</parent>
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>io.netty</groupId>
        <artifactId>netty-all</artifactId>
        <version>4.1.30.Final</version>
    </dependency>
</dependencies>
```

**(2) 主启动目录中创建资源文件夹`resources`，并创建`application.properties`文件**

`application.properties`文件中内容：

```ini
server.port=8310
netty.port = 8080
```

**(3) 创建`netty`包，并创建`NettyServer.java`文件**

`NettyServer.java`文件中内容：

```java
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.SimpleChannelInboundHandler;
import io.netty.channel.group.ChannelGroup;
import io.netty.channel.group.DefaultChannelGroup;
import io.netty.handler.codec.http.websocketx.TextWebSocketFrame;
import io.netty.util.concurrent.GlobalEventExecutor;

public class NettyHandler extends SimpleChannelInboundHandler<TextWebSocketFrame> {

    public static ChannelGroup channelGroup;

    static {
        channelGroup = new DefaultChannelGroup(GlobalEventExecutor.INSTANCE);
    }

    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
        // 添加到channelGroup通道组
        channelGroup.add(ctx.channel());
        System.out.println("与客户端建立连接，通道开启！");
    }

    // 客户端与服务器关闭连接的时候触发
    @Override
    public void channelInactive(ChannelHandlerContext ctx) throws Exception {
        System.out.println("与客户端建立连接，通道关闭！");
        channelGroup.remove(ctx.channel());
    }

    @Override
    protected void channelRead0(ChannelHandlerContext ctx, TextWebSocketFrame msg) throws Exception {

        String message = msg.text() + " --- 你好，" + ctx.channel().localAddress() + " 给固定的人发消息";
        ctx.channel().writeAndFlush(new TextWebSocketFrame(message));
        //群发消息
        message = "我是服务器，这里发送的是群消息";
        channelGroup.writeAndFlush(new TextWebSocketFrame(message));
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx,Throwable cause) throws Exception{
        ctx.close();
    }
}
```

**(4) 在`netty`包中创建`NettyChannel.java`文件**

`NettyChannel.java`文件中内容：

```java
import io.netty.channel.ChannelInitializer;
import io.netty.channel.socket.SocketChannel;
import io.netty.handler.codec.http.HttpObjectAggregator;
import io.netty.handler.codec.http.HttpServerCodec;
import io.netty.handler.codec.http.websocketx.WebSocketServerProtocolHandler;
import io.netty.handler.stream.ChunkedWriteHandler;

public class NettyChannel extends ChannelInitializer<SocketChannel>{

    @Override
    protected void initChannel(SocketChannel ch) throws Exception {
        ch.pipeline().addLast(new HttpServerCodec())
            .addLast(new ChunkedWriteHandler())
            .addLast(new HttpObjectAggregator(65536 * 10000))
            .addLast(new WebSocketServerProtocolHandler("/ws", "WebSocket", true, 65536 * 10))
            .addLast(new NettyHandler());
    }
}
```

**(5) 在`netty`包中创建`NettyHandler.java`文件**

`NettyHandler.java`文件中内容：

```java
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.SimpleChannelInboundHandler;
import io.netty.channel.group.ChannelGroup;
import io.netty.channel.group.DefaultChannelGroup;
import io.netty.handler.codec.http.websocketx.TextWebSocketFrame;
import io.netty.util.concurrent.GlobalEventExecutor;

public class NettyHandler extends SimpleChannelInboundHandler<TextWebSocketFrame> {

    public static ChannelGroup channelGroup;

    static {
        channelGroup = new DefaultChannelGroup(GlobalEventExecutor.INSTANCE);
    }

    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
        // 添加到channelGroup通道组
        channelGroup.add(ctx.channel());
        System.out.println("与客户端建立连接，通道开启！");
    }

    // 客户端与服务器关闭连接的时候触发
    @Override
    public void channelInactive(ChannelHandlerContext ctx) throws Exception {
        System.out.println("与客户端建立连接，通道关闭！");
        channelGroup.remove(ctx.channel());
    }

    @Override
    protected void channelRead0(ChannelHandlerContext ctx, TextWebSocketFrame msg) throws Exception {

        String message = msg.text() + " --- 你好，" + ctx.channel().localAddress() + " 给固定的人发消息";
        ctx.channel().writeAndFlush(new TextWebSocketFrame(message));
        //群发消息
        message = "我是服务器，这里发送的是群消息";
        channelGroup.writeAndFlush(new TextWebSocketFrame(message));
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx,Throwable cause) throws Exception{
        ctx.close();
    }
}
```

**(6) 主启动目录中内容**

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.context.annotation.ComponentScan;

@EnableAutoConfiguration
@ComponentScan("netty")
public class Start {

    public static void main(String[] args) {
        SpringApplication.run(Start.class, args);
    }

}
```

***

##### <font size="4" color="red">26. Springboot中集成WebSocket(2)</font>

**(1) 依赖包**

```xml
<!-- springboot 父节点区域 -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.4.1</version>
</parent>
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>  
        <groupId>org.springframework.boot</groupId>  
        <artifactId>spring-boot-starter-websocket</artifactId>  
    </dependency> 
</dependencies>
```

**(2) 创建`Configure`包，并创建`Configure.java`文件**

`Configure.java`文件中内容：

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.socket.server.standard.ServerEndpointExporter;

@Configuration
public class Configure {
    
    @Bean  
    public ServerEndpointExporter serverEndpointExporter() {  
        return new ServerEndpointExporter();  
    }  
}
```

**(3) 创建`controller`包，并创建`controller.java`文件**

`controller.java`文件中内容：

```java
import java.util.concurrent.ConcurrentHashMap;
import javax.websocket.OnClose;
import javax.websocket.OnError;
import javax.websocket.OnMessage;
import javax.websocket.OnOpen;
import javax.websocket.Session;
import javax.websocket.server.PathParam;
import javax.websocket.server.ServerEndpoint;
import org.springframework.stereotype.Component;

@ServerEndpoint("/websocket/{userId}")
@Component
public class controller {

    private static ConcurrentHashMap<String,Session> webSocketMap = new ConcurrentHashMap<>();

    @OnOpen
    public void onOpen(Session session,@PathParam("userId") String userId){
        System.out.println(userId);

        System.out.println("websocket打开");
    }

    @OnMessage
    public void onMessage(String message,Session session){
        System.out.println("接收：" + message);
        try{
            session.getBasicRemote().sendText("字符串");
        }
        catch(Exception e){

        }
    }

    @OnError
    public void onError(Session session,Throwable error){
        error.printStackTrace();
    }

    @OnClose
    public void onClose(Session session){
        System.out.print("关闭");
    }

}
```

> **注：**`Websocket`访问地址`ws://127.0.0.1:8310/websocket/<id值>`

**(4) 主启动目录中内容**

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.context.annotation.ComponentScan;

@EnableAutoConfiguration
@ComponentScan("Configure,controller")
public class Start {

    public static void main(String[] args) {
        SpringApplication.run(Start.class, args);
    }
}
```

***

##### <font size="4" color="red">27. Springboot中集成线程池(2)</font>

**(1) 依赖包**

```xml
<!-- springboot 父节点区域 -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.4.1</version>
</parent>
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>  
        <groupId>org.springframework.boot</groupId>  
        <artifactId>spring-boot-starter-websocket</artifactId>  
    </dependency> 
</dependencies>
```

**(2) 创建`Configure`包，并创建`Configure.java`文件**

`Configure.java`文件中内容：

```java
import java.util.concurrent.ThreadPoolExecutor;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.task.TaskExecutor;
import org.springframework.scheduling.annotation.EnableAsync;
import org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor;

@Configuration
@EnableAsync
public class Configure {

    @Bean
    public TaskExecutor taskExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        // 设置核心线程数
        executor.setCorePoolSize(5);
        // 设置最大线程数
        executor.setMaxPoolSize(10);
        // 设置队列容量
        executor.setQueueCapacity(20);
        // 设置线程活跃时间（秒）
        executor.setKeepAliveSeconds(60);
        // 设置默认线程名称
        executor.setThreadNamePrefix("hello-");
        // 设置拒绝策略
        executor.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());
        // 等待所有任务结束后再关闭线程池
        executor.setWaitForTasksToCompleteOnShutdown(true);
        return executor;
    }
}
```

**(3) 创建`service`包，并创建`UserService.java`文件**

`UserService.java`文件中内容：

```java
import org.springframework.stereotype.Component;

@Component
public interface UserService {
    String sayHello();
}
```

**(4) 在`service`包中创建`UserServiceImpl.java`文件**

`UserServiceImpl.java`文件中内容：

```java
import org.slf4j.LoggerFactory;
import org.springframework.scheduling.annotation.Async;
import org.springframework.stereotype.Component;

@Component
public class UserServiceImpl implements UserService {

    @Async
    public String sayHello() {
        LoggerFactory.getLogger(UserServiceImpl.class).info( ":Hello World!");
        return "hello";
    }
}
```

**(5) 创建`controller`包，并创建`controller.java`文件**

controller.java文件中内容：

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;
import service.*;

@RestController
public class controller {

    @Autowired
    private UserService userService;

    @RequestMapping(value = "/index",method = RequestMethod.GET)
    public String Index() {

        return userService.sayHello();
    }
}
```

**(6) 主启动目录中内容**

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.context.annotation.ComponentScan;

@EnableAutoConfiguration
@ComponentScan("Configure,service,controller")
public class Start {

    public static void main(String[] args) {
        SpringApplication.run(Start.class, args);
    }
    
}
```

***

##### <font size="4" color="red">28. Springboot中集成Sentinel降级和限流</font>

**1.自定义异常回复**

**(1) 依赖包**

```xml
<!-- springboot 父节点区域 -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.0.2.RELEASE</version>
</parent>
<!--指定项目中公有的模块-->
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>Finchley.SR2</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
        <version>2.0.2.RELEASE</version>
    </dependency>
</dependencies>
```

**(2) 主启动目录中创建资源文件夹`resources`文件夹，并创建`application.properties`文件**

`application.properties`文件中内容：

```ini
server.port=8310
spring.cloud.sentinel.transport.dashboard=localhost:8080
spring.cloud.sentinel.transport.eager= true
```

> **注：**`Sentinel`采用延迟加载，只有在主动发起一次请求后，才会被拦截并发送给服务端。如果想关闭这个延迟，需将`eager`设置为`true`

**(3) 创建`Configure`包，并创建`ResponseData.java`文件**

`ResponseData.java`文件中内容：

```java
public class ResponseData {
    private int code;
    private String message;

    public ResponseData(int code, String message) {
        this.code = code;
        this.message = message;
    }

    public int getCode() {
        return code;
    }

    public void setCode(int code) {
        this.code = code;
    }

    public String getMessage() {
        return message;
    }

    public void setMessage(String message) {
        this.message = message;
    }
}
```

**(4) 在`Configure`包中创建`SentinelException.java`文件**

`SentinelException.java`文件中内容：

```java
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import org.springframework.stereotype.Component;
import com.alibaba.csp.sentinel.adapter.spring.webmvc.callback.BlockExceptionHandler;
import com.alibaba.csp.sentinel.slots.block.BlockException;
import com.alibaba.csp.sentinel.slots.block.authority.AuthorityException;
import com.alibaba.csp.sentinel.slots.block.degrade.DegradeException;
import com.alibaba.csp.sentinel.slots.block.flow.FlowException;
import com.alibaba.csp.sentinel.slots.block.flow.param.ParamFlowException;
import com.alibaba.csp.sentinel.slots.system.SystemBlockException;
import com.alibaba.fastjson.JSON;

@Component
public class SentinelException implements BlockExceptionHandler {

    @Override
    public void handle(HttpServletRequest request, HttpServletResponse response, BlockException e) throws Exception {
        response.setContentType("application/json;charset=utf-8");
        ResponseData data = null;
        if (e instanceof FlowException) {
            data = new ResponseData(-1, "流控规则被触发......");
        } else if (e instanceof DegradeException) {
            data = new ResponseData(-2, "降级规则被触发...");
        } else if (e instanceof AuthorityException) {
            data = new ResponseData(-3, "授权规则被触发...");
        } else if (e instanceof ParamFlowException) {
            data = new ResponseData(-4, "热点规则被触发...");
        } else if (e instanceof SystemBlockException) {
            data = new ResponseData(-5, "系统规则被触发...");
        }
        response.getWriter().write(JSON.toJSONString(data));
    }
}
```

**(5) 创建`controller`包，并创建`controller.java`文件**

`controller.java`文件中内容：

```java
import org.springframework.web.bind.annotation.CrossOrigin;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;

@CrossOrigin
@RestController
public class controller {
    
    @RequestMapping(value = "/index",method = RequestMethod.GET)
    public String Index() {
        return "成功";
    }
}
```

**(6) 主启动目录中内容**

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.context.annotation.ComponentScan;

@EnableAutoConfiguration
@ComponentScan("Configure,controller")
public class Start {

	public static void main(String[] args) {
		
		SpringApplication.run(Start.class, args);
	}
}
```

> **注：**需要在`Sentinel`管理界面设置相应的限流和降级过后才能触发

**2.全局异常回复**

**(1) 依赖包**

```xml
<!-- springboot 父节点区域 -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.0.2.RELEASE</version>
</parent>
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
        <version>2.0.2.RELEASE</version>
    </dependency>
</dependencies>
```

**(2) 主启动目录中创建资源文件夹`resources`文件夹，并创建`application.properties`文件**

`application.properties`文件中内容：

```ini
server.port=8310
spring.cloud.sentinel.transport.dashboard=localhost:8080
spring.cloud.sentinel.transport.eager= true
```

> **注：**`Sentinel`采用延迟加载，只有在主动发起一次请求后，才会被拦截并发送给服务端。如果想关闭这个延迟，需将`eager`设置为`true`

**(3) 创建`Configure`包，并创建`ResponseData.java`文件**

`ResponseData.java`文件中内容：

```java
public class ResponseData {
    private int code;
    private String message;

    public ResponseData(int code, String message) {
        this.code = code;
        this.message = message;
    }

    public int getCode() {
        return code;
    }

    public void setCode(int code) {
        this.code = code;
    }

    public String getMessage() {
        return message;
    }

    public void setMessage(String message) {
        this.message = message;
    }
}
```

**(4) 在`Configure`包中创建`ExcpetionHandler.java`文件**

`ExcpetionHandler.java`文件中内容：

```java
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;
import com.alibaba.csp.sentinel.slots.block.flow.param.ParamFlowException;

@RestControllerAdvice
public class ExcpetionHandler {

    @ExceptionHandler(ParamFlowException.class)
    public ResponseData test() {
        return new ResponseData(-4, "热点规则被触发...");
    }
}
```

**(5) 创建`controller`包，并创建`controller.java`文件**

`controller.java`文件中内容：

```java
import org.springframework.web.bind.annotation.CrossOrigin;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;

@CrossOrigin
@RestController
public class controller {
    
    @RequestMapping(value = "/index",method = RequestMethod.GET)
    public String Index() {
        return "成功";
    }
}
```

**(6) 主启动目录中内容**

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.context.annotation.ComponentScan;

@EnableAutoConfiguration
@ComponentScan("Configure,controller")
public class Start {

	public static void main(String[] args) {
		
		SpringApplication.run(Start.class, args);
	}
}
```

> **注：**需要在`Sentinel`管理界面设置相应的限流和降级过后才能触发

***

##### <font size="4" color="red">29. Springboot中集成Mybaits和Netty服务</font>

**(1) 依赖包**

```xml
<!-- springboot 父节点区域 -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.0.2.RELEASE</version>
</parent>
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>io.netty</groupId>
        <artifactId>netty-all</artifactId>
        <version>4.1.30.Final</version>
    </dependency>
    <!--mybatis-->
    <dependency>
        <groupId>org.mybatis.spring.boot</groupId>
        <artifactId>mybatis-spring-boot-starter</artifactId>
        <version>1.3.2</version>
    </dependency>
    <!--mysql-->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
    </dependency>
    <!-- druid的starter -->
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>druid-spring-boot-starter</artifactId>
        <version>1.1.9</version>
    </dependency>
</dependencies>
```

**(2) 主启动目录中创建资源文件夹`resources`文件夹，并创建`application.properties`文件**

`application.properties`文件中内容：

```ini
server.port=8310
netty.port=8010

spring.datasource.type=com.alibaba.druid.pool.DruidDataSource
spring.datasource.url=jdbc:mysql://192.168.0.19:3860/test?useUnicode=true&characterEncoding=utf-8&useSSL=false
spring.datasource.username=mysql
spring.datasource.password=123456
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
spring.datasource.druid.initial-size=5
spring.datasource.druid.min-idle=5
spring.datasource.druid.max-active=20
spring.datasource.druid.max-wait=60000
spring.datasource.druid.time-between-eviction-runs-millis= 60000
spring.datasource.druid.min-evictable-idle-time-millis= 30000
spring.datasource.druid.validation-query= SELECT 1 FROM DUAL
spring.datasource.druid.test-while-idle= true
spring.datasource.druid.test-on-borrow= true
spring.datasource.druid.test-on-return= false
spring.datasource.druid.pool-prepared-statements= true
spring.datasource.druid.max-pool-prepared-statement-per-connection-size=20
mybatis.mapper-locations=classpath:mapper/*.xml
mybatis.type-aliases-package=Entity
```

**(3) 在资源文件夹`resources`中创建`mapper`文件夹，并在`mapper`文件夹中创建`AccountMapper.xml`文件**

`AccountMapper.xml`文件中内容：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
<mapper namespace="Dao.AccountDao">
    <resultMap id="accountList" type="Entity.Account">
        <result column="id" property="id"/>
        <result column="name" property="name"/>
        <result column="balance" property="balance"/>
    </resultMap>
    <select id="findAll" resultMap="accountList">
        select * from tbl_account
    </select>
</mapper>
```

**(4) 创建`Entity`包，并创建`Account.java`文件**

`Account.java`文件中内容：

````java
import org.springframework.stereotype.Component;

@Component
public class Account {

    private int id; 
    private String name;
    private float balance;

    public Account() {

    }

    public Account(int id, String name, float balance) {
        this.id = id;
        this.name = name;
        this.balance = balance;
    }

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public float getBalance() {
        return balance;
    }

    public void setBalance(float balance) {
        this.balance = balance;
    }

    @Override
    public String toString() {
        return "Account [id=" + id + ", name=" + name + ", balance=" + balance + "]";
    }
}
````

**(5) 创建`Dao`包，并创建`AccountDao.java`文件**

`AccountDao.java`文件中内容：

```java
import java.util.List;
import org.springframework.stereotype.Repository;
import Entity.*;

@Repository
public interface AccountDao {

    public List<Account> findAll();

}
```

**(6) 创建`netty`包，并创建`NettyServer.java`文件**

`NettyServer.java`文件中内容：

````java
import javax.annotation.PostConstruct;
import javax.annotation.PreDestroy;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;
import Dao.*;
import io.netty.bootstrap.ServerBootstrap;
import io.netty.channel.ChannelFuture;
import io.netty.channel.ChannelOption;
import io.netty.channel.EventLoopGroup;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.nio.NioServerSocketChannel;

@Component
public class NettyServer {

    @Value("${netty.port}")
    private int nettyPort;

    @Autowired
    private AccountDao accountDao;

    private EventLoopGroup bossGroup = new NioEventLoopGroup();
    private EventLoopGroup workGroup = new NioEventLoopGroup();
    /**
     * 启动netty服务
     * @throws InterruptedException
     */
    @PostConstruct
    public void start() throws InterruptedException {
        ServerBootstrap b=new ServerBootstrap();
        b.group(bossGroup,workGroup)
            .channel(NioServerSocketChannel.class)
            .option(ChannelOption.SO_BACKLOG,128)
            .childHandler(new NettyChannel(accountDao));
        ChannelFuture future = b.bind(nettyPort).sync();
        if (future.isSuccess()) {
            System.out.println("启动 Netty 成功");
        }
    }
    /**
     * 销毁
     */
    @PreDestroy
    public void destroy() {
        bossGroup.shutdownGracefully().syncUninterruptibly();
        workGroup.shutdownGracefully().syncUninterruptibly();
        System.out.println("关闭 Netty 成功");
    }
}
````

**(7) 在`netty`包中创建`NettyChannel.java`文件**

`NettyChannel.java`文件中内容：

```java
import org.springframework.stereotype.Component;
import Dao.*;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.socket.SocketChannel;

@Component
public class NettyChannel extends ChannelInitializer<SocketChannel>{

    private AccountDao accountDao;
    public NettyChannel(AccountDao accountDao) {
        this.accountDao = accountDao;
    }

    @Override
    protected void initChannel(SocketChannel ch) throws Exception {

        ch.pipeline().addLast(new NettyHandler(accountDao));
    }
}
```

**(8) 在`netty`包中创建`NettyHandler.java`文件**

`NettyHandler.java`文件中内容：

```java
import java.util.List;
import org.springframework.stereotype.Component;
import Dao.*;
import Entity.*;
import io.netty.buffer.ByteBuf;
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.SimpleChannelInboundHandler;
import io.netty.util.CharsetUtil;

@Component
public class NettyHandler  extends SimpleChannelInboundHandler<ByteBuf> {

    private AccountDao accountDao;

    public NettyHandler(AccountDao accountDao) {
        this.accountDao=accountDao;
    }


    @Override
    protected void channelRead0(ChannelHandlerContext ctx, ByteBuf buf) throws Exception {
        ByteBuf msg=(ByteBuf) buf;
        System.out.println(msg.toString(CharsetUtil.UTF_8));
        List<Account> users = accountDao.findAll();
        users.forEach(user-> System.out.println(user.toString()));
    }

    public void exceptionCaught(ChannelHandlerContext ctx,Throwable cause) throws Exception{
        cause.printStackTrace();
        ctx.close();
    }

}
```

**(9) 主启动目录中内容**

```java
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.context.annotation.ComponentScan;

@EnableAutoConfiguration
@MapperScan(basePackages ="Dao")
@ComponentScan("netty")
public class Start {

    public static void main(String[] args) {
        SpringApplication.run(Start.class, args);
    }
}
```

****

##### <font size="4" color="red">30. Springboot(2.4)中集成Springdata操作Mongodb(多数据源)(1)</font>

**(1) 依赖包**

```xml
<!-- springboot 父节点区域 -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.4.1</version>
</parent>
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-mongodb</artifactId>
    </dependency>
</dependencies>
```

**(2) 主启动目录中创建资源文件夹`resources`文件夹，并创建`application.properties`文件**

`application.properties`文件中内容：

```ini
server.port=8310
spring.data.mongodb.url=mongodb://192.168.0.19:8092
spring.data.mongodb.database=test
```

**(3) 创建`Configure`包，并创建`Configure.java`文件**

`Configure.java`文件中内容：

```java
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;
import org.springframework.data.mongodb.core.SimpleMongoClientDatabaseFactory;
import org.springframework.data.mongodb.repository.config.EnableMongoRepositories;
import com.mongodb.ConnectionString;
import com.mongodb.MongoClientSettings;
import com.mongodb.client.MongoClient;
import com.mongodb.client.MongoClients;

@Configuration
@EnableMongoRepositories(basePackages = {"Dao"})
public class Configure {

    @Value("${spring.data.mongodb.url}")
    private String url;

    @Value("${spring.data.mongodb.database}")
    private String database;

    @Bean
    @Primary
    public SimpleMongoClientDatabaseFactory mongoDbFactory() throws Exception{
        ConnectionString connectionString = new ConnectionString(url);
        MongoClientSettings settings = MongoClientSettings.builder()
            .applyConnectionString(connectionString)
            .retryWrites(true)
            .build();
        MongoClient mongClient = MongoClients.create(settings);
        return new SimpleMongoClientDatabaseFactory(mongClient,database);
    }
}
```

**(4) 创建`Entity`包，并创建`User.java`文件**

`User.java`文件中内容：

```java
import org.springframework.data.mongodb.core.mapping.Document;

@Document(collection = "user")
public class User {
    private String userName;
    public User(){

    }

    public User(String userName){
        this.userName = userName;
    }

    public void setUserName(String userName){
        this.userName = userName;
    }
    public String getUserName(){
        return userName;
    }

    @Override
    public String toString() {
        return "User [userName=" + userName + "]";
    }
}
```

**(5) 创建`Dao`包，并创建`userDao.java`文件**

`userDao.java`文件中内容：

```java
import org.bson.types.ObjectId;
import org.springframework.data.mongodb.repository.MongoRepository;
import org.springframework.stereotype.Repository;
import Entity.*;

@Repository
public interface userDao extends MongoRepository<User,ObjectId>{

}
```

**(6) 创建`controller`包，并创建`controller.java`文件**

`controller.java`文件中内容：

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.CrossOrigin;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;
import Dao.*;
import Entity.*;

@CrossOrigin
@RestController
public class controller {

    @Autowired
    userDao userRepository;

    @RequestMapping(value="/index",method = {RequestMethod.GET})
    public String Index() {
        User user = new User("张三");
        userRepository.save(user);
        return "成功";
    }
}
```

**(7) 主启动目录中内容**

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.boot.autoconfigure.mongo.MongoAutoConfiguration;
import org.springframework.context.annotation.ComponentScan;

@EnableAutoConfiguration(exclude={MongoAutoConfiguration.class})
@ComponentScan("Configure,controller")
public class Start {

    public static void main(String[] args) {
        SpringApplication.run(Start.class, args);
    }
}
```

***

##### <font size="4" color="red">31. Springboot(2.4)中集成Springdata操作Mongodb(多数据源)(2)</font>

**(1) 依赖包**

```xml
<!-- springboot 父节点区域 -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.4.1</version>
</parent>
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-mongodb</artifactId>
    </dependency>
</dependencies>
```

**(2) 主启动目录中创建资源文件夹`resources`文件夹，并创建`application.properties`文件**

`application.properties`文件中内容：

```ini
server.port=8310
spring.data.mongodb.url=mongodb://192.168.0.19:8092
spring.data.mongodb.database=test
```

**(3) 创建`Configure`包，并创建`Configure.java`文件**

`Configure.java`文件中内容：

```java
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;
import org.springframework.data.mongodb.core.MongoTemplate;
import org.springframework.data.mongodb.core.SimpleMongoClientDatabaseFactory;
import com.mongodb.ConnectionString;
import com.mongodb.MongoClientSettings;
import com.mongodb.client.MongoClient;
import com.mongodb.client.MongoClients;

@Configuration
public class Configure {

    @Value("${spring.data.mongodb.url}")
    private String url;


    @Value("${spring.data.mongodb.database}")
    private String database;

    @Primary
    public SimpleMongoClientDatabaseFactory mongoDbFactory() throws Exception{
        ConnectionString connectionString = new ConnectionString(url);
        MongoClientSettings settings = MongoClientSettings.builder()
            .applyConnectionString(connectionString)
            .retryWrites(true)
            .build();
        MongoClient mongClient = MongoClients.create(settings);
        return new SimpleMongoClientDatabaseFactory(mongClient,database);

    }

    @Bean(name = "primaryMongoTemplate")
    public MongoTemplate getMongoTemplate() throws Exception {
        return new MongoTemplate(mongoDbFactory());
    }

}
```

**(4) 创建`Entity`包，并创建`User.java`文件**

`User.java`文件中内容：

```java
import org.springframework.data.mongodb.core.mapping.Document;

@Document(collection = "user")
public class User {
    private String userName;
    public User(){

    }

    public User(String userName){
        this.userName = userName;
    }

    public void setUserName(String userName){
        this.userName = userName;
    }
    public String getUserName(){
        return userName;
    }

    @Override
    public String toString() {
        return "User [userName=" + userName + "]";
    }
}
```

**(5) 创建`controller`包，并创建`controller.java`文件**

`controller.java`文件中内容：

```java
import javax.annotation.Resource;
import org.springframework.data.mongodb.core.MongoTemplate;
import org.springframework.web.bind.annotation.CrossOrigin;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;
import Entity.*;

@CrossOrigin
@RestController
public class controller {

    @Resource(name = "primaryMongoTemplate")
    private MongoTemplate mongoTemplate;

    @RequestMapping(value="/index",method = {RequestMethod.GET})
    public String Index() {
        User user = new User("张三");
        mongoTemplate.save(user);
        return "成功";
    }
}
```

**(6) 主启动目录中内容**

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.boot.autoconfigure.mongo.MongoAutoConfiguration;
import org.springframework.context.annotation.ComponentScan;

@EnableAutoConfiguration(exclude={MongoAutoConfiguration.class})
@ComponentScan("Configure,controller")
public class Start {

	public static void main(String[] args) {
		SpringApplication.run(Start.class, args);
	}

}
```

***

##### <font size="4" color="red">32. Springboot(2.4)中集成Springdata操作Mongodb(多数据源)(3)</font>

**(1) 依赖包**

```xml
<!-- springboot 父节点区域 -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.4.1</version>
</parent>
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-mongodb</artifactId>
    </dependency>
</dependencies>
```

**(2) 主启动目录中创建资源文件夹`resources`文件夹，并创建`application.properties`文件**

`application.properties`文件中内容：

```ini
first.uri=mongodb://192.168.0.19:8092
first.database=test1
first.host=192.168.0.19
first.port=8092
second.uri=mongodb://192.168.0.19:8092
second.database=test2
second.host=192.168.0.19
second.port=8092
```

**(3) 创建`firstMongo`包，并创建`FirstMongoProperties.java`文件**

`FirstMongoProperties.java`文件中内容：

```java
import org.springframework.boot.autoconfigure.mongo.MongoProperties;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;

@Configuration
public class FirstMongoProperties {

    @Primary
    @Bean(name="firstMongoProperties1")
    @ConfigurationProperties(prefix="first")
    public MongoProperties firstMongoProperties(){
        return new MongoProperties();
    }
}
```

**(4) 在`firstMongo`创建`FirstMongoTemplate.java`文件**

`FirstMongoTemplate.java`文件中内容：

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.boot.autoconfigure.mongo.MongoProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;
import org.springframework.data.mongodb.core.MongoTemplate;
import org.springframework.data.mongodb.core.SimpleMongoClientDatabaseFactory;
import org.springframework.data.mongodb.repository.config.EnableMongoRepositories;
import com.mongodb.ConnectionString;
import com.mongodb.MongoClientSettings;
import com.mongodb.client.MongoClient;
import com.mongodb.client.MongoClients;

@Configuration
@EnableMongoRepositories(basePackages="firstDao",mongoTemplateRef="firstMongo")
public class FirstMongoTemplate {

    @Autowired
    @Qualifier("firstMongoProperties1")
    private MongoProperties mongoProperties;

    @Primary
    @Bean
    public SimpleMongoClientDatabaseFactory firstFactory(MongoProperties mongo) throws Exception{
        ConnectionString connectionString = new ConnectionString(mongo.getUri());
        MongoClientSettings settings = MongoClientSettings.builder()
            .applyConnectionString(connectionString)
            .retryWrites(true)
            .build();
        MongoClient mongClient = MongoClients.create(settings);
        return new SimpleMongoClientDatabaseFactory(mongClient,mongo.getDatabase());
    }

    @Primary
    @Bean(name="firstMongo")
    public MongoTemplate firstMongoTemplate() throws Exception {
        return new MongoTemplate(firstFactory(this.mongoProperties));
    }
}
```

**(5) 创建`secondMongo`包，并创建`SecondMongoProperties.java`文件**

`SecondMongoProperties.java`文件中内容：

```java
import org.springframework.boot.autoconfigure.mongo.MongoProperties;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class SecondMongoProperties {
    
    @Bean(name="secondMongoProperties2")
    @ConfigurationProperties(prefix="second")
    public MongoProperties firstMongoProperties(){
        return new MongoProperties();
    }

}
```

**(6) 在`secondMongo`创建`SecondMongoTemplate.java`文件**

`SecondMongoTemplate.java`文件中内容：

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.boot.autoconfigure.mongo.MongoProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.mongodb.core.MongoTemplate;
import org.springframework.data.mongodb.core.SimpleMongoClientDatabaseFactory;
import org.springframework.data.mongodb.repository.config.EnableMongoRepositories;
import com.mongodb.ConnectionString;
import com.mongodb.MongoClientSettings;
import com.mongodb.client.MongoClient;
import com.mongodb.client.MongoClients;

@Configuration
@EnableMongoRepositories(basePackages="secondDao",mongoTemplateRef="secondMongo")
public class SecondMongoTemplate {

    @Autowired
    @Qualifier("secondMongoProperties2")
    private MongoProperties mongoProperties;
    
    @Bean
    public SimpleMongoClientDatabaseFactory secondFactory(MongoProperties mongo) throws Exception{
        ConnectionString connectionString = new ConnectionString(mongo.getUri());
        MongoClientSettings settings = MongoClientSettings.builder()
            .applyConnectionString(connectionString)
            .retryWrites(true)
            .build();
        MongoClient mongClient = MongoClients.create(settings);
        return new SimpleMongoClientDatabaseFactory(mongClient,mongo.getDatabase());
    }

    @Bean(name="secondMongo")
    public MongoTemplate firstMongoTemplate() throws Exception {
        return new MongoTemplate(secondFactory(this.mongoProperties));
    }
    
}
```

**(7) 创建`Entity`包，并创建`User.java`文件**

`User.java`文件中内容：

```java
import org.springframework.data.mongodb.core.mapping.Document;

@Document(collection = "user")
public class User {
    private String userName;
    public User(){

    }

    public User(String userName){
        this.userName = userName;
    }

    public void setUserName(String userName){
        this.userName = userName;
    }
    public String getUserName(){
        return userName;
    }

    @Override
    public String toString() {
        return "User [userName=" + userName + "]";
    }
}
```

**(8) 创建`firstDao`包，并创建`FirstUserDao.java`文件**

`FirstUserDao.java`文件中内容：

```java
import org.bson.types.ObjectId;
import org.springframework.data.mongodb.repository.MongoRepository;
import org.springframework.stereotype.Repository;
import Entity.*;

@Repository
public interface FirstUserDao extends MongoRepository<User,ObjectId> {
    User findByUserName(String userName);
}
```

**(9) 创建`secondDao`包，并创建`SecondUserDao.java`文件**

`SecondUserDao.java`文件中内容：

```java
import org.bson.types.ObjectId;
import org.springframework.data.mongodb.repository.MongoRepository;
import org.springframework.stereotype.Repository;
import Entity.*;

@Repository
public interface SecondUserDao extends MongoRepository<User,ObjectId> {
    User findByUserName(String userName);
}
```

**(10) 创建`controller`包，并创建`controller.java`文件**

`controller.java`文件中内容：

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.CrossOrigin;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;
import Entity.*;
import firstDao.*;
import secondDao.*;

@CrossOrigin
@RestController
public class controller {

    @Autowired
    FirstUserDao firstRepository;

    @Autowired
    SecondUserDao secondRepository;

    @RequestMapping(value="/first",method = {RequestMethod.GET})
    public String First() {
        User user = new User("张三");
        firstRepository.save(user);
        return "成功";
    }

    @RequestMapping(value="/second",method = {RequestMethod.GET})
    public String Second() {
        User user = new User("张三");
        secondRepository.save(user);
        return "成功";
    }
}
```

**(11) 主启动目录中内容**

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.boot.autoconfigure.mongo.MongoAutoConfiguration;
import org.springframework.context.annotation.ComponentScan;

@EnableAutoConfiguration(exclude={MongoAutoConfiguration.class})
@ComponentScan("firstMongo,secondMongo,controller")
public class Start {

    public static void main(String[] args) {
        SpringApplication.run(Start.class, args);
    }
}
```

****

#####  <font size="4" color="red">33. Springboot(2.0)集成mybaits开启Ecache二级缓存(1)</font>

**(1) 依赖包**

```xml
 <!-- springboot 父节点区域 -->
 <parent>
  	  <groupId>org.springframework.boot</groupId>
  	  <artifactId>spring-boot-starter-parent</artifactId>
  	  <version>2.0.2.RELEASE</version>
  </parent>
  <dependencies>
       <dependency>
	       <groupId>org.springframework.boot</groupId>
	       <artifactId>spring-boot-starter-web</artifactId>
	  </dependency>
	  <dependency>
	      <groupId>mysql</groupId>
	      <artifactId>mysql-connector-java</artifactId>
	</dependency>
	<dependency>
	      <groupId>org.springframework.boot</groupId>
	      <artifactId>spring-boot-starter-data-jpa</artifactId>
	</dependency>
    <dependency>
          <groupId>com.alibaba</groupId>
          <artifactId>druid</artifactId>
          <version>1.0.13</version>
    </dependency>
    <dependency>
		  <groupId>dom4j</groupId>
		  <artifactId>dom4j</artifactId>
		  <version>1.6.1</version>
    </dependency>
	<!-- Spring Boot Mybatis 依赖 -->
	<dependency>
	     <groupId>org.mybatis.spring.boot</groupId>
	     <artifactId>mybatis-spring-boot-starter</artifactId>
	     <version>2.1.1</version>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-cache</artifactId>
  </dependency>
  <dependency>
       <groupId>net.sf.ehcache</groupId>
       <artifactId>ehcache</artifactId>
  </dependency>
</dependencies>
```

**(2) 主启动目录创建资源文件夹`resources`文件夹，并创建`application.properties`文件**

`application.properties`中内容：

```ini
server.port=8310
spring.mysql.url=jdbc:mysql://Ip地址:端口/数据库名称?serverTimezone=UTC&useSSL=false 
spring.mysql.username=用户名
spring.mysql.password=密码
spring.mysql.driver-class=com.mysql.jdbc.Driver
```

**(3) 在`resources`文件夹中，创建`maper`文件夹并创建`personMapper.xml`文件**

`personMapper.xml`中内容：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
<mapper namespace="Dao.PersonDao">
   <resultMap id="BaseTreeResultMap" type="Entity.Person">
       <result column="id" property="id"/>
       <result column="parent_id" property="parentId"/>
       <result column="name" property="name"/>
       <collection column="id" property="child" javaType="java.util.ArrayList"
                   ofType="Entity.Person" select="getNextPersonTree"/>
   </resultMap>
   <resultMap id="NextTreeResultMap" type="Entity.Person">
      <result column="id" property="id"/>
      <result column="name" property="name"/>
      <result column="parent_id" property="parentId"/>
      <collection column="id" property="child" javaType="java.util.ArrayList"
                  ofType="Entity.Person" select="getNextPersonTree"/>
  </resultMap>
  <sql id="Base_Column_List">id, name,parent_id</sql>
  <select id="getNextPersonTree" resultMap="NextTreeResultMap">
      SELECT id,name FROM person WHERE parent_id = #{id}
  </select>
  <select id="getNodeTree" resultMap="BaseTreeResultMap">
      SELECT
      <include refid="Base_Column_List"/>
      FROM person
      WHERE parent_id = 23
  </select>
</mapper>
```

**(4) 创建`Configure`包，并创建`Congifure.java`文件**

`Configure.java`中内容：

```java
import javax.sql.DataSource;
import org.apache.ibatis.session.SqlSessionFactory;
import org.mybatis.spring.SqlSessionFactoryBean;
import org.mybatis.spring.SqlSessionTemplate;
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;
import org.springframework.core.io.support.PathMatchingResourcePatternResolver;
import com.alibaba.druid.pool.DruidDataSource;

@Configuration
@MapperScan(basePackages ="Dao")
public class Configure {
    
	@Value("${spring.mysql.url}")
	private String url;
	@Value("${spring.mysql.username}")
	private String username;
	@Value("${spring.mysql.password}")
	private String password;
	@Value("${spring.mysql.driver-class}")
	private String driverName;
	
	@Bean
	@Primary
	public DataSource basicDataSource() throws Exception {
		DruidDataSource dataSource = new DruidDataSource();
        dataSource.setUrl(url);
        dataSource.setUsername(username);
        dataSource.setPassword(password);
        dataSource.setDriverClassName(driverName);
        //连接池配置
        dataSource.setMaxActive(3000);
        dataSource.setMinIdle(5);
        dataSource.setInitialSize(5);
        dataSource.setMaxWait(60000);
        dataSource.setTimeBetweenEvictionRunsMillis(60000);
        dataSource.setMinEvictableIdleTimeMillis(300000);
        dataSource.setTestWhileIdle(true);
        dataSource.setTestOnBorrow(true);
        dataSource.setTestOnReturn(true);
        dataSource.setValidationQuery("SELECT 'x'");
        dataSource.setPoolPreparedStatements(true);
        dataSource.setMaxPoolPreparedStatementPerConnectionSize(20);
		return dataSource;
	}
	
	@Bean(name = "testSqlSessionFactory")
    public SqlSessionFactory testSqlSessionFactory() throws Exception {
        SqlSessionFactoryBean factoryBean = new SqlSessionFactoryBean();
        factoryBean.setDataSource(basicDataSource());
        PathMatchingResourcePatternResolver pathMatch = new       PathMatchingResourcePatternResolver();
        factoryBean.setMapperLocations(pathMatch.getResources("classpath:mapper/*.xml"));
        return factoryBean.getObject();
    }
	
	@Bean(name = "testSqlSessionTemplate")
	public SqlSessionTemplate testSqlSessionTemplate(@Qualifier("testSqlSessionFactory")      SqlSessionFactory sqlSessionFactory) {
        return new SqlSessionTemplate(sqlSessionFactory);
    }

}
```

**(5) 在`Configure`包中创建`MyEhcacheConfig.java`文件**

`MyEhcacheConfig.java`中内容：

```java
import org.springframework.cache.annotation.EnableCaching;
import org.springframework.cache.ehcache.EhCacheCacheManager;
import org.springframework.cache.ehcache.EhCacheManagerFactoryBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.io.ClassPathResource;

@Configuration
@EnableCaching
public class MyEhcacheConfig {

    @Bean
    public EhCacheCacheManager ehCacheCacheManager(EhCacheManagerFactoryBean bean) {
        return new EhCacheCacheManager(bean.getObject());
    }

    /**
     * 据shared与否的设置,
     * Spring分别通过CacheManager.create()
     * 或new CacheManager()方式来创建一个ehcache基地.
     * 也说是说通过这个来设置cache的基地是这里的Spring独用,还是跟别的(如hibernate的Ehcache共享)
     *
     * @return
     */
    @Bean
    public EhCacheManagerFactoryBean ehCacheManagerFactoryBean() {
        EhCacheManagerFactoryBean cacheManagerFactoryBean = new                            EhCacheManagerFactoryBean();
        cacheManagerFactoryBean.setConfigLocation(new ClassPathResource("ehcache.xml"));
        cacheManagerFactoryBean.setShared(true);
        return cacheManagerFactoryBean;
    }

}
```

**(6) 创建`Entity`包,并创建`Person.java`文件**

`Person.java`中内容：

```java
package Entity;
import java.io.Serializable;
import java.util.List;
public class Person implements Serializable{

	private static final long serialVersionUID = 1L;
	private int id;
	private int parentId;
	private String name;
	//子节点
	private List<Person> child;

	public Person() {
		
	}
    public Person(int id, int parentId, String name, List<Person> child) {
		this.id = id;
		this.parentId = parentId;
		this.name = name;
		this.child = child;
	}
	public void setId(int id) {
		this.id = id;
	}
	public int getId() {
		return id;
	}
	public void setParentId(int parentId) {
		this.parentId = parentId;
	}
	public int getParentId() {
		return parentId;
	}
	public void setName(String name) {
		this.name = name;
	}
	public String getName() {
		return name;
	}
	public void setChild(List<Person> child) {
		this.child = child;
	}
	public List<Person> getChild() {
		return child;
	}
}
```

**(7) 创建`Dao`包,并创建`PersonDao.java`文件**

`PersonDao.java`中内容：

```java
import java.util.List;
import org.apache.ibatis.annotations.CacheNamespace;
import org.apache.ibatis.annotations.Mapper;
import org.apache.ibatis.annotations.Select;
import Entity.*;

//分布式环境下必然会出现脏数据；
//多表联合查询的情况下极大可能会出现脏数据；

@Mapper
//整个类开启缓存
//@CacheNamespace
public interface PersonDao {

	@Select("select * from person where id = #{id}")
	Person getPerson(int id);
	//查询所有节点
	List<Person> getNodeTree();
	//查询节点
	List<Person> getNextPersonTree(int parentId);
}
```

**(8) 创建controller包,并创建controller.java文件**

controller.java中内容：

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.cache.annotation.Cacheable;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.CrossOrigin;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.ResponseBody;
import Dao.*;
import domain.ResultJson;

@CrossOrigin
@Controller
public class controller {

	@Autowired
    private PersonDao personDao;
	@Cacheable(key="'user_'+#uid",value="userCache")
	@RequestMapping(value = "/index",method = RequestMethod.GET)
	@ResponseBody
    public Person Index() throws Exception {
		return personDao.getPerson(1);
	}
	
}
```

**(9) 主启动目录中内容**

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.ComponentScan;

@SpringBootApplication
@ComponentScan("Configure,controller")
public class Start {
	public static void main(String[] args) {
		// TODO Auto-generated method stub
		SpringApplication.run(Start.class, args);
	}

}
```

***

##### <font size="4" color="red">34. Springboot集成Kaptcha图形验证码</font>

**(1) 依赖包**

```xml
<!-- springboot 父节点区域 -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.0.2.RELEASE</version>
</parent>
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>com.github.penggle</groupId>
        <artifactId>kaptcha</artifactId>
        <version>2.3.2</version>
    </dependency>
</dependencies>
```

**(2) 主启动目录中创建资源文件夹resources文件夹，并创建application.properties文件**

application.properties文件中内容：

```ini
server.port=8310
kaptcha.border=yes
kaptcha.border.color=105,179,90
kaptcha.textproducer.font.color=blue
kaptcha.textproducer.font.size=30
kaptcha.textproducer.font.names=宋体
kaptcha.textproducer.char.length=4
kaptcha.image.width=120
kaptcha.image.height=40
kaptcha.session.key=code
```

**(3) 创建Configure包，并创建Configure.java文件**

Configure.java文件中内容：

```java
import java.util.Properties;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import com.google.code.kaptcha.impl.DefaultKaptcha;
import com.google.code.kaptcha.util.Config;

@Configuration
public class Configure {
	
	@Value("${kaptcha.border}")
    private String border;

    @Value("${kaptcha.border.color}")
    private String borderColor;

    @Value("${kaptcha.textproducer.font.color}")
    private String textproducerFontColor;

    @Value("${kaptcha.textproducer.font.size}")
    private String textproducerFontSize;

    @Value("${kaptcha.textproducer.font.names}")
    private String textproducerFontNames;

    //验证码长度
    @Value("${kaptcha.textproducer.char.length}")
    private String textproducerCharLength;

    @Value("${kaptcha.image.width}")
    private String imageWidth;

    @Value("${kaptcha.image.height}")
    private String imageHeight;

    @Value("${kaptcha.session.key}")
    private String sessionKey;

    @Bean
    public DefaultKaptcha getDefaultKapcha(){
        DefaultKaptcha defaultKaptcha = new DefaultKaptcha();
        Properties properties = new Properties();
        properties.setProperty("kaptcha.border", border);
        properties.setProperty("kaptcha.border.color", borderColor);
        properties.setProperty("kaptcha.textproducer.font.color", textproducerFontColor);
        properties.setProperty("kaptcha.textproducer.font.size", textproducerFontSize);
        properties.setProperty("kaptcha.textproducer.font.names", textproducerFontNames);
        properties.setProperty("kaptcha.textproducer.char.length", textproducerCharLength);
        properties.setProperty("kaptcha.image.width", imageWidth);
        properties.setProperty("kaptcha.image.height", imageHeight);
        properties.setProperty("kaptcha.session.key", sessionKey);

        Config config = new Config(properties);
        defaultKaptcha.setConfig(config);
        return defaultKaptcha;
    }
}
```

**(3) 创建controller包，并创建controller.java文件**

controller.java文件中内容：

```java
import java.awt.image.BufferedImage;
import javax.imageio.ImageIO;
import javax.servlet.ServletOutputStream;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.CrossOrigin;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.servlet.ModelAndView;
import com.google.code.kaptcha.impl.DefaultKaptcha;

@CrossOrigin
@Controller
public class controller {

    /* 注入Kaptcha */
    @Autowired
    private DefaultKaptcha defaultKaptcha;

    @GetMapping("/code")
    @ResponseBody
    public ModelAndView getKaptchaImage(HttpServletRequest request, HttpServletResponse response) throws Exception {
        HttpSession session = request.getSession();
        response.setDateHeader("Expires", 0);
        // Set standard HTTP/1.1 no-cache headers.
        response.setHeader("Cache-Control", "no-store, no-cache, must-revalidate");
        // Set IE extended HTTP/1.1 no-cache headers (use addHeader).
        response.addHeader("Cache-Control", "post-check=0, pre-check=0");
        // Set standard HTTP/1.0 no-cache header.
        response.setHeader("Pragma", "no-cache");
        // return a jpeg
        response.setContentType("image/jpeg");
        // create the text for the image
        String capText = defaultKaptcha.createText();
        //存储文字
        session.setAttribute("test", capText);
        // create the image with the text
        BufferedImage bi = defaultKaptcha.createImage(capText);
        ServletOutputStream out = response.getOutputStream();
        // write the data out
        ImageIO.write(bi, "jpg", out);
        try {
            out.flush();
        } finally {
            out.close();
        }
        return null;
    }
	
}
```

**(5) 主启动目录中内容**

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.ComponentScan;

@SpringBootApplication
@ComponentScan("Configure,controller")

public class Start {

	public static void main(String[] args) {
		SpringApplication.run(Start.class, args);
	}
}
```

***

##### <font size="4" color="red">35. Springboot(2.0)集成zookeeper分布式锁</font>

**(1) 依赖包**

```xml
<!-- springboot 父节点区域 -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.2.0.RELEASE</version>
</parent>
<dependencies>
      <dependency>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-starter-web</artifactId>
     </dependency>
    <!-- zookeeper 客户端 -->
    <dependency>
         <groupId>org.apache.curator</groupId>
         <artifactId>curator-framework</artifactId>
         <version>2.12.0</version>
    </dependency>
    <dependency>
        <groupId>org.apache.curator</groupId>
        <artifactId>curator-recipes</artifactId>
        <version>2.12.0</version>
    </dependency>
</dependencies>
```

**(2) 主启动目录中创建资源文件夹resources文件夹，并创建application.properties文件**

application.properties文件中内容：

```ini
server.port=8310
##ZooKeeper地址
zk.url = 127.0.0.1:2181
```

**(3) 创建Configure包，并创建Configure.java文件**

Configure.java文件中内容：

```java
import org.apache.curator.RetryPolicy;
import org.apache.curator.framework.CuratorFramework;
import org.apache.curator.framework.CuratorFrameworkFactory;
import org.apache.curator.retry.ExponentialBackoffRetry;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class Configure {

    @Value("${zk.url}")
    private String zkUrl;

    @Bean
    public CuratorFramework getCuratorFramework() {
        // 用于重连策略，1000毫秒是初始化的间隔时间，3代表尝试重连次数
        RetryPolicy retryPolicy = new ExponentialBackoffRetry(1000, 3);
        //操作zookeeper集群zkUrl路径ip:xx,ip:xx
        CuratorFramework client = CuratorFrameworkFactory.newClient(zkUrl, retryPolicy);
        //必须调用start开始连接ZooKeeper
        client.start();
         /**
        * 使用Curator，可以通过LeaderSelector来实现领导选取；
        * 领导选取：选出一个领导节点来负责其他节点；如果领导节点不可用，则在剩下的机器里再选出一个领导节点
        */
        //构造一个监听器
        /*LeaderSelectorListenerAdapter listener = new LeaderSelectorListenerAdapter() {
               @Override
               public void takeLeadership(CuratorFramework curatorFramework) throws Exception {
                   log.info("get leadership");
                   // 领导节点，方法结束后退出领导。zk会再次重新选择领导
               }
        };
        LeaderSelector selector = new LeaderSelector(client, "/schedule", listener);
        selector.autoRequeue();
        selector.start();
        */
        return client;
      }
}
```

**(4) 创建controller包，并创建controller.java文件**

controller.java文件中内容：

```java
import java.util.concurrent.TimeUnit;
import org.apache.curator.framework.CuratorFramework;
import org.apache.curator.framework.recipes.locks.InterProcessMutex;
import org.apache.curator.framework.recipes.locks.InterProcessSemaphoreMutex;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class controller {

	@Autowired
    private CuratorFramework zkClient;

	@GetMapping("/zookeeperLock1")
	public ResultJson ZookeeperLock1() throws  Exception {
	 try {
		// InterProcessMutex 构建一个分布式锁该节点会在客户端链接断开时被删除
        InterProcessMutex lock = new InterProcessMutex(zkClient, "/test");
          try {
        	    //锁定5秒钟
	            if (lock.acquire(5, TimeUnit.HOURS)) {
	            	// 模拟业务处理耗时5秒
	                Thread.sleep(5*1000);
	                System.out.println("处理事务");
	               }
		        } finally {
		        	// 释放该锁
		            //lock.release();
		       }

 			} catch (Exception e) {
		    	// zk异常
		        e.printStackTrace();
	          }
		return "zookeeper1锁定";
	}

	@GetMapping("/zookeeperLock2")
	public String ZookeeperLock2() throws  Exception {
		// 获取锁
        InterProcessSemaphoreMutex balanceLock = new InterProcessSemaphoreMutex(zkClient, "/zktest");
        try {
        	// 执行加锁操作
            balanceLock.acquire();
            System.out.println("执行事务");

        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            try {
            	// 释放锁资源
                balanceLock.release();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
		return "zookeeper2锁定";
	}
}
```

**(5) 主启动目录中内容**

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.ComponentScan;

@SpringBootApplication
@ComponentScan("controller,Configure")
public class Start {

	public static void main(String[] args) {
		SpringApplication.run(Start.class, args);
	}
}
```

****

##### <font size="4" color="red">36. Springboot(2.0)中集成Springdata操作Mysql(自动建表)</font>

**(1) 依赖包**

```xml
<!-- springboot 父节点区域 -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.2.0.RELEASE</version>
</parent>
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>druid</artifactId>
        <version>1.0.13</version>
    </dependency>
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    <dependency>
        <groupId>dom4j</groupId>
        <artifactId>dom4j</artifactId>
        <version>1.6.1</version>
    </dependency>
</dependencies>
```

**(2) 主启动目录中创建资源文件夹`resources`文件夹，并创建`application.properties`文件**

`application.properties`文件中内容：

```ini
server.port=8310
spring.mysql.url=jdbc:mysql://127.0.0.1:3358/test?serverTimezone=UTC&useSSL=true
spring.mysql.username=mysql
spring.mysql.password=123456
spring.mysql.driver-class=com.mysql.cj.jdbc.Driver
spring.mysql.minPoolSize = 3
spring.mysql.maxPoolSize = 25
spring.mysql.maxLifetime = 20000
spring.mysql.borrowConnectionTimeout = 30
spring.mysql.loginTimeout = 30
spring.mysql.maintenanceInterval = 60
spring.mysql.maxIdleTime = 60
#在控制台打印出SQL语句
spring.jpa.show-sql=true
#每次运行程序，没有表格会新建表格，表内有数据不会清空，只会更新
spring.jpa.hibernate.ddl-auto=update
#Hibernate生成的SQL语句为MySQL方言
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL5Dialect
```

**(3) 创建`Configure`包，并创建`Configure.java`文件**

`Configure.java`文件中内容：

```java
import javax.sql.DataSource;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.autoconfigure.domain.EntityScan;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.jpa.repository.config.EnableJpaRepositories;
import com.alibaba.druid.pool.DruidDataSource;

@Configuration
@EnableJpaRepositories(basePackages="Dao")
@EntityScan("Entity")
public class Configure {

    @Value("${spring.mysql.url}")
    private String url;

    @Value("${spring.mysql.username}")
    private String username;

    @Value("${spring.mysql.password}")
    private String password;

    @Value("${spring.mysql.driver-class}")
    private String driverName;

    @Bean
    public DataSource basicDataSource() throws Exception {
        DruidDataSource dataSource = new DruidDataSource();
        dataSource.setUrl(url);
        dataSource.setUsername(username);
        dataSource.setPassword(password);
        dataSource.setDriverClassName(driverName);
        //连接池配置
        dataSource.setMaxActive(3000);
        dataSource.setMinIdle(5);
        dataSource.setInitialSize(5);
        dataSource.setMaxWait(60000);
        dataSource.setTimeBetweenEvictionRunsMillis(60000);
        dataSource.setMinEvictableIdleTimeMillis(300000);
        dataSource.setTestWhileIdle(true);
        dataSource.setTestOnBorrow(true);
        dataSource.setTestOnReturn(true);
        dataSource.setValidationQuery("SELECT 1");
        dataSource.setPoolPreparedStatements(true);
        dataSource.setMaxPoolPreparedStatementPerConnectionSize(20);
        return dataSource;
    }
}
```

**(4) 创建`Entity`包，并创建`Person.java`文件**

`controller.java`文件中内容：

```java
import java.util.Date;
import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;
import javax.persistence.Table;

@Entity
@Table(name="person_info")
public class Person {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private int id; 
    @Column(name="name")
    private String name;
    @Column(name="age")
    private int age;
    @Column(name="address")
    private String address;
    @Column(name="create_time")
    private Date createTime = new Date();
    @Column(name="update_time")
    private Date updateTime = new Date();

    public Person() {

    }

    public Person(String name, int age, String address) {
        this.name = name;
        this.age = age;
        this.address = address;
    }

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public String getAddress() {
        return address;
    }

    public void setAddress(String address) {
        this.address = address;
    }

    public Date getCreateTime() {
        return createTime;
    }

    public void setCreateTime(Date createTime) {
        this.createTime = createTime;
    }

    public Date getUpdateTime() {
        return updateTime;
    }

    public void setUpdateTime(Date updateTime) {
        this.updateTime = updateTime;
    }
}
```

**(5) 创建`Dao`包，并创建`PersonDao.java`文件**

`PersonDao.java`文件中内容：

```java
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;
import Entity.*;

@Repository
public interface PersonDao extends JpaRepository<Person,Integer>{

}
```

**(6) 创建`controller`包，并创建`controller.java`文件**

`controller.java`文件中内容：

```java
import java.util.ArrayList;
import java.util.List;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.CrossOrigin;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.ResponseBody;
import Dao.*;
import Entity.*;

@CrossOrigin
@Controller
public class controller {

    @Autowired
    private PersonDao personDao;

    @RequestMapping(value = "/index",method = RequestMethod.GET)
    @ResponseBody
    public List<Person> Index() throws Exception {
        List<Person> persons = personDao.findAll();
        return persons;
    }

    @RequestMapping(value = "/save",method = RequestMethod.GET)
    @ResponseBody
    public String Save() throws Exception {
        List<Person> persons = new ArrayList<Person>();
        Person person1 = new Person("李四",12,"广东");
        Person person2 = new Person("张三",20,"广东");
        persons.add(person1);
        persons.add(person2);
        personDao.saveAll(persons);
        return "成功";
    }
}
```

**(7) 主启动目录中内容**

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.context.annotation.ComponentScan;

@EnableAutoConfiguration
@ComponentScan("Configure,controller")
public class Start {
    public static void main(String[] args) {
        SpringApplication.run(Start.class, args);
    }
}
```

> **注：**
>
> ```
> #每次运行该程序，没有表格会新建表格，表内有数据会清空
> spring.jpa.hibernate.ddl-auto:create
> #每次程序结束的时候会清空表
> spring.jpa.hibernate.ddl-auto:create-drop
> #每次运行程序，没有表格会新建表格，表内有数据不会清空，只会更新
> spring.jpa.hibernate.ddl-auto:update
> #运行程序会校验数据与数据库的字段类型是否相同，不同会报错
> spring.jpa.hibernate.ddl-auto:validate
> ```

***

##### <font size="4" color="red">37. Springboot中挂载静态资源(1)</font>

**(1) 依赖包**

```xml
<!-- springboot 父节点区域 -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.2.0.RELEASE</version>
</parent>
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-thymeleaf</artifactId>
    </dependency>
</dependencies>
```

**(2) 主启动目录中创建资源文件夹`resources`文件夹，并创建`application.properties`文件**

`application.properties`文件中内容：

```ini
server.port=8310
spring.freemarker.request-context-attribute=request
spring.mvc.view.prefix=classpath:/templates/
spring.freemarker.suffix=.html
spring.freemarker.content-type=text/html
spring.freemarker.enabled=true
spring.freemarker.cache=false
spring.freemarker.charset=UTF-8
spring.freemarker.allow-request-override=false
spring.freemarker.expose-request-attributes=true
spring.freemarker.expose-session-attributes=true
spring.freemarker.expose-spring-macro-helpers=true
```

**(3) 在资源文件夹`resources`文件夹中，创建`templates`文件夹，并创建`index.html`文件**

`index.html`文件中内容：

```html
<html>
    <head>
        <title>测试页面</title>
    </head>
    <body>
        hello
    </body>
</html>
```

**(4) 创建`controller`包，并创建`controller.java`文件**

`controller.java`文件中内容：

```java
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.CrossOrigin;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;

@CrossOrigin
@Controller  
public class controller {

    @RequestMapping(value = "/index",method = RequestMethod.GET)
    public String Index() throws Exception {
        return "index";
    }
}
```

**(5) 主启动目录中内容**

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.context.annotation.ComponentScan;

@EnableAutoConfiguration
@ComponentScan("controller")
public class Start {

    public static void main(String[] args) {
        SpringApplication.run(Start.class, args);
    }
}
```

**(6) 访问`Web`页面**

```
地址：http://127.0.0.1:8310/index
```

***

##### <font size="4" color="red">38. Springboot中挂载静态资源(2)</font>

**(1) 依赖包**

```xml
<!-- springboot 父节点区域 -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.2.0.RELEASE</version>
</parent>
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-thymeleaf</artifactId>
    </dependency>
</dependencies>
```

**(2) 主启动目录中创建资源文件夹`resources`文件夹，并创建`application.properties`文件**

`application.properties`文件中内容：

```ini
server.port=8310
spring.freemarker.request-context-attribute=request
spring.mvc.view.prefix=classpath:/templates/
spring.freemarker.suffix=.html
spring.freemarker.content-type=text/html
spring.freemarker.enabled=true
spring.freemarker.cache=false
spring.freemarker.charset=UTF-8
spring.freemarker.allow-request-override=false
spring.freemarker.expose-request-attributes=true
spring.freemarker.expose-session-attributes=true
spring.freemarker.expose-spring-macro-helpers=true
```

**(3) 创建资源文件夹`static`，并创建`index.html`文件**

`index.html`文件中内容：

```html
<html>
    <head>
        <title>测试页面</title>
    </head>
    <body>
        hello
    </body>
</html>
```

**(4) 创建`controller`包，并创建`controller.java`文件**

`controller.java`文件中内容：

```java
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.CrossOrigin;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;

@CrossOrigin
@Controller  
public class controller {

    @RequestMapping(value = "/index",method = RequestMethod.GET)
    public String Index() throws Exception {
        return "index";
    }
}
```

**(5) 主启动目录中内容**

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.context.annotation.ComponentScan;

@EnableAutoConfiguration
@ComponentScan("controller")
public class Start {

    public static void main(String[] args) {
        SpringApplication.run(Start.class, args);
    }
}
```

**(6) 访问`Web`页面**

```
地址：http://127.0.0.1:8310/index.html
```

****

##### <font size="4" color="red">39. Springboot(2.0)集成Feign传递Oauth2-Token服务</font>

**1.`Eureka`服务器端**

**(1) 依赖包**

```xml
<!-- springboot 父节点区域 -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.0.2.RELEASE</version>
</parent>
<!--指定项目中公有的模块-->
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>Finchley.SR2</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
        <version>2.0.2.RELEASE</version>
    </dependency>
</dependencies>
```

**(2) 主启动目录中创建资源文件夹`resources`文件夹，并创建`application.properties`文件**

`application.properties`文件中内容：

```ini
server.port=8310
spring.application.name=eureka-server
eureka.client.fetch-registry=false
eureka.client.register-with-eureka=false
eureka.instance.hostname = localhost
eureka.client.serviceUrl.defaultZone=http://localhost:${server.port}/eureka/
spring.security.user.name=admin
spring.security.user.password=123456
```

**(3) 主启动目录中内容**

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;

@SpringBootApplication
@EnableEurekaServer
public class Start {

    public static void main(String[] args) {
        SpringApplication.run(Start.class, args);
    }
}
```

**2.服务提供者**

**(1) 依赖包**

```xml
<!-- springboot 父节点区域 -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.0.2.RELEASE</version>
</parent>
<!--指定项目中公有的模块-->
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>Finchley.SR2</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-openfeign</artifactId>
    </dependency>
</dependencies>
```

**(2) 主启动目录中创建资源文件夹`resources`文件夹，并创建`application.properties`文件**

`application.properties`文件中内容：

```ini
server.port=8313
spring.application.name = producer
eureka.client.serviceUrl.defaultZone=http://admin:123456@localhost:8310/eureka/
eureka.client.healthcheck.enabled = true
```

**(3) 创建`Entity`包，并创建`User.java`文件**

`User.java`文件中内容：

```java
public class User {

    private String name;
    private String address;

    public User() {

    }

    public User(String name, String address) {
        this.name = name;
        this.address = address;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getAddress() {
        return address;
    }

    public void setAddress(String address) {
        this.address = address;
    }
}
```

**(4) 创建`controller`包，并创建`UserApi.java`文件**

`UserApi.java`文件中内容：

```java
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import Entity.*;

@RequestMapping("user")
public interface UserApi {

    @GetMapping("hello")
    public String hello();

    @GetMapping("findById")
    public User findById(@RequestParam(value = "id")Integer id);

}
```

**(5) 在`controller`包，并创建`controller.java`文件**

`controller.java`文件中内容：

```java
import org.springframework.stereotype.Service;
import org.springframework.web.bind.annotation.ResponseBody;
import Entity.*;

@Service
@ResponseBody
public class controller implements UserApi {

    @Override
    public String hello() {
        System.out.println("我进来了");
        return "我来了" ;
    }

    @Override
    public User findById(Integer id) {
        User user = new User();
        user.setName("张三");
        user.setAddress("广东");
        return user;
    }
}
```

**(6) 主启动目录中内容**

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;
import org.springframework.context.annotation.ComponentScan;

@EnableAutoConfiguration
@ComponentScan("controller")
@EnableEurekaClient
public class Start {

    public static void main(String[] args) {
        SpringApplication.run(Start.class, args);
    }
}
```

**3.服务消费者**

**(1) 依赖包**

```xml
<!-- springboot 父节点区域 -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.0.2.RELEASE</version>
</parent>
<!--指定项目中公有的模块-->
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>Finchley.SR2</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-openfeign</artifactId>
    </dependency>
</dependencies>
```

**(2) 主启动目录中创建资源文件夹`resources`文件夹，并创建`application.properties`文件**

`application.properties`文件中内容：

```ini
server.port=8390
spring.application.name=consumer
eureka.client.serviceUrl.defaultZone=http://admin:123456@localhost:8310/eureka/
eureka.client.healthcheck.enabled = true
```

**(3) 创建`Entity`包，并创建`User.java`文件**

`User.java`文件中内容：

```java
public class User {

    private String name;
    private String address;

    public User() {

    }

    public User(String name, String address) {
        this.name = name;
        this.address = address;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getAddress() {
        return address;
    }

    public void setAddress(String address) {
        this.address = address;
    }
}
```

**(4) 创建`service`包，并创建`UserApi.java`文件**

`UserApi.java`文件中内容：

```java
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.ResponseBody;
import Entity.*;

@RequestMapping("user")
@Controller
public interface UserApi {

    @GetMapping("hello")
    @ResponseBody
    public String hello();

    @GetMapping("findById")
    @ResponseBody
    public User findById(@RequestParam(value = "id")Integer id);
}
```

**(5) 在`service`包中创建`ConsumerService.java`文件**

`ConsumerService.java`文件中内容：

```java
import org.springframework.cloud.openfeign.FeignClient;
@FeignClient(name = "producer")
public interface ConsumerService extends UserApi{

}
```

**(6) 创建`Interceptor`包，并创建`TokenFeignClientInterceptor.java`文件**

`TokenFeignClientInterceptor.java`文件中内容：

```java
import javax.servlet.http.HttpServletRequest;
import org.springframework.http.HttpHeaders;
import org.springframework.stereotype.Component;
import org.springframework.web.context.request.RequestAttributes;
import org.springframework.web.context.request.RequestContextHolder;
import org.springframework.web.context.request.ServletRequestAttributes;
import feign.RequestInterceptor;
import feign.RequestTemplate;
@Component
public class TokenFeignClientInterceptor implements RequestInterceptor {

    /**
   * token放在请求头.
   * @param requestTemplate 请求参数
   */
    @Override
    public void apply(RequestTemplate requestTemplate) {
        RequestAttributes requestAttributes = RequestContextHolder.currentRequestAttributes();
        if (requestAttributes != null) {
            ServletRequestAttributes attributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
            HttpServletRequest request = attributes.getRequest();
            //添加token
            requestTemplate.header(HttpHeaders.AUTHORIZATION, request.getHeader(HttpHeaders.AUTHORIZATION));
        }
    }
}
```

**(7) 创建`controller`包，并创建`controller.java`文件**

`controller.java`文件中内容：

```java
import java.net.URI;
import java.util.List;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.cloud.client.ServiceInstance;
import org.springframework.cloud.client.discovery.DiscoveryClient;
import org.springframework.cloud.client.loadbalancer.LoadBalancerClient;
import org.springframework.http.ResponseEntity;
import org.springframework.stereotype.Service;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;
import com.netflix.appinfo.InstanceInfo;
import com.netflix.appinfo.InstanceInfo.InstanceStatus;
import com.netflix.discovery.EurekaClient;
import Entity.*;
import service.*;

@Service
@RestController
public class controller {

    @Autowired
    DiscoveryClient discoveryClient;

    @Autowired
    private ConsumerService consumerService;
    @Autowired
    EurekaClient eurekaClient;
    @Autowired
    LoadBalancerClient loadBalancerClient;


    @GetMapping("hello")
    public String getHello() {
        return consumerService.hello();
    }

    @GetMapping("findById")
    public User findById(Integer id) {
        return consumerService.findById(id);
    }


    @GetMapping("client1")
    public void getClient() {
        List<String> list = discoveryClient.getServices();
        for (String str : list) {
            System.out.println(str);
        }
    }

    @GetMapping("client2")
    public Object getClient2() {
        return discoveryClient.getInstances("producer");
    }

    @GetMapping("client3")
    public Object getClient3() {
        @SuppressWarnings("unchecked")
        List<InstanceInfo> list = eurekaClient.getInstancesById("HWL-2020:producer:8313");
        InstanceInfo instanceInfo = list.get(0);
        if(instanceInfo.getStatus() == InstanceStatus.UP) {
            String url = "http://" + instanceInfo.getHostName() + ":" + instanceInfo.getPort() + "/user/hello" ;
            RestTemplate restTemplate = new RestTemplate();
            ResponseEntity<String> responseEntity = restTemplate.getForEntity(url, String.class);
            System.out.println(responseEntity.getBody());
        }
        return null;
    }

    @GetMapping("client4")
    public Object getClient4() {
        ServiceInstance serviceInstance = loadBalancerClient.choose("producer");
        URI uri = serviceInstance.getUri();
        RestTemplate restTemplate = new RestTemplate();
        ResponseEntity<String> responseEntity = restTemplate.getForEntity(uri.toString()+"/user/hello", String.class);
        return responseEntity.getBody();
    }
}
```

**(8) 主启动目录中内容**

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.cloud.openfeign.EnableFeignClients;
import org.springframework.context.annotation.ComponentScan;

@EnableAutoConfiguration
@EnableFeignClients("service")
@ComponentScan("controller")
public class Start {

    public static void main(String[] args) {
        SpringApplication.run(Start.class, args);
    }
}
```

> **注：**在客户端调用提供者时头传递`Authorization`信息

***

##### <font size="4" color="red">40. Springboot(2.0)集成数字计算验证码</font>

**(1) 依赖包**

```xml
<!-- springboot 父节点区域 -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.0.2.RELEASE</version>
</parent>
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <!--验证码-->
    <dependency>
        <groupId>com.github.penggle</groupId>
        <artifactId>kaptcha</artifactId>
        <version>2.3.2</version>
    </dependency>
</dependencies>
```

**(2) 主启动目录中创建资源文件夹`resources`文件夹，并创建`application.properties`文件**

`application.properties`文件中内容：

```ini
server.port=8310
```

**(3) 创建`controller`包，并创建`controller.java`文件**

`controller.java`文件中内容：

```java
import java.awt.image.BufferedImage;
import javax.imageio.ImageIO;
import javax.servlet.ServletOutputStream;
import javax.servlet.http.HttpServletResponse;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.CrossOrigin;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;
import com.google.code.kaptcha.Producer;

@CrossOrigin
@RestController
public class controller {

    @Autowired
    private Producer producer;

    @RequestMapping(value = "/index",method = RequestMethod.GET)
    public void Index(HttpServletResponse response) throws Exception {
        response.setHeader("Cache-Control", "no-store, no-cache");
        response.setContentType("image/jpeg");
        //生成文字验证码
        String text = producer.createText();
        //个位数字相加
        String s1 = text.substring(0, 1);
        String s2 = text.substring(1, 2);
        int count = Integer.valueOf(s1).intValue() + Integer.valueOf(s2).intValue();
        //生成图片验证码
        BufferedImage image = producer.createImage(s1 + "+" + s2 + "=?");
        //保存 redis key 自己设置
        //stringRedisTemplate.opsForValue().set("",String.valueOf(count));
        ServletOutputStream out = response.getOutputStream();
        ImageIO.write(image, "jpg", out);
    }
}
```

**(4) 主启动目录中内容**

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.context.annotation.ComponentScan;

@EnableAutoConfiguration
@ComponentScan("Configure,controller")
public class Start {

    public static void main(String[] args) {
        SpringApplication.run(Start.class, args);
    }
}
```

****

##### <font size="4" color="red">41. Springboot(2.0)中文件上传和下载(1)</font>

**(1) 依赖包**

```xml
<!-- springboot 父节点区域 -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.4.0</version>
</parent>
<dependencies>
    <!-- springboot依赖包 -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```

**(2) 主启动目录中创建资源文件夹`resources`，并创建`application.properties`文件**

`application.properties`文件中内容：

```ini
server.port = 8310
spring.servlet.multipart.max-file-size=100
spring.servlet.multipart.max-request-size=100
```

**(3) 创建`Configure`包，并创建`Configure.java`文件**

`Configure.java`文件中内容：

```java
import javax.servlet.MultipartConfigElement;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.web.servlet.MultipartConfigFactory;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.util.unit.DataSize;

@Configuration
public class Configure {

    @Value("${spring.servlet.multipart.max-request-size}")
    private Long maxRequestSize;

    @Value("${spring.servlet.multipart.max-file-size}")
    private Long maxFileSize;

    @Bean
    public MultipartConfigElement multipartConfigElement() {
        MultipartConfigFactory factory = new MultipartConfigFactory();
        // 置文件大小限制 ,超出此大小页面会抛出异常信息
        factory.setMaxRequestSize(DataSize.ofMegabytes(maxRequestSize)); //KB,MB
        // 设置总上传数据总大小
        factory.setMaxRequestSize(DataSize.ofMegabytes(maxFileSize));
        // 设置文件临时文件夹路径
        // factory.setLocation("E://test//");
        // 如果文件大于这个值，将以文件的形式存储，如果小于这个值文件将存储在内存中，默认为0
        // factory.setMaxRequestSize(0);
        return factory.createMultipartConfig();
    }

}
```

**(4) 创建`controller`包，并创建`controller.java`文件**

`controller.java`文件中内容：

```java
import java.io.BufferedInputStream;
import java.io.File;
import java.io.FileInputStream;
import java.io.IOException;
import java.io.OutputStream;
import java.util.List;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import org.apache.shiro.SecurityUtils;
import org.apache.shiro.crypto.hash.SimpleHash;
import org.apache.shiro.subject.Subject;
import org.apache.shiro.util.ByteSource;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.multipart.*;

@RestController
public class Controller {

    private static String s = System.getProperty("user.dir");

    @PostMapping("/upload")
    public String upload(@RequestParam(value = "file", required = false) MultipartFile file) {
        if (file.isEmpty()) {
            return "上传失败，请选择文件";
        }
        String fileName = file.getOriginalFilename();
        String filePath = s+"/temp/";
        File dest = new File(filePath + fileName);
        File filefoder = new File(filePath);
        if(!filefoder.exists()) {
            filefoder.mkdirs();
        }
        try {
            file.transferTo(dest);
            return "上传成功";
        } catch (IOException e) {

        }
        return "上传失败！";
    }

    @PostMapping("/multiUpload")
    public String multiUpload(HttpServletRequest request) {
        List<MultipartFile> files = ((MultipartHttpServletRequest) request).getFiles("file");
        String filePath = s+"/temp/";
        File filefoder = new File(filePath);
        if(!filefoder.exists()) {
            filefoder.mkdirs();
        }
        for (int i = 0; i < files.size(); i++) {
            MultipartFile file = files.get(i);
            if (file.isEmpty()) {
                return "上传第" + (i++) + "个文件失败";
            }
            String fileName = file.getOriginalFilename();
            File dest = new File(filePath + fileName);
            try {
                file.transferTo(dest);
            } catch (IOException e) {
                return "上传第" + (i++) + "个文件失败";
            }
        }

        return "上传成功";

    }

    //文件下载相关代码
    @RequestMapping("/download")
    public void downloadFile(HttpServletRequest request, HttpServletResponse response) {
        String fileName = "zookeeper-3.4.13.tar.gz";// 设置文件名，根据业务需要替换成要下载的文件名
        System.out.println(request.getParameter("filename"));
        if (fileName != null) {
            //设置文件路径
            String filePath = s+"/temp/";
            File file = new File(filePath , fileName);
            if (file.exists()) {
                response.setContentType("application/force-download");// 设置强制下载不打开
                response.addHeader("Content-Disposition", "attachment;fileName=" + fileName);// 设置文件名
                byte[] buffer = new byte[1024];
                FileInputStream fis = null;
                BufferedInputStream bis = null;
                try {
                    fis = new FileInputStream(file);
                    bis = new BufferedInputStream(fis);
                    OutputStream os = response.getOutputStream();
                    int i = bis.read(buffer);
                    while (i != -1) {
                        os.write(buffer, 0, i);
                        i = bis.read(buffer);
                    }
                }
                catch (Exception e) {
                    if (bis != null) {
                        try {
                            bis.close();
                        } catch (IOException ex) {
                            e.printStackTrace();
                        }
                    }
                    if (fis != null) {
                        try {
                            fis.close();
                        } catch (IOException ex) {
                            e.printStackTrace();
                        }
                    }
                } 
            }
        }
    }
}
```

**(5) 主启动目录中内容**

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

@Configuration
@EnableAutoConfiguration
@ComponentScan("Configure,controller")
public class Start {
    
    public static void main(String[] args){
        SpringApplication.run(Start.class,args);
    }
}
```

***

##### <font size="4" color="red">42. Springboot(2.0)中文件上传和下载(2)</font>

**(1) 依赖包**

```xml
<!-- springboot 父节点区域 -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.4.0</version>
</parent>
<dependencies>
    <!-- springboot依赖包 -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```

**(2) 主启动目录中创建资源文件夹`resources`，并创建`application.properties`文件**

`application.properties`文件中内容：

```ini
server.port = 8310
spring.servlet.multipart.max-file-size=1024MB
spring.servlet.multipart.max-request-size=1024MB
```

**(3) 创建`controller`包，并创建`controller.java`文件**

`controller.java`文件中内容：

```java
import java.io.BufferedInputStream;
import java.io.File;
import java.io.FileInputStream;
import java.io.IOException;
import java.io.OutputStream;
import java.util.List;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import org.apache.shiro.SecurityUtils;
import org.apache.shiro.crypto.hash.SimpleHash;
import org.apache.shiro.subject.Subject;
import org.apache.shiro.util.ByteSource;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.multipart.*;

@RestController
public class Controller {

    private static String s = System.getProperty("user.dir");

    @PostMapping("/upload")
    public String upload(@RequestParam(value = "file", required = false) MultipartFile file) {
        if (file.isEmpty()) {
            return "上传失败，请选择文件";
        }
        String fileName = file.getOriginalFilename();
        String filePath = s+"/temp/";
        File dest = new File(filePath + fileName);
        File filefoder = new File(filePath);
        if(!filefoder.exists()) {
            filefoder.mkdirs();
        }
        try {
            file.transferTo(dest);
            return "上传成功";
        } catch (IOException e) {

        }
        return "上传失败！";
    }

    @PostMapping("/multiUpload")
    public String multiUpload(HttpServletRequest request) {
        List<MultipartFile> files = ((MultipartHttpServletRequest) request).getFiles("file");
        String filePath = s+"/temp/";
        File filefoder = new File(filePath);
        if(!filefoder.exists()) {
            filefoder.mkdirs();
        }
        for (int i = 0; i < files.size(); i++) {
            MultipartFile file = files.get(i);
            if (file.isEmpty()) {
                return "上传第" + (i++) + "个文件失败";
            }
            String fileName = file.getOriginalFilename();
            File dest = new File(filePath + fileName);
            try {
                file.transferTo(dest);
            } catch (IOException e) {
                return "上传第" + (i++) + "个文件失败";
            }
        }
        return "上传成功";
    }

    //文件下载相关代码
    @RequestMapping("/download")
    public void downloadFile(HttpServletRequest request, HttpServletResponse response) {
        String fileName = "zookeeper-3.4.13.tar.gz";// 设置文件名，根据业务需要替换成要下载的文件名
        System.out.println(request.getParameter("filename"));
        if (fileName != null) {
            //设置文件路径
            String filePath = s+"/temp/";
            File file = new File(filePath , fileName);
            if (file.exists()) {
                response.setContentType("application/force-download");// 设置强制下载不打开
                response.addHeader("Content-Disposition", "attachment;fileName=" + fileName);// 设置文件名
                byte[] buffer = new byte[1024];
                FileInputStream fis = null;
                BufferedInputStream bis = null;
                try {
                    fis = new FileInputStream(file);
                    bis = new BufferedInputStream(fis);
                    OutputStream os = response.getOutputStream();
                    int i = bis.read(buffer);
                    while (i != -1) {
                        os.write(buffer, 0, i);
                        i = bis.read(buffer);
                    }
                }
                catch (Exception e) {
                    if (bis != null) {
                        try {
                            bis.close();
                        } catch (IOException ex) {
                            e.printStackTrace();
                        }
                    }
                    if (fis != null) {
                        try {
                            fis.close();
                        } catch (IOException ex) {
                            e.printStackTrace();
                        }
                    }
                } 
            }
        }
    }
}
```

**(4) 主启动目录中内容**

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

@Configuration
@EnableAutoConfiguration
@ComponentScan("controller")
public class Start {
    
    public static void main(String[] args){
        SpringApplication.run(Start.class,args);
    }
}
```

***

##### <font size="4" color="red">43. Springboot(2.0)中限制文件大小</font>

**(1) 依赖包**

```xml
<!-- springboot 父节点区域 -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.4.0</version>
</parent>
<dependencies>
    <dependency>  
        <groupId>org.springframework.boot</groupId>  
        <artifactId>spring-boot-starter-web</artifactId>  
    </dependency> 
</dependencies>
```

**(2) 主启动目录中创建资源文件夹`resources`，并创建`application.properties`文件**

`application.properties`文件中内容：

```ini
server.port = 8310
spring.servlet.multipart.max-file-size=1024MB
spring.servlet.multipart.max-request-size=1024MB
```

**(3) 创建`controller`包，并创建`controller.java`文件**

`controller.java`文件中内容：

```java
import java.awt.image.BufferedImage;
import java.io.File;
import java.io.IOException;
import java.util.UUID;
import javax.imageio.ImageIO;
import org.springframework.web.bind.annotation.CrossOrigin;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.multipart.MultipartFile;

@CrossOrigin
@RestController
public class controller {

    @RequestMapping(value = "/index",method = RequestMethod.POST)
    public String Index(@RequestParam("file") MultipartFile file) throws Exception {
        if(file.isEmpty()) {
            return "选择图片";
        }
        //检查文件大小
        if(file.getSize() > 1*1024*1024) {
            return "请上传1M以内的图片";    
        }
        //检查是否是图片
        BufferedImage bi = ImageIO.read(file.getInputStream());
        if(bi == null){ 
            return "上传的文件不是图片";    
        }
        String originalFilename = file.getOriginalFilename();
        String fileType = null;
        if(originalFilename.contains(".")) {
            fileType = originalFilename.substring(originalFilename.lastIndexOf("."));
        } else {
            fileType = ".jpg";
        }
        String filePath = "D://test//"; // 上传后的路径
        String fileName = UUID.randomUUID() + fileType; // 新文件名
        File dest = new File(filePath + fileName);
        if (!dest.getParentFile().exists()) {
            dest.getParentFile().mkdirs();
        }
        try {
            file.transferTo(dest);
        } catch (IOException e) {
            e.printStackTrace();
        }
        return "成功";
    }
}
```

**(4) 主启动目录中内容**

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.context.annotation.ComponentScan;

@EnableAutoConfiguration
@ComponentScan("controller")
public class Start {

    public static void main(String[] args) {
        SpringApplication.run(Start.class, args);
    }
}
```

***

##### <font size="4" color="red">44. Springboot(2.0)中检测上传文件名</font>

**(1) 依赖包**

```xml
<!-- springboot 父节点区域 -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.4.0</version>
</parent>
<dependencies>
    <dependency>  
        <groupId>org.springframework.boot</groupId>  
        <artifactId>spring-boot-starter-web</artifactId>  
    </dependency> 
</dependencies>
```

**(2) 主启动目录中创建资源文件夹`resources`，并创建`application.properties`文件**

`application.properties`文件中内容：

```ini
server.port = 8310
spring.servlet.multipart.max-file-size=1024MB
spring.servlet.multipart.max-request-size=1024MB
```

**(3) 创建`Utils`包，并创建`FileTypeUtils.java`文件**

`FileTypeUtils.java`文件中内容：

```java
import java.io.IOException;
import java.io.InputStream;
import java.util.HashMap;
import java.util.Map;

public class FileTypeUtils {

    // 默认判断文件头前三个字节内容
    public static int default_check_length = 3;
    final static HashMap<String, String> fileTypeMap = new HashMap<>();

    // 初始化文件头类型，不够的自行补充
    static {
        fileTypeMap.put("ffd8ffe000104a464946", "jpg"); //JPEG (jpg)
        fileTypeMap.put("89504e470d0a1a0a0000", "png"); //PNG (png)
        fileTypeMap.put("47494638396126026f01", "gif"); //GIF (gif)
        fileTypeMap.put("49492a00227105008037", "tif"); //TIFF (tif)
        fileTypeMap.put("424d228c010000000000", "bmp"); //16色位图(bmp)
        fileTypeMap.put("424d8240090000000000", "bmp"); //24位位图(bmp)
        fileTypeMap.put("424d8e1b030000000000", "bmp"); //256色位图(bmp)
        fileTypeMap.put("41433130313500000000", "dwg"); //CAD (dwg)
        fileTypeMap.put("3c21444f435459504520", "html"); //HTML (html)
        fileTypeMap.put("3c21646f637479706520", "htm"); //HTM (htm)
        fileTypeMap.put("48544d4c207b0d0a0942", "css"); //css
        fileTypeMap.put("696b2e71623d696b2e71", "js"); //js
        fileTypeMap.put("7b5c727466315c616e73", "rtf"); //Rich Text Format (rtf)
        fileTypeMap.put("38425053000100000000", "psd"); //Photoshop (psd)
        fileTypeMap.put("46726f6d3a203d3f6762", "eml"); //Email [Outlook Express 6] (eml)
        fileTypeMap.put("d0cf11e0a1b11ae10000", "doc"); //MS Excel 注意：word、msi 和 excel的文件头一样
        fileTypeMap.put("d0cf11e0a1b11ae10000", "vsd"); //Visio 绘图
        fileTypeMap.put("5374616E64617264204A", "mdb"); //MS Access (mdb)
        fileTypeMap.put("252150532D41646F6265", "ps");
        fileTypeMap.put("255044462d312e350d0a", "pdf"); //Adobe Acrobat (pdf)
        fileTypeMap.put("2e524d46000000120001", "rmvb"); //rmvb/rm相同
        fileTypeMap.put("464c5601050000000900", "flv"); //flv与f4v相同
        fileTypeMap.put("00000020667479706d70", "mp4");
        fileTypeMap.put("49443303000000002176", "mp3");
        fileTypeMap.put("000001ba210001000180", "mpg"); //
        fileTypeMap.put("3026b2758e66cf11a6d9", "wmv"); //wmv与asf相同
        fileTypeMap.put("52494646e27807005741", "wav"); //Wave (wav)
        fileTypeMap.put("52494646d07d60074156", "avi");
        fileTypeMap.put("4d546864000000060001", "mid"); //MIDI (mid)
        fileTypeMap.put("504b0304140000000800", "zip");
        fileTypeMap.put("526172211a0700cf9073", "rar");
        fileTypeMap.put("235468697320636f6e66", "ini");
        fileTypeMap.put("504b03040a0000000000", "jar");
        fileTypeMap.put("4d5a9000030000000400", "exe");//可执行文件
        fileTypeMap.put("3c25402070616765206c", "jsp");//jsp文件
        fileTypeMap.put("4d616e69666573742d56", "mf");//MF文件
        fileTypeMap.put("3c3f786d6c2076657273", "xml");//xml文件
        fileTypeMap.put("494e5345525420494e54", "sql");//xml文件
        fileTypeMap.put("7061636b616765207765", "java");//java文件
        fileTypeMap.put("406563686f206f66660d", "bat");//bat文件
        fileTypeMap.put("1f8b0800000000000000", "gz");//gz文件
        fileTypeMap.put("6c6f67346a2e726f6f74", "properties");//bat文件
        fileTypeMap.put("cafebabe0000002e0041", "class");//bat文件
        fileTypeMap.put("49545346030000006000", "chm");//bat文件
        fileTypeMap.put("04000000010000001300", "mxp");//bat文件
        fileTypeMap.put("504b0304140006000800", "docx");//docx文件
        fileTypeMap.put("d0cf11e0a1b11ae10000", "wps");//WPS文字wps、表格et、演示dps都是一样的
        fileTypeMap.put("6431303a637265617465", "torrent");
        fileTypeMap.put("6D6F6F76", "mov"); //Quicktime (mov)
        fileTypeMap.put("FF575043", "wpd"); //WordPerfect (wpd)
        fileTypeMap.put("CFAD12FEC5FD746F", "dbx"); //Outlook Express (dbx)
        fileTypeMap.put("2142444E", "pst"); //Outlook (pst)
        fileTypeMap.put("AC9EBD8F", "qdf"); //Quicken (qdf)
        fileTypeMap.put("E3828596", "pwl"); //Windows Password (pwl)
        fileTypeMap.put("2E7261FD", "ram"); //Real Audio (ram)

    }

    /**
	 * @param fileName
	 * @return String
	 * @description 通过文件后缀名获取文件类型
	 */
    public static String getFileTypeBySuffix(String fileName) {
        return fileName.substring(fileName.lastIndexOf(".") + 1, fileName.length());
    }

    /**
	 * @param inputStream
	 * @return String
	 * @description 通过文件头魔数获取文件类型
	 */
    public static String getFileTypeByMagicNumber(InputStream inputStream) {
        byte[] bytes = new byte[default_check_length];
        try {
            // 获取文件头前三位魔数的二进制
            inputStream.read(bytes, 0, bytes.length);
            // 文件头前三位魔数二进制转为16进制
            String code = bytesToHexString(bytes);
            for (Map.Entry<String, String> item : fileTypeMap.entrySet()) {
                if (code.startsWith(item.getKey()) || item.getKey().startsWith(code)) {
                    return item.getValue();
                }
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
        return "";
    }

    public static String bytesToHexString(byte[] bytes) {
        StringBuilder stringBuilder = new StringBuilder();
        for (int i = 0; i < bytes.length; i++) {
            int v = bytes[i] & 0xFF;
            String hv = Integer.toHexString(v);
            if (hv.length() < 2) {
                stringBuilder.append(0);
            }
            stringBuilder.append(hv);
        }
        return stringBuilder.toString();
    }
}
```

**(4) 创建`controller`包，并创建`controller.java`文件**

`controller.java`文件中内容：

```java
import java.io.File;
import java.io.IOException;
import java.util.UUID;
import org.springframework.web.bind.annotation.CrossOrigin;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.multipart.MultipartFile;
import Utils.FileTypeUtils;

@CrossOrigin
@RestController
public class controller {


    @RequestMapping(value = "/index",method = RequestMethod.POST)
    public String Index(@RequestParam("file") MultipartFile file) throws Exception {

        String type = FileTypeUtils.getFileTypeByMagicNumber(file.getInputStream());
        System.out.println(type);
        String originalFilename = file.getOriginalFilename();
        String fileType = null;
        if(originalFilename.contains(".")) {
            fileType = originalFilename.substring(originalFilename.lastIndexOf("."));
        } else {
            fileType = ".jpg";
        }
        String filePath = "D://test//"; // 上传后的路径
        String fileName = UUID.randomUUID() + fileType; // 新文件名
        File dest = new File(filePath + fileName);
        if (!dest.getParentFile().exists()) {
            dest.getParentFile().mkdirs();
        }
        try {
            file.transferTo(dest);
        } catch (IOException e) {
            e.printStackTrace();
        }
        return "成功";
    }
}
```

**(5) 主启动目录中内容**

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.context.annotation.ComponentScan;

@EnableAutoConfiguration
@ComponentScan("controller")
public class Start {

    public static void main(String[] args) {
        SpringApplication.run(Start.class, args);
    }
}
```

***

##### <font size="4" color="red">45. Springboot集成Zipkin分布式监控</font>

**(1) 依赖包**

```xml
<!-- springboot 父节点区域 -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.1.1.RELEASE</version>
</parent>
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-zipkin</artifactId>
    </dependency>
</dependencies>
```

**(2) 主启动目录中创建资源文件夹`resources`文件夹，并创建`application.properties`文件**

`application.properties`文件中内容：

```ini
server.port=8310
#服务追踪
spring.zipkin.service.name=zipkin-test
spring.zipkin.base-url=http://127.0.0.1:9411/
spring.zipkin.discovery-client-enabled=true
spring.zipkin.sender.type=web
spring.sleuth.sampler.probability=1
```

**(3) 创建`controller`包，并创建`controller.java`文件**

`controller.java`文件中内容：

```java
import org.springframework.web.bind.annotation.CrossOrigin;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;

@CrossOrigin
@RestController
public class controller {

    @RequestMapping(value = "/index",method = RequestMethod.GET)
    public String Index1() throws Exception {
        return "成功";
    }
}
```

**(4) 主启动目录中内容**

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.context.annotation.ComponentScan;

@EnableAutoConfiguration
@ComponentScan("controller")
public class Start {

    public static void main(String[] args) {
        SpringApplication.run(Start.class, args);
    }

}
```

***

##### <font size="4" color="red">46. Springboot集成Came-Ftp文件定时同步</font>

**(1) 依赖包**

```xml
<!-- springboot 父节点区域 -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.0.2.RELEASE</version>
</parent>
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.apache.camel</groupId>
        <artifactId>camel-spring-boot-starter</artifactId>
        <version>2.22.1</version>
    </dependency>
    <dependency>
        <groupId>org.apache.camel</groupId>
        <artifactId>camel-ftp</artifactId>
        <version>2.22.1</version>
    </dependency>
</dependencies>
```

**(2) 主启动目录中创建资源文件夹`resources`文件夹，并创建`application.properties`文件**

`application.properties`文件中内容：

```ini
server.port=8310
ftp.img.url=ftp://192.168.1.6:21/disk/D?username=root&password=123456
ftp.img.dir=file:D:/test/img
ftp.file.url=ftp://192.168.1.7:21/disk/D?username=root&password=123456
ftp.file.dir=file:D:/test/file
#后台进程
camel.springboot.main-run-controller=true
```

**(3) 创建`Configure`包，并创建`FtpRouteBuilder.java`文件**

`FtpRouteBuilder.java`文件中内容：

```java
import org.apache.camel.LoggingLevel;
import org.apache.camel.builder.RouteBuilder;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

@Component
public class FtpRouteBuilder extends RouteBuilder {
    private static Logger logger = LoggerFactory.getLogger(FtpRouteBuilder.class );


    @Value("${ftp.img.url}")
    private String sftpServerImg;

    @Value("${ftp.img.dir}")
    private String downloadLocationImg;

    @Value("${ftp.file.url}")
    private String sftpServerFile;

    @Value("${ftp.file.dir}")
    private String downloadLocationFile;

    @Autowired
    private DataProcessor dataProcessor;

    @Override
    public void configure() throws Exception {
        // from 内为 URL
        from(sftpServerImg)
            // 本地路径
            .to(downloadLocationImg)
            // 数据处理器
            .process(dataProcessor)
            .log(LoggingLevel.INFO, logger, "Download img ${file:name} complete.");

        from(sftpServerFile)		
            .to(downloadLocationFile)
            .log(LoggingLevel.INFO, logger, "Download file ${file:name} complete.");
    }
}
```

**(4) 在`Configure`包中创建`ImgFilter.java`文件**

`ImgFilter.java`文件中内容：

```java
import org.apache.camel.component.file.GenericFile;
import org.apache.camel.component.file.GenericFileFilter;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

@Component
public class ImgFilter implements GenericFileFilter<Object> {

    @Value("${ftp.img.dir}")
    private String fileDir;

    @Override
    public boolean accept(GenericFile<Object> genericFile) {
        return genericFile.getFileName().endsWith(".jpg") || genericFile.isDirectory();
    }
}
```

**(5) 在`Configure`包中创建`fileFilter.java`文件**

`fileFilter.java`文件中内容：

```java
import org.apache.camel.component.file.GenericFile;
import org.apache.camel.component.file.GenericFileFilter;

public class fileFilter implements GenericFileFilter {

    @Override
    public boolean accept(GenericFile file) {
        return file.getFileName().endsWith(".zip") || file.isDirectory();
    }

}
```

**(6) 在`Configure`包中创建`DataProcessor.java`文件**

`DataProcessor.java`文件中内容：

```java
import java.io.RandomAccessFile;
import org.apache.camel.Exchange;
import org.apache.camel.Processor;
import org.apache.camel.component.file.GenericFileMessage;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

@Component
public class DataProcessor implements Processor {

    @Value("${ftp.img.dir}")
    private String fileDir;

    @SuppressWarnings("unchecked")
    @Override
    public void process(Exchange exchange) throws Exception {
        try {
            GenericFileMessage<RandomAccessFile> inMsg = (GenericFileMessage<RandomAccessFile>) exchange.getIn();
            String fileName = inMsg.getGenericFile().getFileName();
            System.out.println(fileName);
            String sql = "insert into file(filename) values (?)";

        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

**(7) 创建`controller`包，并创建`controller.java`文件**

`controller.java`文件中内容：

```java
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.CrossOrigin;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.ResponseBody;

@CrossOrigin
@Controller
public class controller {

    @RequestMapping(value = "/index",method = RequestMethod.GET)
    @ResponseBody
    public String Index() throws Exception {
        return "成功";
    }
}
```

**(8) 主启动目录中内容**

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.context.annotation.ComponentScan;

@EnableAutoConfiguration
@ComponentScan("Configure,controller")
public class Start {

    public static void main(String[] args) {
        SpringApplication.run(Start.class, args);
    }

}
```

****

##### <font size="4" color="red">47. Springboot集成JustAuth第三方登录</font>

​		支持`Github`、`Gitee`、微博、钉钉、百度、`Coding`、腾讯云开发者平台、`OSChina`、支付宝、`QQ`、微信、淘宝、`Google`、`Facebook`、抖音、领英、小米、微软、今日头条、`Teambition`、`StackOverflow`、`Pinterest`、人人、华为、企业微信、酷家乐、`Gitlab`、美团、饿了么和推特等第三方平台的授权登录还在继续扩展。

配置示例：

```ini
justauth.enabled=true
justauth.type.github.client-id=101d*************8b3a
justauth.type.github.client-secret=58e*************************5edd
justauth.type.github.redirect-uri=http://127.0.0.1/demo/oauth/github/callback
justauth.type.qq.client-id=10******85
justauth.type.qq.client-secret=1f7d************************d629e
justauth.type.qq.redirect-uri=http://127.0.0.1/demo/oauth/qq/callback
justauth.type.wechat.client-id=wxdcb******4ff4
justauth.type.wechat.client-secret=b4e9dc************************a08ed6d
justauth.type.wechat.redirect-uri=http://127.0.0.1/demo/oauth/wechat/callback
justauth.type.google.client-id=716******17-6db******vh******ttj320i******userco******t.com
justauth.type.google.client-secret=9IBorn************7-E
justauth.type.google.redirect-uri=http://127.0.0.1/demo/oauth/google/callback
justauth.type.microsoft.client-id=7bdce8******************e194ad76c1b
justauth.type.microsoft.client-secret=Iu0zZ4************************tl9PWan_.
justauth.type.microsoft.redirect-uri=https://127.0.0.1/demo/oauth/microsoft/callback
justauth.type.mi.client-id=288************2994
justauth.type.mi.client-secret=nFeTt89************************==
justauth.type.mi.redirect-uri=http://127.0.0.1/demo/oauth/mi/callback
justauth.type.wechat_enterprise.client-id=ww58******f3************fbc
justauth.type.wechat_enterprise.client-secret=8G6PCr00j************************rgk************AyzaPc78
justauth.type.wechat_enterprise.redirect-uri=http://127.0.0.1/demo/oauth/wechat_enterprise/callback
justauth.type.wechat_enterprise.agent-id=1*******2
```

**(1) 在`Mysql`中创建相应表**

```sql
DROP TABLE IF EXISTS `justauth`.`t_ja_user`;
CREATE TABLE  `justauth`.`t_ja_user` (
    `uuid` varchar(64) NOT NULL COMMENT '用户第三方系统的唯一id',
    `username` varchar(100) DEFAULT NULL COMMENT '用户名',
    `nickname` varchar(100) DEFAULT NULL COMMENT '用户昵称',
    `avatar` varchar(255) DEFAULT NULL COMMENT '用户头像',
    `blog` varchar(255) DEFAULT NULL COMMENT '用户网址',
    `company` varchar(50) DEFAULT NULL COMMENT '所在公司',
    `location` varchar(255) DEFAULT NULL COMMENT '位置',
    `email` varchar(50) DEFAULT NULL COMMENT '用户邮箱',
    `gender` varchar(10) DEFAULT NULL COMMENT '性别',
    `remark` varchar(500) DEFAULT NULL COMMENT '用户备注（各平台中的用户个人介绍）',
    `source` varchar(20) DEFAULT NULL COMMENT '用户来源',
    PRIMARY KEY (`uuid`)
);
```

**(2) 依赖包**

```xml
<!-- springboot 父节点区域 -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.0.2.RELEASE</version>
</parent>
<dependencies>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>com.xkcoding.justauth</groupId>
        <artifactId>justauth-spring-boot-starter</artifactId>
        <version>1.4.0</version>
    </dependency>
    <dependency>
        <groupId>org.apache.commons</groupId>
        <artifactId>commons-pool2</artifactId>
    </dependency>
    <dependency>
        <groupId>com.baomidou</groupId>
        <artifactId>mybatis-plus-boot-starter</artifactId>
        <version>3.3.1</version>
    </dependency>
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-redis</artifactId>
    </dependency>
</dependencies>
```

**(3) 主启动目录中创建资源文件夹`resources`文件夹，并创建`application.properties`文件**

`application.properties`文件中内容：

```ini
server.port=8310

spring.datasource.driverClassName=com.mysql.jdbc.Driver
spring.datasource.url=jdbc:mysql://192.168.1.8:3860/test?useSSL=false&useUnicode=true&characterEncoding=utf-8&zeroDateTimeBehavior=convertToNull
spring.datasource.username=mysql
spring.datasource.password=123456
spring.redis.host=192.168.1.8
spring.redis.port=8302
#spring.redis.password=123456
spring.redis.timeout=2000ms
spring.redis.database=0
spring.redis.lettuce.pool.maxActive=8
spring.redis.lettuce.pool.maxIdle=8
mybatis-plus.mapper-locations=classpath:mapper/*.xml
mybatis-plus.type-aliases-package=Entity
mybatis-plus.global-config.db-config.id-type=AUTO
mybatis-plus.configuration.log-impl=org.apache.ibatis.logging.stdout.StdOutImpl
logging.level.com.xkcoding=debug
justauth.enabled=true
justauth.type.BAIDU.client-id=xxxxxx
justauth.type.BAIDU.client-secret=xxxxxx
justauth.type.BAIDU.redirect-uri=http://127.0.0.1:8310/oauth/baidu/callback
justauth.type.GITEE.client-id=xxx
justauth.type.GITEE.client-secret=xxx
justauth.type.GITEE.redirect-uri=http://127.0.0.1:8310/oauth/gitee/callback
justauth.cache.type=redis
justauth.cache.prefix=JUATAUTH::STATE::
justauth.cache.timeout=3m
```

**(4) 创建`Configure`包，并创建`RedisConfig.java`文件**

`RedisConfig.java`文件中内容：

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.boot.autoconfigure.condition.ConditionalOnMissingBean;
import org.springframework.cache.Cache;
import org.springframework.cache.annotation.CachingConfigurerSupport;
import org.springframework.cache.annotation.EnableCaching;
import org.springframework.cache.interceptor.CacheErrorHandler;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.cache.RedisCacheConfiguration;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.Jackson2JsonRedisSerializer;
import org.springframework.data.redis.serializer.RedisSerializationContext;
import org.springframework.data.redis.serializer.StringRedisSerializer;

@EnableCaching
@Configuration
public class RedisConfig extends CachingConfigurerSupport {

    Logger log = LoggerFactory.getLogger(RedisConfig.class);

    @Bean(name = "redisTemplate")
    @ConditionalOnMissingBean(name = "redisTemplate")
    public RedisTemplate<Object, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) {
        RedisTemplate<Object, Object> template = new RedisTemplate<>();
        template.setValueSerializer(new Jackson2JsonRedisSerializer<Object>(Object.class));
        template.setHashValueSerializer(new Jackson2JsonRedisSerializer<>(Object.class));
        template.setKeySerializer(new StringRedisSerializer());
        template.setHashKeySerializer(new StringRedisSerializer());
        template.setConnectionFactory(redisConnectionFactory);
        return template;
    }

    /**
	 * 缓存错误处理
	 */
    @Bean
    public CacheErrorHandler errorHandler() {

        return new CacheErrorHandler() {
            @Override
            public void handleCacheGetError(RuntimeException e, Cache cache, Object key) {
                log.error("Redis occur handleCacheGetError：key -> [{}]", key, e);
            }

            @Override
            public void handleCachePutError(RuntimeException e, Cache cache, Object key, Object value) {
                log.error("Redis occur handleCachePutError：key -> [{}]；value -> [{}]", key, value, e);
            }

            @Override
            public void handleCacheEvictError(RuntimeException e, Cache cache, Object key) {
                log.error("Redis occur handleCacheEvictError：key -> [{}]", key, e);
            }

            @Override
            public void handleCacheClearError(RuntimeException e, Cache cache) {
                log.error("Redis occur handleCacheClearError：", e);
            }
        };
    }

    /**
	 * 注入Redis缓存配置类
	 * 
	 * @return
	 */
    @Bean(name = { "redisCacheConfiguration" })
    public RedisCacheConfiguration redisCacheConfiguration() {
        RedisCacheConfiguration configuration = RedisCacheConfiguration.defaultCacheConfig();
        configuration = configuration
            .serializeKeysWith(
            RedisSerializationContext.SerializationPair.fromSerializer(new StringRedisSerializer()))
            .serializeValuesWith(RedisSerializationContext.SerializationPair
                                 .fromSerializer(new Jackson2JsonRedisSerializer<>(Object.class)));
        return configuration;
    }

}
```

**(5) 在`Configure`包中创建`JustAuthTokenCache.java`文件**

`JustAuthTokenCache.java`文件中内容：

```java
import java.util.LinkedList;
import java.util.List;
import java.util.Objects;
import javax.annotation.PostConstruct;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.core.BoundHashOperations;
import org.springframework.data.redis.core.RedisTemplate;
import com.alibaba.fastjson.JSONObject;
import me.zhyd.oauth.model.AuthToken;

@Configuration
public class JustAuthTokenCache {


    @SuppressWarnings("rawtypes")
    @Autowired
    private RedisTemplate redisTemplate;

    private BoundHashOperations<String, String, AuthToken> valueOperations;

    @SuppressWarnings("unchecked")
    @PostConstruct
    public void init() {
        valueOperations = redisTemplate.boundHashOps("JUSTAUTH::TOKEN");
    }

    /**
	 * 保存Token
	 * 
	 * @param uuid     用户uuid
	 * @param authUser 授权用户
	 * @return
	 */
    public AuthToken saveorUpdate(String uuid, AuthToken authToken) {
        valueOperations.put(uuid, authToken);
        return authToken;
    }

    /**
	 * 根据用户uuid查询Token
	 * 
	 * @param uuid 用户uuid
	 * @return
	 */
    public AuthToken getByUuid(String uuid) {
        Object token = valueOperations.get(uuid);
        if (null == token) {
            return null;
        }
        return JSONObject.parseObject(JSONObject.toJSONString(token), AuthToken.class);
    }

    /**
	 * 查询所有Token
	 * 
	 * @return
	 */
    public List<AuthToken> listAll() {
        return new LinkedList<>(Objects.requireNonNull(valueOperations.values()));
    }

    /**
	 * 根据用户uuid移除Token
	 * 
	 * @param uuid 用户uuid
	 * @return
	 */
    public void remove(String uuid) {
        valueOperations.delete(uuid);
    }

}
```

**(6) 创建`Entity`包，并创建`JustAuthUser.java`文件**

`JustAuthUser.java`文件中内容：

```java
import java.io.Serializable;

import com.alibaba.fastjson.JSONObject;
import com.baomidou.mybatisplus.annotation.IdType;
import com.baomidou.mybatisplus.annotation.TableField;
import com.baomidou.mybatisplus.annotation.TableId;
import com.baomidou.mybatisplus.annotation.TableName;

import me.zhyd.oauth.model.AuthToken;
import me.zhyd.oauth.model.AuthUser;

@TableName(value = "t_ja_user")
public class JustAuthUser extends AuthUser implements Serializable {

    private static final long serialVersionUID = 1L;

    /**
	 * 用户第三方系统的唯一id。在调用方集成该组件时，可以用uuid + source唯一确定一个用户
	 */
    @TableId(type = IdType.INPUT)
    private String uuid;

    /**
	 * 用户授权的token信息
	 */
    @TableField(exist = false)
    private AuthToken token;

    /**
	 * 第三方平台返回的原始用户信息
	 */
    @TableField(exist = false)
    private JSONObject rawUserInfo;

    /**
	 * 自定义构造函数
	 * 
	 * @param authUser 授权成功后的用户信息，根据授权平台的不同，获取的数据完整性也不同
	 */
    public JustAuthUser(AuthUser authUser) {
        super(authUser.getUuid(), authUser.getUsername(), authUser.getNickname(), authUser.getAvatar(),
              authUser.getBlog(), authUser.getCompany(), authUser.getLocation(), authUser.getEmail(),
              authUser.getRemark(), authUser.getGender(), authUser.getSource(), authUser.getToken(),
              authUser.getRawUserInfo());
    }
}
```

**(7) 创建`Dao`包，并创建`JustAuthUserMapper.java`文件**

`JustAuthUserMapper.java`文件中内容：

```java
import org.apache.ibatis.annotations.Mapper;
import com.baomidou.mybatisplus.core.mapper.BaseMapper;
import Entity.*;

@Mapper
public interface JustAuthUserMapper extends BaseMapper<JustAuthUser> {

}
```

**(8) 创建`service`包，并创建`JustAuthUserService.java`文件**

`JustAuthUserService.java`文件中内容：

```java
import com.baomidou.mybatisplus.extension.service.IService;
import Entity.*;
public interface JustAuthUserService extends IService<JustAuthUser> {

    /**
	 * 保存或更新授权用户
	 * 
	 * @param justAuthUser 授权用户
	 * @return
	 */
    boolean saveOrUpdate(JustAuthUser justAuthUser);

    /**
	 * 根据用户uuid查询信息
	 * 
	 * @param uuid 用户uuid
	 * @return
	 */
    JustAuthUser getByUuid(String uuid);

    /**
	 * 根据用户uuid移除信息
	 * 
	 * @param uuid 用户uuid
	 * @return
	 */
    boolean removeByUuid(String uuid);
}
```

**(9) 在`service`包中创建`JustAuthUserServiceImpl.java`文件**

`JustAuthUserServiceImpl.java`文件中内容：

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import com.baomidou.mybatisplus.extension.service.impl.ServiceImpl;
import Configure.JustAuthTokenCache;
import Dao.*;
import Entity.*;

@Service
public class JustAuthUserServiceImpl extends ServiceImpl<JustAuthUserMapper, JustAuthUser>
    implements JustAuthUserService {

    @Autowired
    private JustAuthTokenCache justAuthTokenCache;

    /**
	 * 保存或更新授权用户
	 * 
	 * @param justAuthUser 授权用户
	 * @return
	 */
    @Override
    public boolean saveOrUpdate(JustAuthUser justAuthUser) {
        justAuthTokenCache.saveorUpdate(justAuthUser.getUuid(), justAuthUser.getToken());
        return super.saveOrUpdate(justAuthUser);
    }

    /**
	 * 根据用户uuid查询信息
	 * 
	 * @param uuid 用户uuid
	 * @return
	 */
    @Override
    public JustAuthUser getByUuid(String uuid) {
        JustAuthUser justAuthUser = super.getById(uuid);
        if (justAuthUser != null) {
            justAuthUser.setToken(justAuthTokenCache.getByUuid(uuid));
        }
        return justAuthUser;
    }

    /**
	 * 根据用户uuid移除信息
	 * 
	 * @param uuid 用户uuid
	 * @return
	 */
    @Override
    public boolean removeByUuid(String uuid) {
        justAuthTokenCache.remove(uuid);
        return super.removeById(uuid);
    }

}
```

**(10) 创建`domain`包，并创建`ResultCode.java`文件**

`ResultCode.java`文件中内容：

```java
import com.fasterxml.jackson.annotation.JsonFormat;

@JsonFormat(shape = JsonFormat.Shape.OBJECT)
public enum ResultCode {
    /*
    请求返回状态码和说明信息
     */
    SUCCESS(200, "成功"),
    BAD_REQUEST(400, "参数或者语法不对"),
    UNAUTHORIZED(401, "认证失败"),
    LOGIN_ERROR(401, "登陆失败，用户名或密码无效"),
    FORBIDDEN(403, "禁止访问"),
    NOT_FOUND(404, "请求的资源不存在"),
    OPERATE_ERROR(405, "操作失败，请求操作的资源不存在"),
    TIME_OUT(408, "请求超时"),

    SERVER_ERROR(500, "服务器内部错误"),
    ;
    private int code;
    private String msg;

    ResultCode(int code, String msg) {
        this.code = code;
        this.msg = msg;
    }

    public void setCode(int code) {
        this.code = code;
    }

    public int getCode() {
        return code;
    }

    public String getMsg() {
        return msg;
    }
}
```

**(11) 在`domain`包中创建`PageResult.java`文件**

`PageResult.java`文件中内容：

```java
public class PageResult {

    private int page;
    private int rows;
    private int total;
    private Object data;

    public PageResult(int page, int rows, int total, Object data) {
        this.page = page;
        this.rows = rows;
        this.total = total;
        this.data = data;
    }

    public void setPage(int page) {
        this.page = page;
    }

    public int getPage() {
        return page;
    }

    public void setRows(int rows) {
        this.rows = rows;
    }

    public int getRows() {
        return rows;
    }

    public void setTotal(int total) {
        this.total = total;
    }

    public int getTotal() {
        return total;
    }

    public void setData(Object data) {
        this.data = data;
    }

    public Object getData() {
        return data;
    }

}
```

**(12) 在`domain`包中创建`ResultJson.java`文件**

`ResultJson.java`文件中内容：

```java
public class ResultJson {

    private int code;
    private String msg;
    private Object data;

    public static ResultJson Success() {
        return Success("");
    }

    public static ResultJson Success(Object o) {
        return new ResultJson(ResultCode.SUCCESS, o);
    }

    public static ResultJson Failure(Object o) {
        return new ResultJson(ResultCode.BAD_REQUEST, o);
    }

    public static ResultJson Failure(ResultCode code) {
        return Failure(code, "");
    }

    public static ResultJson Failure(ResultCode code, Object o) {
        return new ResultJson(code, o);
    }

    public ResultJson (ResultCode resultCode) {
        setResultCode(resultCode);
    }

    public ResultJson (ResultCode resultCode,Object data) {
        setResultCode(resultCode);
        this.data = data;
    }

    public void setResultCode(ResultCode resultCode) {
        this.code = resultCode.getCode();
        this.msg = resultCode.getMsg();
    }

    public void setCode(int code) {
        this.code = code;
    }
    public int getCode() {
        return code;
    }

    public void setMsg(String msg) {
        this.msg = msg;
    }

    public String getMsg() {
        return msg;
    }
    public void setData(Object data) {
        this.data = data;
    }

    public Object getData() {
        return data;
    }


    public ResultJson(int code, String msg, Object data) {
        this.code = code;
        this.msg = msg;
        this.data = data;
    }

    @Override
    public String toString() {
        return "{" +
            "\"code\":" + code +
            ", \"msg\":\"" + msg + '\"' +
            ", \"data\":\"" + data + '\"'+
            '}';
    }
}
```

**(13) 创建`controller`包，并创建`AuthController.java`文件**

`AuthController.java`文件中内容：

```java
import java.io.IOException;
import javax.servlet.http.HttpServletResponse;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.bind.annotation.RestController;
import com.alibaba.fastjson.JSON;
import com.xkcoding.justauth.AuthRequestFactory;
import Entity.JustAuthUser;
import domain.ResultCode;
import domain.ResultJson;
import me.zhyd.oauth.model.AuthCallback;
import me.zhyd.oauth.model.AuthResponse;
import me.zhyd.oauth.model.AuthToken;
import me.zhyd.oauth.model.AuthUser;
import me.zhyd.oauth.request.AuthRequest;
import me.zhyd.oauth.utils.AuthStateUtils;
import service.JustAuthUserService;

@RestController
@RequestMapping("/oauth")
public class AuthController {

    Logger log = LoggerFactory.getLogger(AuthController.class);

    @Autowired
    private AuthRequestFactory factory;

    @Autowired
    private JustAuthUserService justAuthUserService;

    /**
	 * 登录
	 * 
	 * @param type     第三方系统类型，例如：gitee/baidu
	 * @param response
	 * @throws IOException
	 */
    @GetMapping("/login/{type}")
    public void login(@PathVariable String type, HttpServletResponse response) throws IOException {
        AuthRequest authRequest = factory.get(type);
        response.sendRedirect(authRequest.authorize(AuthStateUtils.createState()));
    }

    /**
	 * 登录回调
	 * 
	 * @param type     第三方系统类型，例如：gitee/baidu
	 * @param callback
	 * @return
	 */
    @SuppressWarnings("unchecked")
    @RequestMapping("/{type}/callback")
    public ResultJson login(@PathVariable String type, AuthCallback callback) {
        AuthRequest authRequest = factory.get(type);
        AuthResponse<AuthUser> response = authRequest.login(callback);
        log.info("【response】= {}", JSON.toJSONString(response));

        if (response.ok()) {
            justAuthUserService.saveOrUpdate(new JustAuthUser(response.getData()));
            return ResultJson.Success(JSON.toJSONString(response));
        }
        return ResultJson.Failure(response.getMsg());
    }

    /**
	 * 收回
	 * 
	 * @param type 第三方系统类型，例如：gitee/baidu
	 * @param uuid 用户uuid
	 * @return
	 */
    @SuppressWarnings("unchecked")
    @RequestMapping("/revoke/{type}/{uuid}")
    public ResultJson revoke(@PathVariable String type, @PathVariable String uuid) {
        AuthRequest authRequest = factory.get(type);

        JustAuthUser justAuthUser = justAuthUserService.getByUuid(uuid);
        if (null == justAuthUser) {
            return ResultJson.Failure("用户不存在");
        }

        AuthResponse<AuthToken> response = null;
        try {
            response = authRequest.revoke(justAuthUser.getToken());
            if (response.ok()) {
                justAuthUserService.removeByUuid(justAuthUser.getUuid());
                return ResultJson.Success("用户 [" + justAuthUser.getUsername() + "] 的 授权状态 已收回！");
            }
            return ResultJson.Failure("用户 [" + justAuthUser.getUsername() + "] 的 授权状态 收回失败！" + response.getMsg());
        } catch (Exception e) {
            return ResultJson.Failure(ResultCode.BAD_REQUEST);
        }
    }

    /**
	 * 刷新
	 * 
	 * @param type 第三方系统类型，例如：gitee/baidu
	 * @param uuid 用户uuid
	 * @return
	 */
    @SuppressWarnings("unchecked")
    @RequestMapping("/refresh/{type}/{uuid}")
    @ResponseBody
    public ResultJson refresh(@PathVariable String type, @PathVariable String uuid) {
        AuthRequest authRequest = factory.get(type);

        JustAuthUser justAuthUser = justAuthUserService.getByUuid(uuid);
        if (null == justAuthUser) {
            return ResultJson.Failure("用户不存在");
        }

        AuthResponse<AuthToken> response = null;
        try {
            response = authRequest.refresh(justAuthUser.getToken());
            if (response.ok()) {
                justAuthUser.setToken(response.getData());
                justAuthUserService.saveOrUpdate(justAuthUser);
                return ResultJson.Success("用户 [" + justAuthUser.getUsername() + "] 的 access token 已刷新！新的 accessToken: "
                                          + response.getData().getAccessToken());
            }
            return ResultJson.Failure("用户 [" + justAuthUser.getUsername() + "] 的 access token 刷新失败！" + response.getMsg());
        } catch (Exception e) {
            return ResultJson.Failure(ResultCode.BAD_REQUEST);
        }
    }

}
```

**(14) 主启动目录中内容**

```java
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.context.annotation.ComponentScan;

@EnableAutoConfiguration
@ComponentScan("Configure,service,controller")
@MapperScan("Dao")
public class Start {

    public static void main(String[] args) {
        SpringApplication.run(Start.class, args);
    }

}
```

(15) 测试验证

```
[登录]：
  浏览器访问地址：http://127.0.0.1:8443/oauth/login/baidu，即可获取到对应信息。
  请观察数据库及Redis中数据变化。
[刷新Token]：
  浏览器访问地址：http://127.0.0.1:8443/oauth/refresh/baidu/[uuid]，其中uuid为数据库表中的uuid。
  目前插件中已实现的可刷新的第三方应用有限，请查看AuthRequest.refresh方法的实现类。
  请观察Redis中数据变化。
[移除Token]：
  浏览器访问地址：http://127.0.0.1:8443/oauth/revoke/baidu/[uuid]，其中uuid为数据库表中的uuid。
  目前插件中已实现的可刷新的第三方应用有限，请查看AuthRequest.revoke方法的实现类。
  请观察数据库及Redis中数据变化。
```

***

##### <font size="4" color="red">48. Springboot集成Xjar程序加密</font>

​		`Xjar`基于对`jar`包内资源的加密以及拓展`ClassLoader`来构建的一套程序加密启动，动态解密运行的方案，避免源码泄露或反编译。它不需要侵入代码，只需要把编译好的`jar`包通过工具加密即可。

**(1) 依赖包**

```xml
<repositories>
    <repository>
        <id>jitpack</id>
        <url>https://jitpack.io</url>
    </repository>
</repositories>
<dependencies>
    <dependency>
        <groupId>com.github.core-lib</groupId>
        <artifactId>xjar</artifactId>
        <version>4.0.0</version>
    </dependency>
    <dependency>
        <groupId>org.apache.commons</groupId>
        <artifactId>commons-compress</artifactId>
        <version>1.20</version>
    </dependency>
</dependencies>
```

**(2) 创建`test.java`文件**

`test.java`文件中内容：

```java
import io.xjar.XCryptos;

public class test {

    public static void encrypt() throws Exception {
        XCryptos.encryption()
            //项目生成的jar
            .from("F://Java//Test01//SpringStart.jar")
            // 加密的密码
            .use("testaa1111122222")
            .include("/**/*.class")
            .include("/**/*.xml")
            .include("/**/*.yml")
            .include("/**/*.properties")
            .to("F://Java//Test01//test.jar");
    }

    public static void main(String[] args) throws Exception {
        encrypt();
    }
}
```

**(3) 将生成文件`xjar.go`放入`GoLang`语言环境中编译**

```shell
go bulid xjar.go
```

> **注：**`xjar.go`在`Windows`生成`xjar.exe`文件，在`Linux`生成`xjar`文件

**(4) 启动`Springboot`项目**

`Windows`启动：

```shell
xjar.exe java -jar test.jar
```

`Linux`启动：

```shell
nohup ./xjar java -jar test.jar
```

***

##### <font size="4" color="red">49. Springboot集成Log4j2操作Sentry异常监控系统</font>

**(1) 依赖包**

```xml
<!-- springboot 父节点区域 -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.4.0</version>
</parent>
<dependencies>
    <dependency>  
        <groupId>org.springframework.boot</groupId>  
        <artifactId>spring-boot-starter-web</artifactId>  
        <exclusions><!-- 去掉springboot默认配置 -->  
            <exclusion>  
                <groupId>org.springframework.boot</groupId>  
                <artifactId>spring-boot-starter-logging</artifactId>  
            </exclusion>  
        </exclusions>  
    </dependency> 
    <dependency>
        <groupId>io.sentry</groupId>
        <artifactId>sentry-log4j2</artifactId>
        <version>1.7.23</version>
    </dependency>
    <dependency> <!-- 引入log4j2依赖 -->  
        <groupId>org.springframework.boot</groupId>  
        <artifactId>spring-boot-starter-log4j2</artifactId>  
    </dependency> 
</dependencies>
```

**(2) 主启动目录中创建资源文件夹`resources`文件夹，并创建`application.properties`文件**

`application.properties`文件中内容：

```ini
logging.config=classpath:log4j2.xml
```

**(3) 在`resources`资源文件夹中创建`sentry.properties`文件**

`sentry.properties`文件中内容：

```ini
stacktrace.app.packages=
dsn=http://edb111ae20f6464baf358d1b0c2662ae@192.168.0.142:9000/17
```

**(4) 在`resources`资源文件夹中创建`log4j2.xml`文件**

`log4j2.xml`文件中内容：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration status="warn" packages="org.apache.logging.log4j.core,io.sentry.log4j2">
    <appenders>
        <Console name="Console" target="SYSTEM_OUT">
            <PatternLayout pattern="%d{HH:mm:ss.SSS} [%t] %-5level %logger{36} - %msg%n" />
        </Console>
        <Sentry name="Sentry"/>
    </appenders>
    <loggers>
        <root level="INFO">
            <appender-ref ref="Console" />
            <!-- Note that the Sentry logging threshold is overridden to the WARN level -->
            <appender-ref ref="Sentry" level="WARN" />
        </root>
    </loggers>
</configuration>
```

**(5) 创建`controller`包，并创建`controller.java`文件**

`controller.java`文件中内容：

```java
import org.apache.logging.log4j.LogManager;
import org.apache.logging.log4j.Logger;
import org.springframework.web.bind.annotation.CrossOrigin;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;

@CrossOrigin
@RestController
public class controller {
    Logger logger = LogManager.getLogger(LogManager.ROOT_LOGGER_NAME);

    @RequestMapping(value = "/index",method = RequestMethod.GET)
    public String Index() throws Exception {
        logger.error("测试sentry打印error日志");  
        return "成功";
    }

}
```

**(6) 主启动目录中内容**

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.context.annotation.ComponentScan;

@EnableAutoConfiguration
@ComponentScan("controller")
public class Start {

    public static void main(String[] args) {
        SpringApplication.run(Start.class, args);
    }
}
```

***

##### <font size="4" color="red">50. Springboot集成Logback操作Sentry异常监控系统</font>

**(1) 依赖包**

```xml
<!-- springboot 父节点区域 -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.4.0</version>
</parent>
<dependencies>
    <dependency>  
        <groupId>org.springframework.boot</groupId>  
        <artifactId>spring-boot-starter-web</artifactId>  
        <exclusions>
            <exclusion>
                <groupId>org.slf4j</groupId>
                <artifactId>slf4j-log4j12</artifactId>
            </exclusion>
        </exclusions>
    </dependency> 
    <dependency>
        <groupId>io.sentry</groupId>
        <artifactId>sentry-logback</artifactId>
        <version>1.7.23</version>
    </dependency>
</dependencies>
```

**(2) 主启动目录中创建资源文件夹`resources`文件夹，并创建`application.properties`文件**

`application.properties`文件中内容：

```ini
logging.config=classpath:logback.xml
```

**(3) 在`resources`资源文件夹中创建`sentry.properties`文件**

`sentry.properties`文件中内容：

```ini
stacktrace.app.packages=
dsn=http://edb111ae20f6464baf358d1b0c2662ae@192.168.0.142:9000/17
```

**(4) 在`resources`资源文件夹中创建`logback.xml`文件**

`logback.xml`文件中内容：

```xml
<configuration>
    <!-- Configure the Console appender -->
    <appender name="Console" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>

    <!-- Configure the Sentry appender, overriding the logging threshold to the WARN level -->
    <appender name="Sentry" class="io.sentry.logback.SentryAppender">
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>WARN</level>
        </filter>
    </appender>

    <!-- Enable the Console and Sentry appenders, Console is provided as an example
 of a non-Sentry logger that is set to a different logging threshold -->
    <root level="INFO">
        <appender-ref ref="Console" />
        <appender-ref ref="Sentry" />
    </root>
</configuration>
```

**(5) 创建`controller`包，并创建`controller.java`文件**

`controller.java`文件中内容：

```java
import org.apache.logging.log4j.LogManager;
import org.apache.logging.log4j.Logger;
import org.springframework.web.bind.annotation.CrossOrigin;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;

@CrossOrigin
@RestController
public class controller {
    Logger logger = LogManager.getLogger(LogManager.ROOT_LOGGER_NAME);

    @RequestMapping(value = "/index",method = RequestMethod.GET)
    public String Index() throws Exception {
        logger.error("测试sentry打印error日志");  
        return "成功";
    }

}
```

**(6) 主启动目录中内容**

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.context.annotation.ComponentScan;

@EnableAutoConfiguration
@ComponentScan("controller")
public class Start {

    public static void main(String[] args) {
        SpringApplication.run(Start.class, args);
    }
}
```

***

##### <font size="4" color="red">51. Springboot集成LucySheet在线协同</font>

**(1) 依赖包**

```xml
<!-- springboot 父节点区域 -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.2.0.RELEASE</version>
</parent>
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-mongodb</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-websocket</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-freemarker</artifactId>
    </dependency>
    <dependency>
        <groupId>cn.hutool</groupId>
        <artifactId>hutool-all</artifactId>
        <version>4.3.2</version>
    </dependency>
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>fastjson</artifactId>
        <version>1.2.73</version>
    </dependency>
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

**(2) 创建资源文件夹`resources`文件夹，并创建`application.properties`文件**

`application.properties`文件中内容：

```ini
server.port=8310
spring.freemarker.request-context-attribute=request
spring.freemarker.suffix=.html
spring.freemarker.content-type=text/html
spring.resources.static-locations=classpath:/resources/static/
spring.freemarker.enabled=true
spring.freemarker.cache=false
spring.freemarker.charset=UTF-8
spring.freemarker.allow-request-override=false
spring.freemarker.expose-request-attributes=true
spring.freemarker.expose-session-attributes=true
spring.freemarker.expose-spring-macro-helpers=true
spring.data.mongodb.uri=mongodb://127.0.0.1:8892/wb?authSource=admin&readPreference=primary&ssl=false
```

**(3) 在`resuorces`资源文件夹中创建`templates`文件夹，并创建`websocket.html`文件**

`websocket.html`文件中内容：

```html
<!DOCTYPE html>
<html>
    <head lang='zh'>
        <meta charset='utf-8'>
        <meta http-equiv="X-UA-Compatible" content="IE=edge"/>
        <meta name="renderer" content="webkit"/>
        <meta name="viewport" content="width=device-width, initial-scale=1,user-scalable=0"/>
        <title>websocket--luckysheet</title>
        <link rel='stylesheet' href='/lib/plugins/css/pluginsCss.css'/>
        <link rel='stylesheet' href='/lib/plugins/plugins.css'/>
        <link rel='stylesheet' href='/lib/css/luckysheet.css'/>
        <link rel='stylesheet' href='/lib/assets/iconfont/iconfont.css'/>
        <script src="/lib/plugins/js/plugin.js"></script>
        <script src="/lib/luckysheet.umd.js"></script>
    </head>
    <body>
        <div id="${wb.option.container}"
             style="margin:0px;padding:0px;position:absolute;width:100%;height:100%;left: 0px;top: 0px;"></div>

        </div>
    </body>
<script>
    var localurl =  "//" + window.location.host;
    // 配置项
    var options = {
        container: "${wb.option.container}", // 设定DOM容器的id
        title: "${wb.option.title}", // 设定表格名称
        lang: "${wb.option.lang}",
        allowUpdate: true,
        showinfobar: true,//作用：是否显示顶部信息栏
        functionButton: '<button id="" class="btn btn-primary" onclick="clicks()" style="padding:3px 6px;font-size: 12px;margin-right: 10px;">保存</button> <button id="" class="btn btn-primary btn-danger" style=" padding:3px 6px; font-size: 12px; margin-right: 85px;" onclick="downExcelData()">下载</button>',
        loadUrl: window.location.protocol + localurl + "/load/${wb.id}",
        loadSheetUrl: window.location.protocol + localurl + "/loadSheet/${wb.id}",
        updateUrl: "ws://"+localurl + "/ws/" + Math.round(Math.random() * 100) + "/${wb.id}"
        // 更多其他设置...
    }

    // 初始化表格
    luckysheet.create(options)

    function uploadExcelData() {
        //console.log(luckysheet.getAllSheets());
        //console.log("lll=" + JSON.stringify(luckysheet.getAllSheets()));
        //上传例子，可以把这个数据保存到服务器上。下次可以从服务器直接加载luckysheet数据了。
        $.post("/excel/uploadData", {
            exceldatas: JSON.stringify(luckysheet.getAllSheets()),
            title: options.title,
        }, function (data) {
            //console.log("data = " + data)
            alert("保存成功！")
        });
    }

    function downExcelData() {
        //这里你要自己写个后台接口，处理上传上来的Excel数据，用post传输。我用的是Java后台处理导出！这里只是写了post请求的写法
        $.post("/excel/downfile", {
            exceldatas: JSON.stringify(luckysheet.getAllSheets()),
        }, function (data) {
            //console.log("data = " + data)
        });
    }
</script>
</html>
```

**(4) 创建资源文件夹`static`文件夹，将`LuckySheet`相关资源放入**

**(5) 创建`utils`包，并创建`PakoGzipUtils.java`文件**

`PakoGzipUtils.java`文件中内容：

```java
import java.io.ByteArrayInputStream;
import java.io.ByteArrayOutputStream;
import java.io.IOException;
import java.net.URLDecoder;
import java.net.URLEncoder;
import java.util.zip.GZIPInputStream;
import java.util.zip.GZIPOutputStream;

public class PakoGzipUtils {
    public static String compress(String str) throws IOException {
        if (str == null || str.length() == 0) {
            return str;
        }
        ByteArrayOutputStream out = new ByteArrayOutputStream();
        GZIPOutputStream gzip = new GZIPOutputStream(out);
        gzip.write(str.getBytes());
        gzip.close();
        return out.toString("ISO-8859-1");
    }

    /**
     * @param str：
     * @return 解压字符串  生成正常字符串。
     * @throws IOException
     */
    public static String uncompress(String str) throws IOException {
        if (str == null || str.length() == 0) {
            return str;
        }
        ByteArrayOutputStream out = new ByteArrayOutputStream();
        ByteArrayInputStream in = new ByteArrayInputStream(str.getBytes("ISO-8859-1"));
        GZIPInputStream gunzip = new GZIPInputStream(in);
        byte[] buffer = new byte[256];
        int n;
        while ((n = gunzip.read(buffer)) >= 0) {
            out.write(buffer, 0, n);
        }
        // toString()使用平台默认编码，也可以显式的指定如toString("GBK")
        return out.toString();
    }

    /**
     * @param jsUriStr :字符串类型为
     * @return 生成正常字符串
     * @throws IOException
     */
    public static String  unCompressURI(String jsUriStr) throws IOException {
        String gzipCompress=uncompress(jsUriStr);
        String result=URLDecoder.decode(gzipCompress,"UTF-8");
        return result;
    }
    /**
     * @param strData :字符串类型为： 正常字符串
     * @return 生成字符串类型为：
     */
    public static String  compress2URI(String strData) throws IOException {
        String encodeGzip=compress(strData);
        String jsUriStr=URLEncoder.encode(encodeGzip,"UTF-8");
        return jsUriStr;
    }
}
```

**(6) 在`utils`包中创建`SheetUtil.java`文件**

`SheetUtil.java`文件中内容：

```java
import java.util.ArrayList;
import java.util.List;

import cn.hutool.core.util.IdUtil;
import cn.hutool.json.JSONObject;

public class SheetUtil {

    /**
     * 获取sheet的默认option
     * @return
     */
    public static JSONObject getDefautOption() {

        JSONObject jsonObject = new JSONObject();
        jsonObject.put("container", "ecsheet");
        jsonObject.put("title", "ecsheet demo");
        jsonObject.put("lang", "zh");
        jsonObject.put("allowUpdate", true);
        jsonObject.put("loadUrl", "");
        jsonObject.put("loadSheetUrl", "");
        jsonObject.put("updateUrl", "");

        return jsonObject;
    }

    /**
     * 获取默认的sheetData
     * @return
     */
    public static List<JSONObject> getDefaultSheetData() {
        List<JSONObject> list = new ArrayList<>();

        for (int i = 1; i < 4; i++) {
            JSONObject jsonObject = new JSONObject();
            jsonObject.put("row", 84);
            jsonObject.put("column", 60);
            jsonObject.put("name", "sheet" + i);
            Integer index = i - 1;
            jsonObject.put("index", IdUtil.simpleUUID());
            jsonObject.put("order", i - 1);
            if (i == 1) {
                jsonObject.put("status", 1);
            } else {
                jsonObject.put("status", 0);
            }
            jsonObject.put("celldata", new ArrayList<JSONObject>() {
            });
            list.add(jsonObject);
        }
        return list;
    }

    /**
     * 获取默认的全部sheetData
     * @return
     */
    public static JSONObject getDefaultAllSheetData() {
        JSONObject result = new JSONObject();

        for (int i = 1; i < 4; i++) {
            JSONObject data = new JSONObject();
            data.put("r", 0);
            data.put("c", 0);
            data.put("v", new JSONObject());
            result.put("sheet" + i, data);
        }
        return result;
    }

    /**
     * 组装异步加载sheet所需的数据
     *
     * @param data
     * @return
     */
    public static JSONObject buildSheetData(List<JSONObject> data) {
        JSONObject result = new JSONObject();
        data.forEach((d) -> {
            result.put(d.get("index").toString(), d.get("celldata"));
        });
        return result;
    }
}
```

**(7) 创建`service`包，并创建`IMessageProcess.java`文件**

`IMessageProcess.java`文件中内容：

```java
import cn.hutool.json.JSONObject;

public interface IMessageProcess {
    void process(String gridKey,JSONObject message);
}
```

**(8) 在`service`包中创建`MessageProcess.java`文件**

`MessageProcess.java`文件中内容：

```java
import java.util.ArrayList;
import java.util.List;
import java.util.Optional;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import cn.hutool.core.util.ObjectUtil;
import cn.hutool.core.util.StrUtil;
import cn.hutool.json.JSONArray;
import cn.hutool.json.JSONObject;
import cn.hutool.json.JSONUtil;
import entity.WorkBookEntity;
import entity.WorkSheetEntity;
import repository.WorkBookRepository;
import repository.WorkSheetRepository;

@Service
public class MessageProcess implements IMessageProcess {

    @Autowired
    private WorkSheetRepository workSheetRepository;

    @Autowired
    private WorkBookRepository workBookRepository;

    @Override
    public void process(String wbId, JSONObject message) {
        //获取操作名
        String action = message.getStr("t");
        //获取sheet的index值
        String index = message.getStr("i");

        //如果是复制sheet，index的值需要另取
        if ("shc".equals(action)) {
            index = message.getJSONObject("v").getStr("copyindex");
        }

        //如果是删除sheet，index的值需要另取
        if ("shd".equals(action)) {
            index = message.getJSONObject("v").getStr("deleIndex");
        }

        //如果是恢复sheet，index的值需要另取
        if ("shre".equals(action)) {
            index = message.getJSONObject("v").getStr("reIndex");
        }

        WorkSheetEntity ws = workSheetRepository.findByindexAndwbId(index, wbId);
        System.out.println("操作："+action);
        switch (action) {
                //单个单元格刷新
            case "v":
                ws = singleCellRefresh(ws, message);
                break;
                //范围单元格刷新
            case "rv":
                ws = rangeCellRefresh(ws, message);
                break;
                //config操作
            case "cg":
                ws = configRefresh(ws, message);
                break;
                //通用保存
            case "all":
                ws = allRefresh(ws, message);
                break;
                //函数链操作
            case "fc":
                ws = calcChainRefresh(ws, message);
                break;
                //删除行或列
            case "drc":
                ws = drcRefresh(ws, message);
                break;
                //增加行或列
            case "arc":
                ws = arcRefresh(ws, message);
                break;
                //清除筛选
            case "fsc":
                ws = fscRefresh(ws, message);
                break;
                //恢复筛选
            case "fsr":
                ws = fscRefresh(ws, message);
                break;
                //新建sheet
            case "sha":
                ws = shaRefresh(wbId, message);
                break;
                //切换到指定sheet
            case "shs":
                shsRefresh(wbId, message);
                break;
                //复制sheet
            case "shc":
                ws = shcRefresh(ws, message);
                break;
                //修改工作簿名称
            case "na":
                naRefresh(wbId, message);
                break;
                //删除sheet
            case "shd":
                ws.setDeleteStatus(1);
                break;
                //删除sheet后恢复操作
            case "shre":
                ws.setDeleteStatus(0);
                break;
                //调整sheet位置
            case "shr":
                shrRefresh(wbId, message);
                break;
                //sheet属性(隐藏或显示)
            case "sh":
                ws = shRefresh(ws, message);
                break;
            default:
                break;
        }
        if (ObjectUtil.isNull(ws)) {
            return;
        }
        workSheetRepository.save(ws);

    }

    /**
     * 单个单元格刷新
     *
     * @param ws
     * @param message
     * @return
     */
    private WorkSheetEntity singleCellRefresh(WorkSheetEntity ws, JSONObject message) {
        //对celldata进行深拷贝
        JSONArray celldata = ObjectUtil.cloneByStream(ws.getData().getJSONArray("celldata"));
        if (StrUtil.isBlank(message.getStr("v"))) {
            celldata.forEach(c -> {
                JSONObject jsonObject = JSONUtil.parseObj(c);
                if (!jsonObject.isEmpty()) {
                    if (jsonObject.getLong("r") == message.getLong("r") && jsonObject.getLong("c") == message.getLong("c")) {
                        ws.getData().getJSONArray("celldata").remove(jsonObject);
                    }
                }
            });
        } else {
            JSONObject collectData = JSONUtil.createObj().put("r", message.getLong("r")).put("c", message.getLong("c")).put("v", message.getJSONObject("v"));

            List<String> flag = new ArrayList<>();
            celldata.forEach(c -> {
                JSONObject jsonObject = JSONUtil.parseObj(c);
                if (!jsonObject.isEmpty()) {
                    if (jsonObject.getLong("r") == message.getLong("r") && jsonObject.getLong("c") == message.getLong("c")) {
                        ws.getData().getJSONArray("celldata").remove(jsonObject);
                        ws.getData().getJSONArray("celldata").add(collectData);
                        flag.add("used");
                    }
                }
            });
            if (flag.isEmpty()) {
                ws.getData().getJSONArray("celldata").add(collectData);
            }
        }
        return ws;
    }


    /**
     * 范围单元格刷新
     *
     * @param ws
     * @param message
     * @return
     */
    private WorkSheetEntity rangeCellRefresh(WorkSheetEntity ws, JSONObject message) {
        JSONArray rowArray = message.getJSONObject("range").getJSONArray("row");
        JSONArray columnArray = message.getJSONObject("range").getJSONArray("column");
        JSONArray vArray = message.getJSONArray("v");
        JSONArray celldata = ObjectUtil.cloneByStream(ws.getData().getJSONArray("celldata"));
        int countRowIndex = 0;

        //遍历行列，对符合行列的内容进行更新
        for (int ri = (int) rowArray.get(0); ri <= (int) rowArray.get(1); ri++) {
            int countColumnIndex = 0;
            for (int ci = (int) columnArray.get(0); ci <= (int) columnArray.get(1); ci++) {
                List<String> flag = new ArrayList<>();
                Object newCell = JSONUtil.parseArray(vArray.get(countRowIndex)).get(countColumnIndex);
                JSONObject collectData = JSONUtil.createObj().put("r", ri).put("c", ci).put("v", newCell);
                int rowIndex = ri;
                int columnIndex = ci;
                celldata.forEach(cell -> {
                    JSONObject jsonObject = JSONUtil.parseObj(cell);

                    if (!jsonObject.isEmpty()) {
                        if (jsonObject.getInt("r") == rowIndex && jsonObject.getInt("c") == columnIndex) {
                            if ("null".equals(newCell.toString()) || JSONUtil.parseObj(newCell).isEmpty()) {
                                ws.getData().getJSONArray("celldata").remove(jsonObject);
                            } else {
                                ws.getData().getJSONArray("celldata").remove(jsonObject);
                                ws.getData().getJSONArray("celldata").add(collectData);

                            }
                            flag.add("used");
                        }
                    }
                });
                if (flag.isEmpty() && !JSONUtil.parseObj(newCell).isEmpty()) {
                    ws.getData().getJSONArray("celldata").add(collectData);
                }
                countColumnIndex++;
            }
            countRowIndex++;
        }


        return ws;
    }


    /**
     * config更新
     *
     * @param ws
     * @param message
     * @return
     */
    private WorkSheetEntity configRefresh(WorkSheetEntity ws, JSONObject message) {
        JSONObject v = message.getJSONObject("v");
        JSONObject newConfig = JSONUtil.createObj().put(message.getStr("k"), v);
        System.out.println(ws.getData());
        if(ws.getData().getJSONObject("config") != null) {
            if (ws.getData().getJSONObject("config").isEmpty()) {
                ws.getData().put("config", newConfig);
            } else {
                ws.getData().getJSONObject("config").put(message.getStr("k"), v);
            }
        }
        return ws;
    }


    /**
     * 通用保存
     *
     * @param ws
     * @param message
     * @return
     */
    private WorkSheetEntity allRefresh(WorkSheetEntity ws, JSONObject message) {
        if (message.getJSONObject("v").isEmpty()) {
            ws.getData().remove(message.getStr("k"));
        } else {
            ws.getData().put(message.getStr("k"), message.getJSONObject("v"));
        }
        return ws;
    }


    /**
     * 函数链操作
     *
     * @param ws
     * @param message
     * @return
     */
    private WorkSheetEntity calcChainRefresh(WorkSheetEntity ws, JSONObject message) {
        JSONObject value = message.getJSONObject("v");
        if (!ws.getData().containsKey("calcChain")) {
            ws.getData().put("calcChain", new JSONArray());
        }
        JSONArray calcChain = ws.getData().getJSONArray("calcChain");
        if ("add".equals(message.getStr("op"))) {
            calcChain.add(value);
        } else if ("update".equals(message.getStr("op"))) {
            calcChain.remove(calcChain.get(message.getInt(message.getStr("pos"))));
            calcChain.add(value);
        } else if ("del".equals(message.getStr("op"))) {
            calcChain.remove(calcChain.get(message.getInt(message.getStr("pos"))));
        }
        return ws;
    }


    /**
     * 删除行或列
     *
     * @param ws
     * @param message
     * @return
     */
    private WorkSheetEntity drcRefresh(WorkSheetEntity ws, JSONObject message) {
        JSONArray celldata = ObjectUtil.cloneByStream(ws.getData().getJSONArray("celldata"));
        int index = message.getJSONObject("v").getInt("index");
        int len = message.getJSONObject("v").getInt("len");
        if ("r".equals(message.getStr("rc"))) {
            ws.getData().put("row", ws.getData().getInt("row") - len);
        } else {
            ws.getData().put("column", ws.getData().getInt("column") - len);
        }
        for (Object cell : celldata) {
            JSONObject jsonObject = JSONUtil.parseObj(cell);
            if ("r".equals(message.getStr("rc"))) {
                //删除行所在区域的内容
                if (jsonObject.getInt("r") >= index && jsonObject.getInt("r") < index + len) {
                    ws.getData().getJSONArray("celldata").remove(jsonObject);
                }
                //增加大于 最大删除行的的行号
                if (jsonObject.getInt("r") >= index + len) {
                    ws.getData().getJSONArray("celldata").remove(jsonObject);
                    jsonObject.put("r", jsonObject.getInt("r") - len);
                    ws.getData().getJSONArray("celldata").add(jsonObject);
                }
            } else {
                //删除列所在区域的内容
                if (jsonObject.getInt("c") >= index && jsonObject.getInt("c") < index + len) {
                    ws.getData().getJSONArray("celldata").remove(jsonObject);
                }
                //增加大于 最大删除列的的列号
                if (jsonObject.getInt("c") >= index + len) {
                    ws.getData().getJSONArray("celldata").remove(jsonObject);
                    jsonObject.put("c", jsonObject.getInt("c") - len);
                    ws.getData().getJSONArray("celldata").add(jsonObject);
                }
            }
        }
        return ws;
    }


    /**
     * 增加行或列,暂未实现插入数据的情况
     *
     * @param ws
     * @param message
     * @return
     */
    private WorkSheetEntity arcRefresh(WorkSheetEntity ws, JSONObject message) {
        JSONArray celldata = ObjectUtil.cloneByStream(ws.getData().getJSONArray("celldata"));
        int index = message.getJSONObject("v").getInt("index");
        int len = message.getJSONObject("v").getInt("len");

        for (Object cell : celldata) {
            JSONObject jsonObject = JSONUtil.parseObj(cell);
            if ("r".equals(message.getStr("rc"))) {
                //如果是增加行，且是向左增加
                if (jsonObject.getInt("r") >= index && "lefttop".equals(message.getJSONObject("v").getStr("direction"))) {
                    ws.getData().getJSONArray("celldata").remove(jsonObject);
                    jsonObject.put("r", jsonObject.getInt("r") + len);
                    ws.getData().getJSONArray("celldata").add(jsonObject);
                }
                //如果是增加行，且是向右增加
                if (jsonObject.getInt("r") > index && "rightbottom".equals(message.getJSONObject("v").getStr("direction"))) {
                    ws.getData().getJSONArray("celldata").remove(jsonObject);
                    jsonObject.put("r", jsonObject.getInt("r") + len);
                    ws.getData().getJSONArray("celldata").add(jsonObject);
                }


            } else {
                //如果是增加列，且是向上增加
                if (jsonObject.getInt("c") >= index && "lefttop".equals(message.getJSONObject("v").getStr("direction"))) {
                    ws.getData().getJSONArray("celldata").remove(jsonObject);
                    jsonObject.put("c", jsonObject.getInt("c") + len);
                    ws.getData().getJSONArray("celldata").add(jsonObject);
                }
                //如果是增加列，且是向下增加
                if (jsonObject.getInt("c") > index && "rightbottom".equals(message.getJSONObject("v").getStr("direction"))) {
                    ws.getData().getJSONArray("celldata").remove(jsonObject);
                    jsonObject.put("c", jsonObject.getInt("c") + len);
                    ws.getData().getJSONArray("celldata").add(jsonObject);
                }

            }
        }
        JSONArray vArray = message.getJSONObject("v").getJSONArray("data");
        if ("r".equals(message.getStr("rc"))) {
            ws.getData().put("row", ws.getData().getInt("row") + len);
            for (int r = 0; r < vArray.size(); r++) {
                for (int c = 0; c < JSONUtil.parseArray(vArray.get(0)).size(); c++) {
                    if (JSONUtil.parseArray(vArray.get(r)).get(c) == null) {
                        continue;
                    }
                    JSONObject newCell = JSONUtil.createObj().put("r", r + index).put("c", c).put("v", JSONUtil.parseArray(vArray.get(r)).get(c));
                    ws.getData().getJSONArray("celldata").add(newCell);
                }
            }

        } else {
            ws.getData().put("column", ws.getData().getInt("column") + len);
            for (int r = 0; r < vArray.size(); r++) {
                for (int c = 0; c < JSONUtil.parseArray(vArray.get(0)).size(); c++) {
                    if (JSONUtil.parseArray(vArray.get(r)).get(c) == null) {
                        continue;
                    }
                    JSONObject newCell = JSONUtil.createObj().put("r", r).put("c", c + index).put("v", JSONUtil.parseArray(vArray.get(r)).get(c));
                    ws.getData().getJSONArray("celldata").add(newCell);
                }
            }
        }


        return ws;
    }


    /**
     * 筛选操作
     *
     * @param ws
     * @param message
     * @return
     */
    private WorkSheetEntity fscRefresh(WorkSheetEntity ws, JSONObject message) {

        if (message.getJSONObject("v").isEmpty()) {
            ws.getData().remove("filter");
            ws.getData().remove("filter_select");
        } else {
            ws.getData().put("filter", message.getJSONObject("v").getJSONArray("filter"));
            ws.getData().put("filter_select", message.getJSONObject("v").getJSONObject("filter_select"));
        }
        return ws;
    }


    /**
     * 新建sheet
     *
     * @param wbId
     * @param message
     * @return
     */
    private WorkSheetEntity shaRefresh(String wbId, JSONObject message) {
        WorkSheetEntity ws = new WorkSheetEntity();
        ws.setWbId(wbId);
        ws.setData(message.getJSONObject("v"));
        return ws;
    }


    /**
     * 复制sheet
     *
     * @param ws
     * @param message
     * @return
     */
    private WorkSheetEntity shcRefresh(WorkSheetEntity ws, JSONObject message) {

        String index = message.getStr("i");
        ws.getData().put("index", index);
        ws.getData().put("name", message.getJSONObject("v").getStr("name"));

        return ws;
    }

    /**
     * 调整sheet位置
     *
     * @param wbId
     * @param message
     */
    private void shrRefresh(String wbId, JSONObject message) {
        List<WorkSheetEntity> allSheets = workSheetRepository.findAllBywbId(wbId);

        allSheets.forEach(sheet -> {
            sheet.getData().put("order", message.getJSONObject("v").getInt(sheet.getData().getStr("index")));
            workSheetRepository.save(sheet);
        });

    }

    /**
     * 切换到指定sheet
     *
     * @param ws
     * @param message
     * @return
     */
    private void shsRefresh(String wbId, JSONObject message) {
        WorkSheetEntity lastWs = workSheetRepository.findBystatusAndwbId(1, wbId);
        lastWs.getData().put("status", 0);
        WorkSheetEntity thisWs = workSheetRepository.findByindexAndwbId(message.getStr("v"), wbId);
        thisWs.getData().put("status", 1);
        workSheetRepository.save(lastWs);
        workSheetRepository.save(thisWs);
    }


    /**
     * sheet属性(隐藏或显示)
     *
     * @param wbId
     * @param message
     */
    private WorkSheetEntity shRefresh(WorkSheetEntity ws, JSONObject message) {
        Integer hideStatus = message.getInt("v");
        ws.getData().put("hide", hideStatus);

        WorkSheetEntity curWs = new WorkSheetEntity();

        if ("hide".equals(message.getStr("op"))) {
            ws.getData().put("status", 0);
            String cur = message.getStr("cur");
            curWs = workSheetRepository.findByindexAndwbId(cur, ws.getWbId());
            curWs.getData().put("status", 1);

        } else {
            curWs = workSheetRepository.findBystatusAndwbId(1, ws.getWbId());
            curWs.getData().put("status", 0);
        }

        workSheetRepository.save(curWs);
        return ws;
    }

    /**
     * 修改工作簿名称
     *
     * @param wbId
     * @param message
     * @return
     */
    private void naRefresh(String wbId, JSONObject message) {
        Optional<WorkBookEntity> wb = workBookRepository.findById(wbId);
        if (wb.isPresent()) {
            WorkBookEntity workBookEntity = wb.get();
            workBookEntity.getOption().put("title", message.getStr("v"));
            workBookRepository.save(workBookEntity);
        }
    }
}
```

**(9) 创建`common`包，并创建`ResponseDTO.java`文件**

`ResponseDTO.java`文件中内容：

```java
public class ResponseDTO {

    private Integer type;
    private String id;
    private String username;
    private String data;

    public ResponseDTO(Integer type, String id, String username, String data) {
        this.type = type;
        this.id = id;
        this.username = username;
        this.data = data;
    }

    public Integer getType() {
        return type;
    }

    public void setType(Integer type) {
        this.type = type;
    }

    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }


    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getData() {
        return data;
    }

    public void setData(String data) {
        this.data = data;
    }

    public static ResponseDTO success(String id,String username,String data) {
        return new ResponseDTO(1,id,username, data);
    }

    public static ResponseDTO update(String id,String username,String data) {
        return new ResponseDTO(2,id,username, data);
    }

    public static ResponseDTO mv(String id,String username,String data) {
        return new ResponseDTO(3,id,username, data);
    }

    public static ResponseDTO bulkUpdate(String id,String username,String data) {
        return new ResponseDTO(4,id,username ,data);
    }
}
```

**(10) 创建`entity`包，并创建`WorkBookEntity.java`文件**

`WorkBookEntity.java`文件中内容：

```java
import org.springframework.data.annotation.Id;
import org.springframework.data.mongodb.core.mapping.Document;
import cn.hutool.json.JSONObject;

@Document(collection = "workbook")
public class WorkBookEntity {

    @Id
    private String id;

    private String name;

    private JSONObject  option;

    public WorkBookEntity() {

    }

    public WorkBookEntity(String id, String name, JSONObject option) {
        this.id = id;
        this.name = name;
        this.option = option;
    }


    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public JSONObject getOption() {
        return option;
    }

    public void setOption(JSONObject  option) {
        this.option = option;
    }

}
```

**(11) 在`entity`包中创建`WorkSheetEntity.java`文件**

`WorkSheetEntity.java`文件中内容：

```java
import org.springframework.data.annotation.Id;
import org.springframework.data.mongodb.core.mapping.Document;
import cn.hutool.json.JSONObject;

@Document(collection = "worksheet")
public class WorkSheetEntity  {

    @Id
    private String id;
    private String wbId;
    private JSONObject data;

    /**
     * 删除标记,0是未删除，1是删除
     */
    private int deleteStatus;

    public WorkSheetEntity() {

    }

    public WorkSheetEntity(String id, String wbId, JSONObject data, int deleteStatus) {
        this.id = id;
        this.wbId = wbId;
        this.data = data;
        this.deleteStatus = deleteStatus;
    }
    
    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }

    public String getWbId() {
        return wbId;
    }

    public void setWbId(String wbId) {
        this.wbId = wbId;
    }

    public JSONObject getData() {
        return data;
    }

    public void setData(JSONObject data) {
        this.data = data;
    }

    public int getDeleteStatus() {
        return deleteStatus;
    }
    
    public void setDeleteStatus(int deleteStatus) {
        this.deleteStatus = deleteStatus;
    }
}
```

**(12) 创建`repository`包，并创建`WorkBookRepository.java`文件**

`WorkBookRepository.java`文件中内容：

```java
import org.springframework.data.mongodb.repository.MongoRepository;
import org.springframework.stereotype.Repository;
import entity.*;

@Repository
public interface WorkBookRepository extends MongoRepository<WorkBookEntity,String> {

}
```

**(13) 在`repository`包中创建`WorkSheetRepository.java`文件**

`WorkSheetRepository.java`文件中内容：

```java
import org.springframework.data.mongodb.repository.MongoRepository;
import org.springframework.data.mongodb.repository.Query;
import org.springframework.stereotype.Repository;
import entity.*;
import java.util.List;

@Repository
public interface WorkSheetRepository extends MongoRepository<WorkSheetEntity,String> {

    @Query(value = "{'wbId':?0,'deleteStatus':0}")
    List<WorkSheetEntity> findAllBywbId(String wbId);
    @Query(value = "{'data.index':?0,'wbId':?1}")
    WorkSheetEntity findByindexAndwbId(String index,String wbId);

    @Query(value = "{'data.status':?0,'wbId':?1}")
    WorkSheetEntity findBystatusAndwbId(int status,String wbId);
}
```

**(14) 创建`controller`包，并创建`controller.java`文件**

`controller.java`文件中内容：

```java
import java.io.IOException;
import java.util.ArrayList;
import java.util.List;
import java.util.Optional;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.CrossOrigin;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.servlet.ModelAndView;
import com.alibaba.fastjson.JSON;
import cn.hutool.json.JSONObject;
import cn.hutool.json.JSONUtil;
import entity.*;
import repository.*;
import utils.*;

@CrossOrigin
@RestController
public class controller {

    @Autowired
    private WorkBookRepository workBookRepository;
    @Autowired
    private WorkSheetRepository workSheetRepository;

    @GetMapping("index/create")
    public void create(HttpServletRequest request, HttpServletResponse response) throws IOException {
        WorkBookEntity wb = new WorkBookEntity();
        wb.setName("default");
        wb.setOption(SheetUtil.getDefautOption());
        WorkBookEntity saveWb = workBookRepository.save(wb);
        //生成sheet数据
        generateSheet(saveWb.getId());
        response.sendRedirect("/index/" + saveWb.getId());
    }


    @GetMapping("/index/{wbId}")
    public ModelAndView index(@PathVariable(value = "wbId") String wbId) {
        Optional<WorkBookEntity> Owb = workBookRepository.findById(wbId);
        WorkBookEntity wb = new WorkBookEntity();
        if (!Owb.isPresent()) {
            wb.setId(wbId);
            wb.setName("default");
            wb.setOption(SheetUtil.getDefautOption());
            WorkBookEntity result = workBookRepository.save(wb);
            generateSheet(wbId);
        } else {
            wb = Owb.get();
        }
        return new ModelAndView("websocket", "wb", wb);
    }

    @PostMapping("/load/{wbId}")
    public String load(@PathVariable(value = "wbId") String wbId) {
        List<WorkSheetEntity> wsList = workSheetRepository.findAllBywbId(wbId);
        List<JSONObject> list = new ArrayList<JSONObject>();
        wsList.forEach(ws -> {
            list.add(ws.getData());
        });
        return JSONUtil.toJsonStr(list);
    }


    @PostMapping("/loadSheet/{wbId}")
    public String loadSheet(@PathVariable(value = "wbId") String wbId) {
        List<WorkSheetEntity> wsList = workSheetRepository.findAllBywbId(wbId);
        List<JSONObject> list = new ArrayList<>();
        wsList.forEach(ws -> {
            list.add(ws.getData());
        });
        if (!list.isEmpty()) {
            return SheetUtil.buildSheetData(list).toString();
        }
        return SheetUtil.getDefaultAllSheetData().toString();
    }


    private void generateSheet(String wbId) {
        SheetUtil.getDefaultSheetData().forEach(jsonObject -> {
            WorkSheetEntity ws = new WorkSheetEntity();
            ws.setWbId(wbId);
            ws.setData(jsonObject);
            ws.setDeleteStatus(0);
            workSheetRepository.save(ws);
        });
    }
}
```

**(15) 创建`config`包，并创建`WebSocketConfig.java`文件**

`WebSocketConfig.java`文件中内容：

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.socket.server.standard.ServerEndpointExporter;

@Configuration
public class WebSocketConfig {
    @Bean
    public ServerEndpointExporter serverEndpointExporter() {
        return new ServerEndpointExporter();
    }
}
```

**(16) 在`controller`包中创建`WebSocketServer.java`文件**

`WebSocketServer.java`文件中内容：

```java
import java.io.IOException;
import java.util.HashMap;
import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;
import javax.annotation.PostConstruct;
import javax.websocket.EncodeException;
import javax.websocket.OnClose;
import javax.websocket.OnError;
import javax.websocket.OnMessage;
import javax.websocket.OnOpen;
import javax.websocket.Session;
import javax.websocket.server.PathParam;
import javax.websocket.server.ServerEndpoint;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;
import cn.hutool.core.util.StrUtil;
import cn.hutool.json.JSONObject;
import cn.hutool.json.JSONUtil;
import cn.hutool.log.Log;
import cn.hutool.log.LogFactory;
import common.*;
import service.*;
import utils.*;

@ServerEndpoint("/ws/{userId}/{gridKey}")
@Component
public class WebSocketServer {
    static Log log = LogFactory.get(WebSocketServer.class);
    private static WebSocketServer webSocketServer;
    /**
     * 静态变量，用来记录当前在线连接数。应该把它设计成线程安全的。
     */
    private static int onlineCount = 0;
    /**
     * concurrent包的线程安全Set，用来存放每个客户端对应的MyWebSocket对象。
     */
    private static ConcurrentHashMap<String, Map<String, WebSocketServer>> webSocketMap = new ConcurrentHashMap<>();
    /**
     * 与某个客户端的连接会话，需要通过它来给客户端发送数据
     */
    private Session session;
    /**
     * 接收userId
     */
    private String userId = "";
    /**
     * 表格主键
     */
    private String gridKey = "";

    @Autowired
    private IMessageProcess messageProcess;


    @PostConstruct //通过@PostConstruct实现初始化bean之前进行的操作
    public void init() {
        webSocketServer = this;
        webSocketServer.messageProcess = this.messageProcess;
    }

    /**
     * 连接建立成功调用的方法
     */
    @OnOpen
    public void onOpen(Session session, @PathParam("userId") String userId, @PathParam("gridKey") String gridKey) {
        this.session = session;
        this.userId = userId;
        this.gridKey = gridKey;
        if (webSocketMap.containsKey(gridKey)) {
            webSocketMap.get(gridKey).put(userId, this);
        } else {
            Map<String, WebSocketServer> map = new HashMap<>();
            map.put(userId, this);
            webSocketMap.put(gridKey, map);
        }
        addOnlineCount();


        log.info("用户连接:" + userId + ",打开的表格为：" + gridKey + ",当前在线人数为:" + getOnlineCount());


    }

    /**
     * 连接关闭调用的方法
     */
    @OnClose
    public void onClose() {
        if (webSocketMap.containsKey(this.gridKey)) {
            webSocketMap.get(this.gridKey).remove(this.userId);
            if (webSocketMap.get(this.gridKey).isEmpty()) {
                webSocketMap.remove(this.gridKey);
            }
        }
        subOnlineCount();
        log.info("用户退出:" + this.userId + ",打开的表格为：" + this.gridKey + ",当前在线人数为:" + getOnlineCount());
    }

    /**
     * 收到客户端消息后调用的方法
     *
     * @param message 客户端发送过来的消息
     */
    @OnMessage
    public void onMessage(String message, Session session) {
        //可以群发消息
        //消息保存到数据库、redis
        if (StrUtil.isNotBlank(message)) {
            try {
                if ("rub".equals(message)) {
                    return;
                }
                String unMessage = PakoGzipUtils.unCompressURI(message);
                log.info("用户消息:" + userId + ",报文:" + unMessage);
                JSONObject jsonObject = JSONUtil.parseObj(unMessage);
                if (!"mv".equals(jsonObject.getStr("t"))) {
                    webSocketServer.messageProcess.process(this.gridKey, jsonObject);
                }

                Map<String, WebSocketServer> sessionMap = webSocketMap.get(this.gridKey);
                if (StrUtil.isNotBlank(unMessage)) {
                    sessionMap.forEach((key, value) -> {

                        //广播到除了发送者外的其它连接端
                        if (!key.equals(this.userId)) {
                            try {
                                //如果是mv,代表发送者的表格位置信息
                                if ("mv".equals(jsonObject.getStr("t"))) {
                                    value.sendMessage(JSONUtil.toJsonStr(ResponseDTO.mv(userId, userId, unMessage)));
                                    //如果是切换sheet，则不发送信息
                                } else if(!"shs".equals(jsonObject.getStr("t"))) {
                                    value.sendMessage(JSONUtil.toJsonStr(ResponseDTO.update(userId, userId, unMessage)));
                                }
                            } catch (Exception e) {
                                e.printStackTrace();
                            }
                        }
                    });
                }
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }

    /**
     * @param session
     * @param error
     */
    @OnError
    public void onError(Session session, Throwable error) {
        log.error("用户错误:" + this.userId + ",原因:" + error.getMessage());
        error.printStackTrace();
    }

    /**
     * 实现服务器主动推送
     */
    public void sendMessage(String message) throws IOException, EncodeException {
        this.session.getAsyncRemote().sendText(message);
    }

    public static synchronized int getOnlineCount() {
        return onlineCount;
    }

    public static synchronized void addOnlineCount() {
        WebSocketServer.onlineCount++;
    }

    public static synchronized void subOnlineCount() {
        WebSocketServer.onlineCount--;
    }
}
```

**(17) 主启动目录中内容**

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.data.mongodb.repository.config.EnableMongoRepositories;

@EnableAutoConfiguration
@ComponentScan("config,service,controller")
@EnableMongoRepositories(basePackages = {"repository"})
public class Start {

    public static void main(String[] args) {
        SpringApplication.run(Start.class, args);
    }

}
```

***

##### <font size="4" color="red">52. Springboot集成Mqtt服务</font>

**(1) 依赖包**

```xml
<!-- springboot 父节点区域 -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.5.6.RELEASE</version>
</parent>
<dependencies>
     <!-- springboot依赖包 -->
  	<dependency>
  		<groupId>org.springframework.boot</groupId>
  		<artifactId>spring-boot-starter-web</artifactId>
  	</dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-integration</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.integration</groupId>
        <artifactId>spring-integration-stream</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.integration</groupId>
        <artifactId>spring-integration-mqtt</artifactId>
    </dependency>
</dependencies>
```

**(2) 主启动目录中创建资源文件夹`resources`文件夹，并创建`application.properties`文件**

`application.properties`文件中内容：

```ini
server.port = 8310
spring.mqtt.username=admin
spring.mqtt.password=public
## 推送信息的连接地址，如果有多个，用逗号隔开
spring.mqtt.url=tcp://127.0.0.1:1883
spring.mqtt.client.id=${random.value}
spring.mqtt.default.topic=topic
```

**(3) 创建Configure包，并创建customerConfigure.java文件**

customerConfigure.java文件中内容：

```java
mport org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.integration.annotation.ServiceActivator;
import org.springframework.integration.channel.DirectChannel;
import org.springframework.integration.core.MessageProducer;
import org.springframework.integration.mqtt.core.DefaultMqttPahoClientFactory;
import org.springframework.integration.mqtt.core.MqttPahoClientFactory;
import org.springframework.integration.mqtt.inbound.MqttPahoMessageDrivenChannelAdapter;
import org.springframework.integration.mqtt.support.DefaultPahoMessageConverter;
import org.springframework.messaging.Message;
import org.springframework.messaging.MessageChannel;
import org.springframework.messaging.MessageHandler;
import org.springframework.messaging.MessagingException;

@Configuration
public class MqttConfigure {

    @Value("${spring.mqtt.username}")
    private String username;

    @Value("${spring.mqtt.password}")
    private String password;

    @Value("${spring.mqtt.url}")
    private String hostUrl;

    @Value("${spring.mqtt.client.id}")
    private String clientId;

    @Value("${spring.mqtt.default.topic}")
    private String defaultTopic;

    @Bean
    public MqttPahoClientFactory mqttClientFactory() {
        DefaultMqttPahoClientFactory factory = new DefaultMqttPahoClientFactory();
        factory.setUserName(username);
        factory.setPassword(password);
        factory.setServerURIs(new String[]{hostUrl});
        factory.setKeepAliveInterval(2);
        return factory;
    }

    //接收通道
    @Bean
    public MessageChannel mqttInputChannel() {
        return new DirectChannel();
    }

    //配置client,监听的topic 
    @Bean
    public MessageProducer inbound() {
        MqttPahoMessageDrivenChannelAdapter adapter =
            new MqttPahoMessageDrivenChannelAdapter(clientId, mqttClientFactory(),defaultTopic);
        adapter.setConverter(new DefaultPahoMessageConverter());
        adapter.setQos(1);
        adapter.setOutputChannel(mqttInputChannel());
        return adapter;
    }

    //通过通道获取数据
    @Bean
    @ServiceActivator(inputChannel = "mqttInputChannel")
    public MessageHandler handler() {
        return new MessageHandler() {
            @Override
            public void handleMessage(Message<?> message) throws MessagingException {
                System.out.println("接收消息内容 : " + message.getPayload());

            }
        };
    }
}
```

**(4) 在`Configure`包中创建`providerConfigure.java`文件**

`providerConfigure.java`文件中内容：

```java
import org.eclipse.paho.client.mqttv3.MqttClient;
import org.eclipse.paho.client.mqttv3.MqttConnectOptions;
import org.eclipse.paho.client.mqttv3.MqttDeliveryToken;
import org.eclipse.paho.client.mqttv3.MqttException;
import org.eclipse.paho.client.mqttv3.MqttMessage;
import org.eclipse.paho.client.mqttv3.MqttPersistenceException;
import org.eclipse.paho.client.mqttv3.MqttTopic;
import org.eclipse.paho.client.mqttv3.persist.MemoryPersistence;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class providerConfigure {
    @Value("${spring.mqtt.username}")
    private String username;

    @Value("${spring.mqtt.password}")
    private String password;

    @Value("${spring.mqtt.url}")
    private String hostUrl;

    @Value("${spring.mqtt.client.id}")
    private String clientId;

    @Value("${spring.mqtt.default.topic}")
    private String defaultTopic;

    private static MqttClient client;

    public static MqttClient getClient() {
        return client;
    }

    public static void setClient(MqttClient client) {
        ProviderService.client = client;
    }

    @Bean
    public MqttConnectOptions getMqttConnectOptions(){
        MqttConnectOptions mqttConnectOptions=new MqttConnectOptions();
        mqttConnectOptions.setUserName(username);
        mqttConnectOptions.setPassword(password.toCharArray());
        mqttConnectOptions.setCleanSession(false);
        mqttConnectOptions.setKeepAliveInterval(2);
        return mqttConnectOptions;
    }

    @Bean
    public MqttClient getMqttPushClient() throws MqttException{
        client = new MqttClient(hostUrl, clientId,new MemoryPersistence());
        client.connect(getMqttConnectOptions());
        client.setCallback(new ProviderCallback());
        return client;
    }

    public void publish(String topic,String pushMessage){
        publish(0, false, topic, pushMessage);
    }

    public void publish(int qos,boolean retained,String topic,String pushMessage){
        MqttMessage message = new MqttMessage();
        message.setQos(qos);
        message.setRetained(retained);
        message.setPayload(pushMessage.getBytes());
        MqttTopic mTopic = ProviderService.getClient().getTopic(topic);
        MqttDeliveryToken token;
        try {
            token = mTopic.publish(message);
            token.waitForCompletion();
        } catch (MqttPersistenceException e) {
            e.printStackTrace();
        } catch (MqttException e) {
            e.printStackTrace();
        }
    }

    public void subscribe(String topic){
        subscribe(topic,0);
    }

    /**
     * 订阅某个主题
     * @param topic
     * @param qos
     */
    public void subscribe(String topic,int qos){
        try {
            ProviderService.getClient().subscribe(topic, qos);
        } catch (MqttException e) {
            e.printStackTrace();
        }
    }
}
```

**(5) 在`Configure`包中创建`providerCallback.java`文件**

`providerCallback.java`文件中内容：

```java
import org.eclipse.paho.client.mqttv3.IMqttDeliveryToken;
import org.eclipse.paho.client.mqttv3.MqttCallback;
import org.eclipse.paho.client.mqttv3.MqttMessage;

public class ProviderCallback implements MqttCallback {

	@Override
	public void connectionLost(Throwable cause) {

	}

	@Override
	public void messageArrived(String topic, MqttMessage message) throws Exception {
		// TODO Auto-generated method stub
		// TODO Auto-generated method stub
		System.out.println("接收消息主题 : " + topic);
        System.out.println("接收消息Qos : " + message.getQos());
        System.out.println("接收消息内容 : " + new String(message.getPayload()));
	}

	@Override
	public void deliveryComplete(IMqttDeliveryToken token) {

	}

}
```

**(6) 创建`controller`包，并创建`controller.java`文件**

`controller.java`文件中内容：

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class Controller {

    @Autowired
    private ProviderService mqttProvider;

    @RequestMapping(value="/index",method = {RequestMethod.GET,RequestMethod.POST})
    public @ResponseBody String Index() {
        mqttProvider.publish("topic", "字符串");
        return "success";

    }
}
```

**(7) 主启动目录中内容**

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.context.annotation.ComponentScan;

@EnableAutoConfiguration
@ComponentScan("Configure,controller")
public class Start {

	public static void main(String[] args) {
		SpringApplication.run(Start.class, args);
	}
}
```

***













