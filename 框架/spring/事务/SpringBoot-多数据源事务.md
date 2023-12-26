[toc]

---

# 前言

我们使用 SpringBoot + Druid 完成了单机多数据源的切换，但是这时会出现一个问题，添加事务了以后，数据源切换出现了问题，那么应该怎么解决呢？（实际是解决**数据一致性**的问题）

抛出异常回滚时，子事务已经提交，无法回滚，会产生数据不一致的问题。

# 单机

实现事务内切换数据源(支持原生Spring声明式事务哟，仅此一家)，并支持多数据源事务回滚(有了 它除了跨服务的事务你需要考虑分布式事务，其他都不需要，极大的减少了系统的复杂程度)  

这里采用的方案是：

1. 扩展 `Spring Jdbc` 提供的抽象类 `AbstractRoutingDataSource`，实现切换数据源
2. 数据源配置主要有两种
   - 基于配置文件（该文章基于这种实现，但支持动态添加）
   - 基于数据库表
3. 基于 `AspectJ` 实现动态数据源切换，支持方法级、类级，优先方法级别
4. 通过实现 `ibatis.Transaction` 和 `TransactionFactory` 支持原生 `Spring` 的 `@Transaction` 的多数据源事务回滚

## 环境准备

```xml

<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.3.12.RELEASE</version>
</parent>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>

<druid.version>1.2.8</druid.version>
<mybatis-spring-boot.version>2.2.2</mybatis-spring-boot.version>
```



## 数据源切换

### 思路

扩展 `Spring Jdbc` 提供的抽象类 `AbstractRoutingDataSource`，实现切换数据源

 ![img](SpringBoot-多数据源事务.assets\71156f3d289cdde2e6a1cbb5e7df7859.png) 

- targetDataSources是目标数据源集合
- defaultTargetDataSource是默认数据源
- resolvedDataSources是解析后的数据源集合
- resolvedDefaultDataSource是解析后的默认数据源
-  determineCurrentLookupKey 为抽象方法，通过扩展这个方法来实现数据源的切换，key是数据源的名称。 `lookup key`通常是绑定在线程上下文中，根据这个`key`去`resolvedDataSources`中取出`DataSource` 

通过注解 + AOP + ThreadLocal 拦截方法后，添加数据源KEY到中，然后查询数据库时会获取到这个key，进而获取对应的数据源进行操作。

### 实现代码

1. 首先我们需要使用一个注解作为AOP切面的拦截标识

```java
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
public @interface DS {
    String value() default DynamicDataSourceProperties.PRIMARY;
}
```

2. 基于线程变量 `ThreadLocal`提供一个数据源切换的工具，采用 双端 队列存储，主要是为了支持嵌套数据源切换。

```java

@Slf4j
public final class DataSourceContentHolder {
    private static final ThreadLocal<Deque<String>> contentHolder = new NamedThreadLocal("dynamic-datasource") {
        @Override
        protected Object initialValue() {
//            return new LinkedList<>();
            return new ArrayDeque<>();
        }
    };

    private DataSourceContentHolder() {
    }

    /**
     * 给当前线程添加数据源
     *
     * @param dataSource
     */
    public static void add(String dataSource) {
        log.debug("添加数据源 => {}", dataSource);
        contentHolder.get().add(dataSource);
    }

    /**
     * 获取当前线程数据源
     *
     * @return
     */
    public static String get() {
        String ds = contentHolder.get().peek();
        ds = StringUtils.isNotBlank(ds) ? ds : DynamicDataSourceProperties.PRIMARY;
        log.debug("获取当前数据源 => {}", ds);
        return ds;
    }

    /**
     * 清空当前数据源
     * 如果当前数据源不为空，则会只移除队列的元素
     */
    public static void poll() {
        Deque<String> queue = contentHolder.get();
        String ds = queue.poll();
        log.debug("移除数据源 => {}", ds);
        if (queue.isEmpty()) {
            contentHolder.remove();
        }
    }

    /**
     * 嵌套执行方法
     *
     * @param dataSource
     * @param callback
     */
    public static void call(String dataSource, Runnable callback) {
        try {
            add(dataSource);
            callback.run();
        } finally {
            poll();
        }
    }
}
```

3. 定义一个接口类并定义动态数据源的基本操作

```java

/**
 * 动态数据源配置
 */
public interface IDynamicDataSource {

    /**
     * 是否包含数据源
     *
     * @param key 数据源KEY
     * @return
     */
    boolean containKey(Object key);

    /**
     * 添加数据源
     *
     * @param key                  数据源KEY
     * @param dataSourceProperties 数据源配置
     */
    void add(Object key, DataSourceProperties dataSourceProperties);

    /**
     * 添加数据源
     *
     * @param key        数据源KEY
     * @param dataSource 数据源
     */
    void add(Object key, DataSource dataSource);

    /**
     * 覆盖数据源Maps
     *
     * @param objectObjectMap
     */
    void setMap(Map<Object, Object> objectObjectMap);

    /**
     * 删除数据源
     *
     * @param key
     */
    void del(Object key);

    /**
     * 替换数据源
     * <p>
     * 如果数据源不存在则添加，存在则替换
     * </p>
     *
     * @param key        被替换的数据源KEY
     * @param dataSource 数据源
     */
    void replace(Object key, DataSource dataSource);

    /**
     * 替换数据源
     * <p>
     * 如果数据源不存在则添加，存在则替换
     * </p>
     *
     * @param key                  被替换的数据源KEY
     * @param dataSourceProperties 数据源配置
     */
    void replace(Object key, DataSourceProperties dataSourceProperties);

    /**
     * 获取当前数据源
     *
     * @return 数据源
     */
    Object get();

    /**
     * 通过数据源KEY，获取数据源
     *
     * @param key 数据源，不存在则为NULL
     * @return
     */
    Object get(String key);

    /**
     * 获取当前数据源的Key
     *
     * @return 数据源Key
     */
    String getKey();

}
```

4.  继承`Spring`的`AbstractRoutingDataSource`并实现`IDynamicDataSource`接口完成数据源的管理

```java

/**
 * 实现多数数据源控制
 */
public class DynamicDataSource extends AbstractRoutingDataSource implements IDynamicDataSource {

    private static volatile DynamicDataSource INSTANCE;

    private static Map<Object, Object> dataSourceMap = new ConcurrentHashMap<>();

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

    public boolean containKey(Object key) {
        boolean b = dataSourceMap.containsKey(key);
        return b;
    }


    public void add(Object key, DataSourceProperties dataSourceProperties) {
        DataSource dataSource = DataSourceFactory.createDataSource(dataSourceProperties);
        add(key, dataSource);
    }

    public void add(Object key, DataSource dataSource) {
        boolean hasKey = dataSourceMap.containsKey(key);
        if (hasKey) {
//            if (o instanceof DataSource) {
//                DruidDataSource druidDataSource = (DruidDataSource) o;
//                druidDataSource.close();
//            }
            throw new SXException("数据库连接池 KEY 重复");
        }
        dataSourceMap.put(key, dataSource);
        setMap(dataSourceMap);
    }

    public void setMap(Map<Object, Object> objectObjectMap) {
        lock.lock();
        this.dataSourceMap = objectObjectMap;
        setPrimary();
        super.setTargetDataSources(dataSourceMap);
        super.afterPropertiesSet();
        lock.unlock();
    }

    /**
     * 设置主数据源
     */
    private void setPrimary() {
        Object o = this.dataSourceMap.get(DynamicDataSourceProperties.PRIMARY);
        if (o != null) {
            this.setDefaultTargetDataSource(o);
        }
    }

    public void del(Object key) {
        Object o = dataSourceMap.get(key);
        if (o != null) {
            if (o instanceof DataSource) {
                DruidDataSource dataSource = (DruidDataSource) o;
                if (dataSource != null) {
                    dataSource.close();
                    dataSourceMap.remove(key);
                    setMap(dataSourceMap);
                }
            } else {
                throw new SXException("数据池类型无法识别");
            }
        }
    }

    /**
     * @param key        被替换的数据源KEY
     * @param dataSource 数据源
     */
    public void replace(Object key, DataSource dataSource) {
        Object o = dataSourceMap.get(key);
        dataSourceMap.put(key, dataSource);
        if (DynamicDataSourceProperties.PRIMARY.equals(key)) {
            this.setDefaultTargetDataSource(dataSource);
        }
        if (o != null) {
            if (o instanceof DataSource) {
                DruidDataSource dataSource1 = (DruidDataSource) o;
                if (dataSource1 != null) {
                    dataSource1.close();
                }
            }
        }
        setMap(dataSourceMap);
    }

    /**
     * @param key                  被替换的数据源KEY
     * @param dataSourceProperties 数据源配置
     */
    public void replace(Object key, DataSourceProperties dataSourceProperties) {
        DruidDataSource dataSource = DataSourceFactory.createDataSource(dataSourceProperties);
        replace(key, dataSource);
    }


    @Override
    protected Object determineCurrentLookupKey() {
        return DataSourceContentHolder.get();
    }

    @Override
    public Object get() {
        return get(getKey());
    }

    @Override
    public Object get(String key) {
        return dataSourceMap.get(key);
    }

    @Override
    public String getKey() {
        return DataSourceContentHolder.get();
    }

}
```

5. 创建AOP切面，实施数据源切换拦截

```java

@Aspect
@Component
@Slf4j
@Order(-1)
public class DataSourceAspect {

    @Pointcut("@within(top.zsmile.common.datasource.annotation.DS) || @annotation(top.zsmile.common.datasource.annotation.DS)")
    public void dataSourceAspect() {

    }

    @Before("dataSourceAspect()")
    public void beforeSwitch(JoinPoint joinPoint) {
        MethodSignature signature = (MethodSignature) joinPoint.getSignature();
        Method method = signature.getMethod();
        DS methodDS = method.getAnnotation(DS.class);
        String value = null;
        if (methodDS != null) {
            value = methodDS.value();
        } else {
            Class<?> aClass = joinPoint.getTarget().getClass();
            DS annotation = aClass.getAnnotation(DS.class);
            if (annotation != null) {
                value = annotation.value();

            }
        }
        if (checkValue(value)) {
            DataSourceContentHolder.add(value);
        }
    }

    /**
     * 检查数据源是否存在
     *
     * @param value
     * @return
     */
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
        DataSourceContentHolder.poll();
    }
}

```

6. 创建 `Druid` 公共配置文件 (DruidProperties)、 动态数据源配置文件(DynamicProperties)、数据源配置文件(DataSourceProperties)

```java

@Data
//@ConfigurationProperties(prefix = "spring.datasource.druid")
public class DataSourceProperties {

    private String driverClassName;
    private String url;
    private String username;
    private String password;

    /**
     * Druid默认参数
     */
    private int initialSize;
    private int maxActive;
    private int minIdle;
    private long maxWait;
    private long timeBetweenEvictionRunsMillis;
    private long minEvictableIdleTimeMillis;
    private long maxEvictableIdleTimeMillis;
    private String validationQuery;
    private int validationQueryTimeout;
    private Boolean testOnBorrow;
    private Boolean testOnReturn;
    private Boolean testWhileIdle;
    private Boolean poolPreparedStatements;
    private int maxOpenPreparedStatements;
    private Boolean sharePreparedStatements;
    private String filters;
    private String connectionProperties;
}




@Data
public class DruidProperties {

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
    private Boolean testOnBorrow = false;
    private Boolean testOnReturn = false;
    private Boolean testWhileIdle = true;
    private Boolean poolPreparedStatements = true;
    private int maxOpenPreparedStatements = -1;
    private Boolean sharePreparedStatements = false;
    private String filters = "stat,wall";
    private String connectionProperties;
}

@Slf4j
@Getter
@Setter
@ConfigurationProperties(prefix = DynamicDataSourceProperties.PREFIX)
public class DynamicDataSourceProperties {
    public static final String PREFIX = "spring.datasource.dynamic";

    /**
     * 默认数据源,master
     */
    public static final String PRIMARY = "master";

    /**
     * 所有数据源配置
     */
    private Map<String, DataSourceProperties> datasource = new LinkedHashMap<>();

    /**
     * Druid配置
     */
    private DruidProperties druid = new DruidProperties();

}

```

7. 数据源配置

```java

@Configuration
@EnableConfigurationProperties(DynamicDataSourceProperties.class)
public class DynamicDataSourceConfig {

    @Resource
    private DynamicDataSourceProperties dynamicDataSourceProperties;

    /**
     * 动态数据源
     *
     * @return
     */
    @ConditionalOnMissingBean
    @Bean(name = "dynamicDataSource")
    public DynamicDataSource dynamicDataSource() {
        DynamicDataSource dynamicDataSource = DynamicDataSource.getInstance();
        Map<Object, Object> dataSourceMap = getDynamicDataSource();
        dynamicDataSource.setMap(dataSourceMap);
        return dynamicDataSource;
    }

    /**
     * mybatis-spring start config sql-session-factory
     *
     * @param dataSource
     * @return
     */
    @Bean
    public SqlSessionFactoryBeanCustomizer sqlSessionFactoryBeanCustomizer(@Qualifier("dynamicDataSource") DynamicDataSource dataSource) {
        return new SqlSessionFactoryBeanCustomizer() {
            @Override
            public void customize(SqlSessionFactoryBean factoryBean) {
                // 动态切换数据源事务必须要添加的。
                //factoryBean.setTransactionFactory(new DynamicTransactionFactory());
                factoryBean.setDataSource(dataSource);
            }
        };
    }

    /**
     * 事务管理器
     *
     * @param dynamicDataSource 动态数据源
     * @return
     */
    @ConditionalOnMissingBean
    @Bean(name = "transactionManager")
    public PlatformTransactionManager transactionManager(@Qualifier("dynamicDataSource") DynamicDataSource dynamicDataSource) {
        return new DataSourceTransactionManager(dynamicDataSource);
    }

    /**
     * 遍历数据源配置并加载
     *
     * @return
     */
    private Map<Object, Object> getDynamicDataSource() {
        DruidProperties druid = dynamicDataSourceProperties.getDruid();
        Map<String, DataSourceProperties> dataSourcePropertiesMap = dynamicDataSourceProperties.getDatasource();
        Map<Object, Object> dataSourceMap = new HashMap<>(dataSourcePropertiesMap.size());
        dataSourcePropertiesMap.forEach((k, v) -> {
            DataSourceProperties mergeProperties = DynamicDataSourceUtils.merge(v, druid);
            DruidDataSource dataSource = DataSourceFactory.createDataSource(mergeProperties);
            dataSourceMap.put(k, dataSource);
        });
        return dataSourceMap;
    }
}
```

到目前为止，已经可以支持多数据源的切换，但是会有一个问题，那就是 **添加了 Spring声明式事务注解@Transactional后就没有办法切换数据源了。**

>  其实市面上比较成熟的Mybatis Plus提供的 多数据源也会存在这个问题，查看源代码`SpringManagedTransaction`其实就可以知道原因，因为Spring 开启事务时会调getConnection()方法，其内部会缓存数据库连接。

那么问题是不是出在了数据库连接上，我们更换一下数据源的 `Connection`获取

## 多库事务问题

### 临时方案

 如何解决切库事务问题？借助`Spring`的声明式事务处理，我们可以在多次切库操作时强制开启新的事务： 

```java
@DS("slave")
@Transactional(rollbackFor = Exception.class, propagation = Propagation.REQUIRES_NEW)
```

但是处理当我们在主事务中抛出异常回滚的时候，子事务已经提交了，无法回滚，这时就产生数据错乱了。

### 实现代码

1. 先在 `DynamicDataSource` 重写 `getConnection` 方法

```java

/**
     * 处理多数据源的事务的关键
     * @return
     * @throws SQLException
     */
@Override
public Connection getConnection() throws SQLException {
    DataSource dataSource =
        super.getResolvedDataSources().get(getKey());
    return dataSource.getConnection();
}
```

2. 实现 `Transaction` 。注意这里的 `Transaction` 是 `org.apache.ibatis.transaction.Transaction`

```java

@Slf4j
public class DynamicTransaction implements Transaction {
    /**
     * 数据源
     */
    private final DataSource dataSource;
    /**
     * 主数据源连接
     */
    private Connection connection;
    /**
     * 是否开启事务
     */
    private Boolean isConnectionTransactional;
    /**
     * 是否自动提交
     */
    private Boolean autoCommit;
    /**
     * 连接标识
     */
    private String identification;

    /**
     * 其他连接缓存
     */
    private ConcurrentMap<String, Connection> connections;

    /**
     * 构造器
     *
     * @param dataSource 数据源
     */
    public DynamicTransaction(DataSource dataSource) {
        Assert.notNull(dataSource, "No DataSource specified");
        // 当前数据源Key
        this.identification = DataSourceContentHolder.get();
        this.dataSource = dataSource;
        connections = new ConcurrentHashMap<>();
        log.debug("init dynamic transaction, identify key => {},address => {}", this.identification, this);
    }

    /**
     * @return
     * @throws SQLException
     */
    @Override
    public Connection getConnection() throws SQLException {
        /* 获取当前生效的数据源标识 */
        String current = DataSourceContentHolder.get();
        log.debug("current key => {}, identify key=> {}", current, this.identification);
        // 如果当前数据源是主数据源
        if (current.equals(this.identification)) {
            // 如果为空则创建连接
            if (this.connection == null) {
                openConnection();
            }
            return this.connection;
        } else {
            /* 不是默认数据源，获取连接并设置属性 */
            if (!connections.containsKey(current)) {
                // 如果连接不包含该数据源KEY获取的连接
                try {
                    Connection conn = this.dataSource.getConnection();
                    /* 自动提交属性和主数据源保持连接 */
                    conn.setAutoCommit(this.autoCommit);
                    connections.put(current, conn);
                } catch (SQLException ex) {
                    throw new CannotGetJdbcConnectionException("could not get jdbc connection", ex);
                }
            }
            return connections.get(current);
        }
    }

    /**
     * 打开连接
     *
     * @throws SQLException
     */
    private void openConnection() throws SQLException {
        // 获取连接
        this.connection = DataSourceUtils.getConnection(this.dataSource);
        // 是否自动提交
        this.autoCommit = this.getConnection().getAutoCommit();
        // 确定当前连接是否是事务性的。
        // 即通过 Spring 的事务管理器绑定到当前线程的
        this.isConnectionTransactional =
                DataSourceUtils.isConnectionTransactional(this.connection, this.dataSource);
        log.debug("jdbc connection [{}] will {} be managed by spring", this.connection, (this.isConnectionTransactional ? "" : "not"));
    }

    /**
     * 提交事务
     *
     * @throws SQLException
     */
    @Override
    public void commit() throws SQLException {
        // 如果主事务不为空 && 如果是 Spring 管理的事务性连接 && 不是自动提交
        if (this.connection != null && this.isConnectionTransactional &&
                !this.autoCommit) {
            // 提交主连接的事务
            log.debug("committing jdbc connection [{}]", this.connection);
            this.connection.commit();

            // 遍历提交子连接的事务
            for (Connection conn : connections.values()) {
                conn.commit();
            }
        }
    }

    /**
     * 回滚事务
     *
     * @throws SQLException
     */
    @Override
    public void rollback() throws SQLException {
        if (this.connection != null && this.isConnectionTransactional &&
                !this.autoCommit) {
            log.debug("rolling back jdbc connection [{}]", this.connection);
            this.connection.rollback();
            for (Connection conn : connections.values()) {
                conn.rollback();
            }
        }
    }

    /**
     * 关闭并释放连接
     *
     * @throws SQLException
     */
    @Override
    public void close() throws SQLException {
        DataSourceUtils.releaseConnection(this.connection, this.dataSource);
        for (Connection conn : connections.values()) {
            DataSourceUtils.releaseConnection(conn, this.dataSource);
        }
    }

    /**
     * 获取事务超时时间
     *
     * @return
     * @throws SQLException
     */
    @Override
    public Integer getTimeout() throws SQLException {
        ConnectionHolder holder = (ConnectionHolder)
                TransactionSynchronizationManager.getResource(dataSource);
        if (holder != null && holder.hasTimeout()) {
            return holder.getTimeToLiveInSeconds();
        }
        return null;
    }
}

```

3. 继承 `SpringManagedTransactionFactory` 实现事务工厂 `DynamicTransactionFactory`

```java
public class DynamicTransactionFactory extends SpringManagedTransactionFactory {
    @Override
    public Transaction newTransaction(DataSource dataSource, TransactionIsolationLevel level, boolean autoCommit) {
        return new DynamicTransaction(dataSource);
    }
}
```

4. 将 `DynamicTransactionFactory` 注入 `Mybaits` 的 `SqlSessionFactory` 

```java

    /**
     * mybatis-spring start config sql-session-factory
     *
     * @param dataSource
     * @return
     */
    @Bean
    public SqlSessionFactoryBeanCustomizer sqlSessionFactoryBeanCustomizer(@Qualifier("dynamicDataSource") DynamicDataSource dataSource) {
        return new SqlSessionFactoryBeanCustomizer() {
            @Override
            public void customize(SqlSessionFactoryBean factoryBean) {
                factoryBean.setTransactionFactory(new DynamicTransactionFactory()); // 添加到这里额
                factoryBean.setDataSource(dataSource);
            }
        };
    }
```





# 分布式事务
## Atomikos
## seata



# 引用

https://blog.csdn.net/qq_35789269/article/details/128125061

https://juejin.cn/post/7103913605539561509

https://blog.csdn.net/m0_73533108/article/details/126688445

https://blog.csdn.net/demohui/article/details/109659540?ops_request_misc=&request_id=&biz_id=102&utm_term=spring%20%E4%BA%8B%E5%8A%A1%E5%86%85%E5%88%87%E6%8D%A2%E6%95%B0%E6%8D%AE%E6%BA%90&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduweb~default-0-109659540.142^v88^control,239^v2^insert_chatgpt
