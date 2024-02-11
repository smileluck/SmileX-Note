[toc]

---

# 前言

之前我们利用 `jackson` 全局配置，解决了我们程序中的 `Long类型精度丢失问题`和 `LocalDateTime和Date类型的时间格式化问题`。假设现在有这么一个需求。

```
1分钟内：刚刚
60分钟内： 40分钟前
24小时内： 15小时前
超过24小时：昨天22:42
超过24小时且跨过发布日期2天及以上：8-3 22:42
跨过发布日期年份： 2022-8-5  22:42
```

要求时间格式按照这样的形势返回。这时我们可以在每个方法返回前自己写格式化，处理完后再返回。那有什么更好的方式呢？这里我们可以采用自定义序列化类来实现。



# jackson 提供的类

- JsonSerializer<T>。序列化类，T为传入的类型。
- JsonDeserializer<T>。反序列化类，T为处理完后传入系统的类型。

## JsonSerializer

来看个简单的例子，这个例子将 `LocalDateTime`格式化后，通过`JsonGenerator.writeString`转字符串传输出去。

```java

public class LocalDateTimeSerialize extends JsonSerializer<LocalDateTime> {

    private static final DateTimeFormatter format = DateTimeFormatter.ofPattern("yyyy-M-d HH:mm");
    
    @Override
    public void serialize(LocalDateTime localDateTime, JsonGenerator jsonGenerator, SerializerProvider serializerProvider) throws IOException {
		jsonGenerator.writeString(localDateTime.format(format));
    }
}

```



## JsonDeserializer

来看个简单的例子，这个例子将 `LocalDateTime`格式化后，通过`JsonParser.writeString`转字符串传输出去。

```java
public class LocalDateTimeDeserialize extends JsonDeserializer<LocalDateTime> {

    private static final DateTimeFormatter format = DateTimeFormatter.ofPattern("yyyy-M-d HH:mm");
    
 	public Date deserialize(JsonParser jp, DeserializationContext ctxt) throws IOException, JsonProcessingException {
        String date = jp.getText();
        try {
            return format.parse(date);
        } catch (ParseException e) {
            throw new RuntimeException(e);
        }
    }
}

```

## 如何使用

现在序列化类和反序列化类我们都有了，那么我们应该如何使用呢？

### 注解方式使用

在对应的类型上添加类注解。

- @JsonDeserialize
- @JsonSerialize

```java
@Data
public class Message{
    @JsonDeserialize(using = LocalDateTimeDeserialize.class)
    @JsonSerialize(using = LocalDateTimeSerialize.class)
    private LocalDateTime createTime;
}
```

### 全局注册

```java

@JsonComponent
public class JacksonConfig {
    
    @Bean
    public ObjectMapper jacksonObjectMapper(Jackson2ObjectMapperBuilder builder) {
        ObjectMapper objectMapper = builder.createXmlMapper(false).build();

        // 简单类型转换
        SimpleModule module = new SimpleModule();
        module.addSerializer(Long.class, ToStringSerializer.instance);
        module.addSerializer(Long.TYPE, ToStringSerializer.instance);

        JavaTimeModule javaTimeModule = new JavaTimeModule();
        module.addSerializer(LocalDateTime.class, new LocalDateTimeSerialize());
        module.addDeserializer(LocalDateTime.class, new LocalDateTimeDeserialize());
        objectMapper.registerModules(module, javaTimeModule);
        // 不序列化为null
        objectMapper.setSerializationInclusion(JsonInclude.Include.NON_NULL);
        return objectMapper;
    }


}
```

# 一些实现的自定义序列化类

# 将LocalDateTime转化成相对时间

需求：

```1分钟内：刚刚
1分钟内：刚刚
60分钟内： 40分钟前
24小时内： 15小时前
超过24小时：昨天22:42
超过24小时且跨过发布日期2天及以上：8-3 22:42
跨过发布日期年份： 2022-8-5  22:42
```

实现：

```java

/**
 * 时间序列化注解
 * 使用方法： @JsonSerialize(using = LocalDateTimeSerialize.class)
 */
public class LocalDateTimeSerialize extends JsonSerializer<LocalDateTime> {

    private static final DateTimeFormatter format = DateTimeFormatter.ofPattern("yyyy-M-d HH:mm");
    private static final DateTimeFormatter format2 = DateTimeFormatter.ofPattern("M-d HH:mm");
    private static final DateTimeFormatter format3 = DateTimeFormatter.ofPattern("昨天HH:mm");

    /**
     * 分钟内：刚刚
     * 60分钟内： 40分钟前
     * 24小时内： 15小时前
     * 超过24小时：昨天22:42
     * 超过24小时且跨过发布日期2天及以上：8-3 22:42
     * 跨过发布日期年份： 2022-8-5  22:42
     */
    @Override
    public void serialize(LocalDateTime localDateTime, JsonGenerator jsonGenerator, SerializerProvider serializerProvider) throws IOException {
        LocalDateTime now = LocalDateTime.now();
        if (localDateTime.getYear() < now.getYear()) {
            jsonGenerator.writeString(localDateTime.format(format));
            return;
        }
        Duration duration = Duration.between(localDateTime, now);
        if (duration.getSeconds() <= 60) {// 小于60s
            jsonGenerator.writeString("刚刚");
        } else if (duration.toMinutes() <= 60) {//小于一小时
            jsonGenerator.writeString(duration.toMinutes() + "分钟前");
        } else if (duration.toHours() < 24) {//小于24小时
            jsonGenerator.writeString(duration.toHours() + "小时前");
        } else if (duration.toDays() == 1) {
            jsonGenerator.writeString(localDateTime.format(format3));
        } else if (duration.toDays() >= 2) {
            jsonGenerator.writeString(localDateTime.format(format2));
        }else{
            jsonGenerator.writeString(localDateTime.format(format2));
        }
    }
}

```

