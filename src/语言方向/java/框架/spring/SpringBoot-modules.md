[toc]

---

# springboot 多模块配置注意点

## 模块依赖且项目下的包路径不一致

当有模块A和模块B时，假设项目结构如下

- module A 
  - com
    - aaa
      - TestABean.java
- module B
  - com
    - bbb
      - TestBBean.java
      - SpringbootApplication.java

此时SpringbootApplication是无法加载到com.aaa下的TestABean。这时因为com.aaa和com.bbb不属于同一目录下，而springbootApplication的扫描，默认是当前目录作为相对根路径进行扫描组件的，那么我们应该怎么处理这个问题。
有几种解决方式：

1. 利用@ComponentScan

```java
@ComponentScan(basePackages = {"com.aaa","com.bbb"})
```

2. 利用@ComponentScans

```java
@ComponentScans({@ComponentScan("com.aaa"),@ComponentScan("com.bbb")})
```

2. 利用@SpringBootApplication。

```java
@SpringBootApplication(scanBasePackages = {"com.aaa","com.bbb"})
```



都能实现扫描到包的效果，那么这三者有什么异同呢？

1. @SpringBootApplication 默认会自动扫描同一包下的子包。

2. @SpringBootApplication和@ComponentScan设置后，都会覆盖@SpringBootApplication的默认扫描行为
3. @ComponentScans不会覆盖@SpringBootApplication的默认扫描行为

```java
// 无法加载到TestBBean
@ComponentScan(basePackages = {"com.aaa"})
@SpringBootApplication(scanBasePackages = {"com.aaa"})

// 可以加载到TestBBean
@ComponentScan(basePackages = {"com.aaa","com.bbb"})
@SpringBootApplication(scanBasePackages = {"com.aaa","com.bbb"})
@ComponentScans({@ComponentScan("com.aaa")})
```

## 包扫描启动分析TODO

这里用的 Spring Boot 2.3.5.RELEASE 版本。

这里只关注重点的关键代码。

```java
// ConfigurationClassParser
/**
	 * Apply processing and build a complete {@link ConfigurationClass} by reading the
	 * annotations, members and methods from the source class. This method can be called
	 * multiple times as relevant sources are discovered.
	 * @param configClass the configuration class being build
	 * @param sourceClass a source class
	 * @return the superclass, or {@code null} if none found or previously processed
	 */
@Nullable
protected final SourceClass doProcessConfigurationClass(
    ConfigurationClass configClass, SourceClass sourceClass, Predicate<String> filter)
    throws IOException {

    // ...

    // Process any @ComponentScan annotations
    Set<AnnotationAttributes> componentScans = AnnotationConfigUtils.attributesForRepeatable(
        sourceClass.getMetadata(), ComponentScans.class, ComponentScan.class);
    if (!componentScans.isEmpty() &&
        !this.conditionEvaluator.shouldSkip(sourceClass.getMetadata(), ConfigurationPhase.REGISTER_BEAN)) {
        for (AnnotationAttributes componentScan : componentScans) {
            // The config class is annotated with @ComponentScan -> perform the scan immediately
            Set<BeanDefinitionHolder> scannedBeanDefinitions =
                this.componentScanParser.parse(componentScan, sourceClass.getMetadata().getClassName());
            // Check the set of scanned definitions for any further config classes and parse recursively if needed
            for (BeanDefinitionHolder holder : scannedBeanDefinitions) {
                BeanDefinition bdCand = holder.getBeanDefinition().getOriginatingBeanDefinition();
                if (bdCand == null) {
                    bdCand = holder.getBeanDefinition();
                }
                if (ConfigurationClassUtils.checkConfigurationClassCandidate(bdCand, this.metadataReaderFactory)) {
                    parse(bdCand.getBeanClassName(), holder.getBeanName());
                }
            }
        }
    }
    // ...
}
```

这里获取了ComponenetScan和ConponentScans两个注解配置，让我们看看 AnnotationConfigUtils.attributesForRepeatable 里面发生了什么。

```java
// AnnotationConfigUtils
static Set<AnnotationAttributes> attributesForRepeatable(AnnotationMetadata metadata,
			Class<?> containerClass, Class<?> annotationClass) {

    return attributesForRepeatable(metadata, containerClass.getName(), annotationClass.getName());
}

@SuppressWarnings("unchecked")
static Set<AnnotationAttributes> attributesForRepeatable(
    AnnotationMetadata metadata, String containerClassName, String annotationClassName) {

    Set<AnnotationAttributes> result = new LinkedHashSet<>();

    // Direct annotation present?
    addAttributesIfNotNull(result, metadata.getAnnotationAttributes(annotationClassName, false));

    // Container annotation present?
    Map<String, Object> container = metadata.getAnnotationAttributes(containerClassName, false);
    if (container != null && container.containsKey("value")) {
        for (Map<String, Object> containedAttributes : (Map<String, Object>[]) container.get("value")) {
            addAttributesIfNotNull(result, containedAttributes);
        }
    }

    // Return merged result
    return Collections.unmodifiableSet(result);
}
```



### 参考链接

- https://my.oschina.net/icebergxty/blog/3142263
- https://my.oschina.net/icebergxty/blog/3103354