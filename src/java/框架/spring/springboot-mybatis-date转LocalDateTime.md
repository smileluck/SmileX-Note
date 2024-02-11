[toc]

---

# mybatis使用LocalDateTime

1. 查看mybatis版本。 从 3.4.5 开始，MyBatis 默认支持 JSR-310 。如果是3.4.5以下的版本需要引入 JSR-310。

```xml
<dependency>
  <groupId>org.mybatis</groupId>
  <artifactId>mybatis-typehandlers-jsr310</artifactId>
  <version>1.0.2</version>
</dependency>
```

2. 给JDBC连接加上时区属性，避免时间出现偏差。

```java
&serverTimezone=Asia/Shanghai
```



# springboot使用LocalDateTime

## maven引入

- 以下几种情况不用单独引入jackson-datatype-jsr
  - 引入了 spring-boot-starter-web 
  - 引入了 spring-boot-starter-json

```xml
<dependency>
         <groupId>com.fasterxml.jackson.datatype</groupId>
         <artifactId>jackson-datatype-jsr310</artifactId>
 </dependency>
```

## 使用jackson 全局统一转换

- 利用ObjectMapper输入输出转换时间格式，添加时间序列化。

```java
public static final String DEFAULT_DATE_TIME_FORMAT = "yyyy-MM-dd HH:mm:ss";
public static final String DEFAULT_DATE_FORMAT = "yyyy-MM-dd";
public static final String DEFAULT_TIME_FORMAT = "HH:mm:ss";

@Bean
public ObjectMapper jacksonObjectMapper(Jackson2ObjectMapperBuilder builder) {
    ObjectMapper objectMapper = builder.createXmlMapper(false).build();

	// 简单类型转换
    SimpleModule module = new SimpleModule();
    module.addSerializer(Long.class, ToStringSerializer.instance);
    module.addSerializer(Long.TYPE, ToStringSerializer.instance);

    /**
         * localDateTime => yyyy-MM-dd HH:mm:ss
         * localDate => yyyy-MM-dd
         * localTime => HH:mm:ss
         */
    JavaTimeModule javaTimeModule = new JavaTimeModule();
    module.addSerializer(LocalDateTime.class, new LocalDateTimeSerializer(DateTimeFormatter.ofPattern(DEFAULT_DATE_TIME_FORMAT)));
    module.addDeserializer(LocalDateTime.class, new LocalDateTimeDeserializer(DateTimeFormatter.ofPattern(DEFAULT_DATE_TIME_FORMAT)));
    module.addSerializer(LocalDate.class, new LocalDateSerializer(DateTimeFormatter.ofPattern(DEFAULT_DATE_FORMAT)));
    module.addDeserializer(LocalDate.class, new LocalDateDeserializer(DateTimeFormatter.ofPattern(DEFAULT_DATE_FORMAT)));
    module.addSerializer(LocalTime.class, new LocalTimeSerializer(DateTimeFormatter.ofPattern(DEFAULT_TIME_FORMAT)));
    module.addDeserializer(LocalTime.class, new LocalTimeDeserializer(DateTimeFormatter.ofPattern(DEFAULT_TIME_FORMAT)));
    objectMapper.registerModules(module, javaTimeModule);
    // 不序列化为null
    objectMapper.setSerializationInclusion(JsonInclude.Include.NON_NULL);
    return objectMapper;
}
```

- 利用 Jackson2ObjectMapperBuilderCustomizer 序列化

```java
public static final String DEFAULT_DATE_TIME_FORMAT = "yyyy-MM-dd HH:mm:ss";
public static final String DEFAULT_DATE_FORMAT = "yyyy-MM-dd";
public static final String DEFAULT_TIME_FORMAT = "HH:mm:ss";

@Bean
public Jackson2ObjectMapperBuilderCustomizer jackson2ObjectMapperBuilderCustomizer() {
    return builder -> {
        builder.serializerByType(LocalDateTime.class, new LocalDateTimeSerializer(DateTimeFormatter.ofPattern(DEFAULT_DATE_TIME_FORMAT)));
        builder.deserializerByType(LocalDateTime.class, new LocalDateTimeDeserializer(DateTimeFormatter.ofPattern(DEFAULT_DATE_TIME_FORMAT)));
        builder.serializerByType(LocalDate.class, new LocalDateSerializer(DateTimeFormatter.ofPattern(DEFAULT_DATE_FORMAT)));
        builder.deserializerByType(LocalDate.class, new LocalDateDeserializer(DateTimeFormatter.ofPattern(DEFAULT_DATE_FORMAT)));
        builder.serializerByType(LocalTime.class, new LocalTimeSerializer(DateTimeFormatter.ofPattern(DEFAULT_TIME_FORMAT)));
        builder.deserializerByType(LocalTime.class, new LocalTimeDeserializer(DateTimeFormatter.ofPattern(DEFAULT_TIME_FORMAT)));
    };
}
```



## 局部转换

### @JsonFormat 注解

利用注解 @JsonFormat 加在需要用的实体类时间字段上，就会完成格式化。

```java
public class testVO {
    @JsonFormat(locale = "zh", timezone = "GMT+8", pattern = "yyyy-MM-dd HH:mm:ss")
    private LocalDateTime createTime;

    @JsonFormat(locale = "zh", timezone = "GMT+8", pattern = "yyyy-MM-dd")
    private LocalDate updateTime;
}
```

