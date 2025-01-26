# SpringCache

## 配置

### pom

```xml
<dependencies>
    <!-- Spring Boot Redis Starter -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-redis</artifactId>
    </dependency>
    <!-- Redisson Spring Boot Starter -->
    <dependency>
        <groupId>org.redisson</groupId>
        <artifactId>redisson-spring-boot-starter</artifactId>
        <version>3.16.6</version> <!-- 使用最新版本 -->
    </dependency>
</dependencies>
```

### 属性

> application.properties或application.yml

```
# application.properties
spring.redis.host=localhost
spring.redis.port=6379
# 可选配置，如需要密码认证等
# spring.redis.password=yourpassword
```

### Config

> CacheConfig

```java
import org.springframework.cache.annotation.EnableCaching;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.cache.RedisCacheConfiguration;
import org.springframework.data.redis.cache.RedisCacheManager;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.serializer.GenericJackson2JsonRedisSerializer;
import org.springframework.data.redis.serializer.RedisSerializationContext;
import java.time.Duration;
 
@Configuration
@EnableCaching
public class CacheConfig {
    @Bean
    public RedisCacheManager cacheManager(RedisConnectionFactory connectionFactory) {
        RedisCacheConfiguration config = RedisCacheConfiguration.defaultCacheConfig()
            .entryTtl(Duration.ofHours(1)) // 设置缓存有效期为1小时
            .serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(new GenericJackson2JsonRedisSerializer())); // 使用JSON序列化值对象
        return RedisCacheManager.builder(connectionFactory)
            .cacheDefaults(config)
            .build();
    }
}
```

## 注解

**Spring内置的三大注解缓存是：**

1. `Cacheable`：缓存
2. `CacheEvict`：删除缓存
3. `CachePut`：更新缓存

## 1）@Cacheable（常用）

在 @Cacheable 注解的使用中，共有 9 个属性供我们来使用，这 9 个属性分别是： value、 cacheNames、 key、 keyGenerator、 cacheManager、 cacheResolver、 condition、 unless、 sync。接下来我们就分别来介绍一下它的使用。

### 1.value/cacheNames 属性

如下图所示，这两个属性代表的意义相同，根据@AliasFor注解就能看出来了。这两个属性都是用来指定缓存组件的名称，即将方法的返回结果放在哪个缓存中，属性定义为数组，可以指定多个缓存；  
![image.png](https://ucc.alicdn.com/pic/developer-ecology/urngpoxx4ky3i_c52408a93d47431b819a3aff434314b5.png?x-oss-process=image/resize,w_1400/format,webp)

```java
//这两种配置等价
@Cacheable(value = "user") //@Cacheable(cacheNames = "user")
User getUser(Integer id);
```

### 2.key属性

可以通过 key 属性来指定缓存数据所使用的的 key，默认使用的是方法调用传过来的参数作为 key。最终缓存中存储的内容格式为：Entry 形式。

1. 如果请求没有参数：key=new SimpleKey()；
2. 如果请求有一个参数：key=参数的值
3. 如果请求有多个参数：key=newSimpleKey(params)； (你只要知道 key不会为空就行了)

key值的编写，可以使用 SpEL 表达式的方式来编写；除此之外，我们同样可以使用 keyGenerator  
生成器的方式来指定 key，我们只需要编写一个 keyGenerator ，将该生成器注册到 IOC 容器即可。(keyGenerator的使用，继续往下看)

![image.png](https://ucc.alicdn.com/pic/developer-ecology/urngpoxx4ky3i_907e201020294b1e888caa0451ba8a59.png?x-oss-process=image/resize,w_1400/format,webp)

使用示例如下：

```java
@Cacheable(value = "user",key = "#root.method.name")
User getUser(Integer id);
```

### 3.keyGenerator 属性

key 的生成器。如果觉得通过参数的方式来指定比较麻烦，我们可以自己指定 key 的生成器的组件 id。key/keyGenerator属性：二选一使用。我们可以通过自定义配置类方式，将 keyGenerator 注册到 IOC 容器来使用。

```java
import org.springframework.cache.interceptor.KeyGenerator;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import java.lang.reflect.Method;
import java.util.Arrays;

@Configuration
public class MyCacheConfig {


    @Bean("myKeyGenerator")
    public KeyGenerator keyGenerator(){

        return new KeyGenerator(){


            @Override
            public Object generate(Object target, Method method, Object... params) {

                return method.getName()+ Arrays.asList(params).toString();
            }
        };
    }

    /**     * 支持 lambda 表达式编写     */
    /*@Bean("myKeyGenerator")    public KeyGenerator keyGenerator(){        return ( target,  method, params)-> method.getName()+ Arrays.asList(params).toString();    }*/
}
```

### 4.cacheManager 属性

该属性，用来指定缓存管理器。针对不同的缓存技术，需要实现不同的 cacheManager，Spring 也为我们定义了如下的一些 cacheManger 实现（）  
![image.png](https://ucc.alicdn.com/pic/developer-ecology/urngpoxx4ky3i_33184ccaf7b746a1a0e64495b3b60914.png?x-oss-process=image/resize,w_1400/format,webp)

### 5.cacheResolver 属性

该属性，用来指定缓存解析器。使用配置同 cacheManager 类似，可自行百度。（cacheManager指定管理器/cacheResolver指定解析器 它俩也是二选一使用）

### 6.condition 属性

条件判断属性，用来指定符合指定的条件下才可以缓存。也可以通过 SpEL 表达式进行设置。这个配置规则和上面表格中的配置规则是相同的。

```java
@Cacheable(value = "user",condition = "#id>0")//传入的 id 参数值>0才进行缓存
User getUser(Integer id);
```

```java
@Cacheable(value = "user",condition = "#a0>1")//传入的第一个参数的值>1的时候才进行缓存
User getUser(Integer id);
```

```java
@Cacheable(value = "user",condition = "#a0>1 and #root.methodName eq 'getUser'")//传入的第一个参数的值>1 且 方法名为 getUser 的时候才进行缓存
User getUser(Integer id);
```

### 7.unless 属性

unless属性，意为"除非"的意思。即只有 unless 指定的条件为 true 时，方法的返回值才不会被缓存。可以在获取到结果后进行判断。

```java
@Cacheable(value = "user",unless = "#result == null")//当方法返回值为 null 时，就不缓存
User getUser(Integer id);
```

```java
@Cacheable(value = "user",unless = "#a0 == 1")//如果第一个参数的值是1,结果不缓存
User getUser(Integer id);
```

### 8.sync 属性

该属性用来指定是否使用异步模式，该属性默认值为 false，默认为同步模式。异步模式指定 sync = true 即可，异步模式下 unless 属性不可用。

## 2）@CachePut（常用）

> @CachePut 也可以声明一个方法支持缓存功能。  
> 与 @Cacheable 不同的是使用 @CachePut标注的方法在执行前不会去检查缓存中是否存在之前执行过的结果，而是每次都会执行该方法，并将执行结果以键值对的形式存入指定的缓存中。  
> ​使用@CachePut 可以指定的属性跟 @Cacheable 是一样的， @CachePut 适用于缓存更新。

## 3）@CacheEvict（常用）

> @CacheEvict 是用来标注在需要清除缓存元素的方法或类上的。当标记在一个类上时表示其中所有的方法的执行都会触发缓存的清除操作。  
> @CacheEvict支持的属性额外增加了两个：  
> ​ 1、allEntries：是否需要清除缓存中的所有元素。默认为 false ，表示不需要。当指定了 allEntries 为 true 时，Spring Cache将忽略指定的key，删除缓存中所有键；  
> ​ 2、beforeInvocation： 是否在方法执行成功之后触发键删除操作，默认是在对应方法成功执行之后触发的，若此时方法抛出异常而未能成功返回，不会触发清除操作。指定该属性值为 true 时，Spring会在调用该方法之前清除缓存中的指定元素。

## 4）@Caching（不常用）

> @Caching 注解可以在一个方法或者类上同时指定多个Spring Cache相关的注解。  
> 其拥有三个属性：cacheable、put 和 evict，分别用于指定@Cacheable、@CachePut 和 @CacheEvict。对于一个数据变动，更新多个缓存的场景，可以通过 @Caching 来实现：

```java
@Caching(cacheable = @Cacheable(cacheNames = "caching", key = "#age"), evict = @CacheEvict(cacheNames = "t4", key = "#age"))
public String caching(int age) {

    return "caching: " + age + "-->" + UUID.randomUUID().toString();
}
```

上面这个就是组合操作：从 caching::#age 缓存取数据，不存在时执行方法并写入缓存；删除缓存 t4::#age。

## 5）@CacheConfig（不常用）

> 如果一个类中，多个方法都有同样的 cacheName，keyGenerator，cacheManager 和 cacheResolver，可以直接使用 @CacheConfig 注解在类上声明，这个类中的方法都会使用@CacheConfig 属性设置的相关配置。

```java
@Component
@CacheConfig(cacheNames = "mall_cache")
public class CacheComponent {



    @Cacheable(key = "'perm-whitelist-'+#clientId", unless="#result == null")
    public List<String> cacheWriteList(String clientId){

        ...
    }

     @Cacheable(key = "'perm-cutom-aci-' + #tenantId + '-' + #roleId + '-' + #tenantLevel + '-' + #subType", unless="#result == null")
    public List<RequestDto> cacheRequest(Long tenantId,Long roleId,Integer tenantLevel,Integer subType){

        ...
    }
}
```
