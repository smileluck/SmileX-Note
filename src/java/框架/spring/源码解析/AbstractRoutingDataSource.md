[toc]

---



# AbstractRoutingDataSource 
我们先看一下这个类里面包含了几个变量。
```java
   @Nullable
   private Map<Object, Object> targetDataSources;

   @Nullable
   private Object defaultTargetDataSource;

   private boolean lenientFallback = true;

   private DataSourceLookup dataSourceLookup = new JndiDataSourceLookup();

   @Nullable
   private Map<Object, DataSource> resolvedDataSources;

   @Nullable
   private DataSource resolvedDefaultDataSource;
```
这里我们先简单的说一下各个变量的作用：
- targetDataSources：存储未解析的目标的数据源Map
- defaultTargetDataSource: 未解析的默认的目标数据源
- lenientFallback：回退。稍后再说
- dataSourceLookup：dataSource解析器，默认是JNDI
- resolvedDataSources：将目标的数据源解析并保存成实际的数据链接后的map
- resolvedDefaultDataSource: 默认的数据源

## Method afterPropertiesSet
> afterPropertiesSet 是用来解析targetDataSources 来获取真正的dataSource
```java
@Override
public void afterPropertiesSet() {
    if (this.targetDataSources == null) {
        throw new IllegalArgumentException("Property 'targetDataSources' is required");
    }
    this.resolvedDataSources = new HashMap<>(this.targetDataSources.size());
    this.targetDataSources.forEach((key, value) -> {
        Object lookupKey = resolveSpecifiedLookupKey(key);
        DataSource dataSource = resolveSpecifiedDataSource(value);
        this.resolvedDataSources.put(lookupKey, dataSource);
    });
    if (this.defaultTargetDataSource != null) {
        this.resolvedDefaultDataSource = resolveSpecifiedDataSource(this.defaultTargetDataSource);
    }
}
```

1. 该方法会先判断targetDataSources是否为空，如果为空抛出异常。
2. 新建一下HashMap，大小为targetDataSources的大小，然后赋值给resolvedDataSources
3. 遍历targetDataSources，解析Key和DataSource，并存储到resolvedDataSources
4. 如果设置了 defaultTargetDataSource【默认数据源】，则会将默认数据源解析后，存储到resolveDefaultDataSource 

以上的流程可以看出，该方法必须在设置了targetDataSources后调用，才能生效。

所以我们在做多数据源时，有时会看到这样的代码：这是为了解析我们配置的数据源Bean。
```java
AbstractRoutingDataSource.setTargetDataSources(Map(...))
AbstractRoutingDataSource.afterPropertiesSet();
```


## Method resolveSpecifiedDataSource

1. 当参数DataSource的类型为DataSource时候，则返回
2. 如果是String时，因为默认时使用的JNDILookUp，所以这里会默认去根据JNDI解析。如果需要替换不同的datasourceLookUp，Spring内置了4种：
    - BeanFactoryDataSourceLookup：在SpringBean容器里面查找dataSource。
    - JndiDataSourceLookup：从JNDI数据源解析
    - MapDataSourceLookup：从Map容器解析
    - SingleDataSourceLookup：从单数据源直接返回
3. 如果不是以上两种类型，则抛出异常信息。

```java
/**
 * Resolve the specified data source object into a DataSource instance.
 * <p>The default implementation handles DataSource instances and data source
 * names (to be resolved via a {@link #setDataSourceLookup DataSourceLookup}).
 * @param dataSource the data source value object as specified in the
 * {@link #setTargetDataSources targetDataSources} map
 * @return the resolved DataSource (never {@code null})
 * @throws IllegalArgumentException in case of an unsupported value type
 */
protected DataSource resolveSpecifiedDataSource(Object dataSource) throws IllegalArgumentException {
    if (dataSource instanceof DataSource) {
        return (DataSource) dataSource;
    }
    else if (dataSource instanceof String) {
        return this.dataSourceLookup.getDataSource((String) dataSource);
    }
    else {
        throw new IllegalArgumentException(
                "Illegal data source value - only [javax.sql.DataSource] and String supported: " + dataSource);
    }
}

```

```java
 protected <T> T lookup(String jndiName, @Nullable Class<T> requiredType) throws NamingException {
    Assert.notNull(jndiName, "'jndiName' must not be null");
    String convertedName = this.convertJndiName(jndiName);

    Object jndiObject;
    try {
        jndiObject = this.getJndiTemplate().lookup(convertedName, requiredType);
    } catch (NamingException var6) {
        if (convertedName.equals(jndiName)) {
            throw var6;
        }

        if (this.logger.isDebugEnabled()) {
            this.logger.debug("Converted JNDI name [" + convertedName + "] not found - trying original name [" + jndiName + "]. " + var6);
        }

        jndiObject = this.getJndiTemplate().lookup(jndiName, requiredType);
    }

    if (this.logger.isDebugEnabled()) {
        this.logger.debug("Located object with JNDI name [" + convertedName + "]");
    }

    return jndiObject;
}
```