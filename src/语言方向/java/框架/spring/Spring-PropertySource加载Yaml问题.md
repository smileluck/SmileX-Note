[TOC]

---

# 前言

关于在`SpringBoot`中使用 `@PropertySource`注解读取`YAML`属性文件存在属性`null`的问题。

因为在 `SpringBoot2` 中默认是使用 `properties` 文件解析，不支持 `Yaml`文件的解析。

# 自定义加载YAML文件

1. 自定义 `YamlPropertySourceFactory`，实现 `propertySourceFactory` 接口
   
   ```java
   /**
    * Yaml 解析工厂
    */
   public class YamlPropertySourceFactory implements PropertySourceFactory {
       @Override
       public PropertySource<?> createPropertySource(String name, EncodedResource encodedResource) throws IOException {
           Properties properties = loadYamlProperties(encodedResource);
           String sourceName = name != null ? name : encodedResource.getResource().getFilename();
           return new PropertiesPropertySource(sourceName, properties);
       }
   
       /**
        * 从resource里面读取属性清单
        *
        * @param resource 
        * @return 属性清单
        * @throws FileNotFoundException
        */
       private Properties loadYamlProperties(EncodedResource resource) throws FileNotFoundException {
           try {
               YamlPropertiesFactoryBean factory = new YamlPropertiesFactoryBean();
               factory.setResources(resource.getResource());
               factory.afterPropertiesSet();
               return factory.getObject();
           } catch (IllegalStateException e) {
               // for ignoreResourceNotFound
               Throwable cause = e.getCause();
               if (cause instanceof FileNotFoundException)
                   throw (FileNotFoundException) e.getCause();
               throw e;
           }
       }
   
   }
   ```

```
2. 在 `@PropertySource` 使用。

```yaml
# tencent.yml文件
tencent:
  secret_id: 
  secret_key: 
```

   在 `@PropertySource` 中设置 `factory`(该属性 从 Spring 4.3 增加) 指向 `YamlPropertySourceFactory.class`

```java
@Component
@ConfigurationProperties(prefix = "tencent")
@PropertySource(value = {"classpath:tencent.yml"}, encoding = "UTF-8", factory = YamlPropertySourceFactory.class)
public class TencentConfig {

    private String secretId;

    private String secretKey;

    private TencentSms sms;

    public String getSecretId() {
        return secretId;
    }

    public void setSecretId(String secretId) {
        this.secretId = secretId;
    }

    public String getSecretKey() {
        return secretKey;
    }

    public void setSecretKey(String secretKey) {
        this.secretKey = secretKey;
    }
}
```