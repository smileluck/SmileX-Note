[toc]

---

# druid 
## Maven引入

```xml
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid-spring-boot-starter</artifactId>
    <version>1.2.8</version>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
    <exclusions>
        <!-- 移除HikariCp数据源 -->
        <exclusion>
            <groupId>com.zaxxer</groupId>
            <artifactId>HikariCP</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```



## 多数据源

### 配置

1. 设置Datasource properties

```java
@Data
public class DataSourceProperties {

    private String driverClassName;
    private String url;
    private String username;
    private String password;

    /**
     * Druid默认参数
     */
    private int initialSize = 5;
    private int maxActive = 20;
    private int minIdle = 5;
    private long maxWait = 60 * 1000L;
    private long timeBetweenEvictionRunsMillis = 60 * 1000L;
    private long minEvictableIdleTimeMillis = 1000L * 60L * 30L;
    private long maxEvictableIdleTimeMillis = 1000L * 60L * 60L * 7;
    private String validationQuery = "select 1 from DUAL";
    private int validationQueryTimeout = -1;
    private boolean testOnBorrow = false;
    private boolean testOnReturn = false;
    private boolean testWhileIdle = true;
    private boolean poolPreparedStatements = true;
    private int maxOpenPreparedStatements = -1;
    private boolean sharePreparedStatements = false;
    private String filters = "stat,wall";
}
```

2. yml配置

```yaml
spring:
  datasource:
    druid:
      # mysql 连接信息
      driver-class-name: com.mysql.cj.jdbc.Driver
      url: jdbc:mysql://127.0.0.1:3306/smilex-boot?useUnicode=true&characterEncoding=UTF-8&useSSL=false&serverTimezone=Asia/Shanghai
      username: root
      password: root
      # 其它配置
      time-between-eviction-runs-millis: 60000
      keep-alive: true
      initial-size: 5
      min-idle: 5
      max-active: 20
      max-wait: 60000
      min-evictable-idle-time-millis: 300000
      validation-query: SELECT 1 FROM DUAL
      test-while-idle: true
      test-on-borrow: false
      test-on-return: false
      pool-prepared-statements: true
      # 配置监控统计拦截的filters，去掉后监控界面sql无法统计，'wall'用于防火墙
      filters: stat,wall
      stat-view-servlet:
        enabled: true
        allow: 127.0.0.1
        url-pattern: /druid/*
```

3. 实现AbstractRoutingDataSource

```java

/**
 * 实现多数数据源控制
 */
public class DynamicDataSource extends AbstractRoutingDataSource {

    private static DynamicDataSource INSTANCE;

    private static Map<Object, Object> dataSourceMap = new HashMap<>();

    private static final ReentrantLock lock = new ReentrantLock();

    public static DynamicDataSource getInstance() {
        if (INSTANCE == null) {
            synchronized (DynamicDataSource.class) {
                if (INSTANCE == null) {
                    INSTANCE = new DynamicDataSource();
                }
            }
        }
        return INSTANCE;
    }

    public void addDataSource(Object key, DataSourceProperties dataSourceProperties) {
        DataSource dataSource = DataSourceFactory.createDataSource(dataSourceProperties);
        addDataSource(key, dataSource);
    }

    public void addDataSource(Object key, DataSource dataSource) {
        Object o = dataSourceMap.get(key);
        if (o != null) {
//            if (o instanceof DataSource) {
//                DruidDataSource druidDataSource = (DruidDataSource) o;
//                druidDataSource.close();
//            }
            throw new SXException("数据库连接池 KEY 重复");
        }
        dataSourceMap.put(key, dataSource);
        setDataSourceMap(dataSourceMap);
    }

    public void setDataSourceMap(Map<Object, Object> objectObjectMap) {
        lock.lock();
        this.dataSourceMap = objectObjectMap;
        super.setTargetDataSources(dataSourceMap);
        super.afterPropertiesSet();
        lock.unlock();
    }

    public void delDataSource(Object key) {
        Object o = dataSourceMap.get(key);
        if (o != null) {
            if (o instanceof DataSource) {
                DruidDataSource dataSource = (DruidDataSource) o;
                if (dataSource != null) {
                    dataSource.close();
                    dataSource = null;
                    dataSourceMap.remove(key);
                    setDataSourceMap(dataSourceMap);
                }
            } else {
                throw new SXException("数据池类型无法识别");
            }
        }
    }

    /**
     * Recording to the datasource of the map with the key
     * @param key
     * @param dataSource
     */
    public void replaceDataSource(Object key, DataSource dataSource) {
        dataSourceMap.put(key, dataSource);
        setDataSourceMap(dataSourceMap);
    }

    @Override
    protected Object determineCurrentLookupKey() {
        return DataSourceContentHolder.getDataSource();
    }
}
```

4. 配置Spring DynamicDatasouce Bean

```java

@Configuration
@Order(-1)
public class DynamicDataSourceConfig {


    @Bean
    @ConfigurationProperties(prefix = "spring.datasource.druid")
    public DataSourceProperties dataSourceProperties() {
        return new DataSourceProperties();
    }

    @Primary
    @Bean(name = "dynamicDataSource")
    public DynamicDataSource dynamicDataSource(DataSourceProperties dataSourceProperties) {
        DynamicDataSource dynamicDataSource = DynamicDataSource.getInstance();

        DataSource dataSource = DataSourceFactory.createDataSource(dataSourceProperties);
        dynamicDataSource.addDataSource("master", dataSource);
        dynamicDataSource.setDefaultTargetDataSource(dataSource);
        return dynamicDataSource;
    }

    @Bean(name = "transactionManager")
    public DataSourceTransactionManager transactionManager(@Qualifier("dynamicDataSource") DynamicDataSource dynamicDataSource) {
        return new DataSourceTransactionManager(dynamicDataSource);
    }
}
```

5. 设置DataSourceFactory 工厂，用于根据配置生成DataSource

```java
public class DataSourceFactory {
    public static DataSource createDataSource(DataSourceProperties properties) {
        DruidDataSource dataSource = new DruidDataSource();
        BeanUtils.copyProperties(properties, dataSource);
        return dataSource;
    }

    public static DataSource createDataSource(DataSource dataSource, DataSourceProperties properties) {
        DruidDataSource druidDataSource = (DruidDataSource) dataSource;
        BeanUtils.copyProperties(properties, druidDataSource);
        return druidDataSource;
    }
}
```

### 使用方式

#### 注解

1. 添加注解 DataSource

```java
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
public @interface DataSource {
    String value() default DynamicDataSourceConfig.MASTER;
}
```

2. 添加拦截器 DataSourceAspect

```java
@Aspect
@Component
@Slf4j
public class DataSourceAspect {
    @Pointcut("@within(top.zsmile.common.datasource.annotation.DS) || @annotation(top.zsmile.common.datasource.annotation.DS)")
    public void dataSourceAspect() {

    }

    @Before("dataSourceAspect()")
    public void beforeSwitch(JoinPoint joinPoint) {
        MethodSignature signature = (MethodSignature) joinPoint.getSignature();
        Method method = signature.getMethod();
        DataSource methodDS = method.getAnnotation(DataSource.class);
        String value = null;
        if (methodDS != null) {
            value = methodDS.value();
        } else {
            Class<?> aClass = joinPoint.getTarget().getClass();
            DataSource annotation = aClass.getAnnotation(DataSource.class);
            if (annotation != null) {
                value = annotation.value();

            }
        }
        if (checkValue(value)) {
            DataSourceContentHolder.setDataSource(value);
        }
    }

    private boolean checkValue(final String value) {
        if (!StringUtils.isEmpty(value)) {
            boolean containKey = DynamicDataSource.getInstance().containKey(value);
            if (!containKey) {
                throw new SXException("数据源" + value + "未准备");
            }
        }
        return true;
    }

    @After("dataSourceAspect()")
    public void afterSwitchDS() {
        DataSourceContentHolder.clear();
    }
}
```

3. 在需要切换数据源的地方添加注解

## 多次切换数据源

前面我们使用的是ThreadLocal<String> 来处理数据源的切换问题，那么相应的也有了新的问题，如果我们有以下伪代码，那这时会有什么结果。

```java
@DataSource("test1")
public void A(){
	AMapper.queryById(1);
    B();
	AMapper.queryById(3);
}

@DataSource("test2")
public void B(){
    BMapper.queryById(2)
}
```

这里我们可以预计一个样子。

1. 当 AMapper.queryById(1) 时，指向数据源 test1
2. 当执行 BMapper.queryById(2) 时，指向 test2
3. 再执行 AMapper.queryById(1) 时，指向数据源 test1。

可是问题来了，我们用ThreadLocal<String>时，他就只支持了一个变量来存储，这意味着，当一个线程存在多次切换环境时无法支持。

我们应该如何来解决这个问题。

我们可以通过替换ThreadLocal<String> 为 ThreadLocal<Queue<String>>来实现。修改DataSourceContentHolder 文件如下

```java

@Slf4j
public class DataSourceContentHolder {
    private static final ThreadLocal<Queue<String>> contentHolder = new ThreadLocal() {
        @Override
        protected Object initialValue() {
            return new LinkedList<>();
        }
    };

    public synchronized static void setDataSource(String dataSource) {
        contentHolder.get().add(dataSource);
    }

    public static String getDataSource() {
        String ds = contentHolder.get().peek();
        log.debug("当前数据源 => {}", ds);
        return ds;
    }

    public static void pollDataSource() {
        Queue<String> queue = contentHolder.get();
        String ds = queue.poll();
        log.debug("移除数据源 => {}", ds);
        if (queue.isEmpty()) {
            contentHolder.remove();
        }
    }
}
```

