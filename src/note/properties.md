[toc]

---

# Spring 加载propertie文件方式

> 当应用比较大的时候，如果所有的内容都放在一个文件，会显得过分臃肿且不好理解，可以将一个文件拆分多个文件，然后在分别加载进系统。

##  @PropertySource
- 加载指定的属性文件(*.properties)到Spring的Environment；
- 和@Value组合使用，可以将properties中的变量注入到当前类使用。
- 和@ConfigurationProperties，可以将properties中的变量注入到当前类使用。

### 固定编码

```java
@PropertySource(value = "classpath:/config/remote-ssh.properties", encoding = "utf-8", ignoreResourceNotFound = true)
@ConfigurationProperties(prefix = "ssh")
public class SshProperties {
    //    @Value("${ssh.enabled:false}")
    private boolean enabled;
}
```

### profile切换

```yaml
# application-dev.properties
smilex.path="classpath:"

# application-prod.properties
smilex.path="/opt/user/smilex"
```

我们通过变量的形式注入到@PropertySource即可。

```java
@PropertySource(value = "${smilex.path}/config/remote-ssh.properties", encoding = "utf-8", ignoreResourceNotFound = true)
@ConfigurationProperties(prefix = "ssh")
public class SshProperties {
    //    @Value("${ssh.enabled:false}")
    private boolean enabled;
}
```



## 在SpringApplication启动时，注入环境

```java

import org.springframework.beans.factory.config.PropertiesFactoryBean;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.builder.SpringApplicationBuilder;
import org.springframework.context.annotation.PropertySource;
import org.springframework.core.env.PropertiesPropertySource;
import org.springframework.core.env.StandardEnvironment;
import org.springframework.core.io.Resource;
import org.springframework.core.io.support.PathMatchingResourcePatternResolver;

import java.io.IOException;
import java.util.Properties;

@SpringBootApplication
public class AdminApiApplication {
    public static void main(String[] args) throws IOException {
        SpringApplicationBuilder springApplicationBuilder = new SpringApplicationBuilder(AdminApiApplication.class);
        Properties properties = getProperties();
        StandardEnvironment environment = new StandardEnvironment();
        environment.getPropertySources().addLast(new PropertiesPropertySource("service-properties", properties));
        springApplicationBuilder.environment(environment);
        springApplicationBuilder.run(args);
    }

    private static Properties getProperties() throws IOException {
        PropertiesFactoryBean propertiesFactoryBean = new PropertiesFactoryBean();
        PathMatchingResourcePatternResolver resolver = new PathMatchingResourcePatternResolver();
        Resource[] resource = resolver.getResources("classpath:*.properties");
        propertiesFactoryBean.setLocations(resource);
        propertiesFactoryBean.afterPropertiesSet();
        return propertiesFactoryBean.getObject();
    }
}

```

