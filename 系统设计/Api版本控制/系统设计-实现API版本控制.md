[toc]

---

# 版本规范

很多时候我们开发的时候，都需要考虑接口的一个版本迭代问题。有时可能会需要重新设计接口。

1. 调整 `Controller` 目录结构

   - 直接区分 v1,v2 的方式。
     - /controller/app/v1;
     - /controller/app/v2;

2. 接口设计

   1. 在域名上做区分。
      - v1.api.com;
      - v2.api.com;

   2. 在Path上做区分
      - api.demo.com/v1/;
      - api.demo.com/v2/;
   3. 利用Request Header区分，自定义请求头，例如 api-version;

   ```shell
   curl "http://api.demo.com/test" ^
     -H "api-version: 2" ^
     --compressed
   ```

   1. 利用Request Params区分

   ```shell
   curl "http://api.demo.com/test?api-version=2"
     --compressed
   ```

3.  区分依据
   1. 任何接口在首次发布都属于初始版本(v1)接口
   2. 项目发布后，如果新增的需求和 v1 **接口不冲突**，属于新增的接口，仍然属于 v1 接口
   3. 项目发布后，如果新增的需求和 v1 **接口冲突**，为了兼容原有接口，属于v2接口
   4. 项目发布后，如果新增的需求和 v1 属于**接口改动**，相当于是兼容升级，则需要在原有代码逻辑耦合进去。

## 小结

1. 接口变化非常大或者整个产品的大版本更新
   - 可以采用URL更新版本的方式，创建新的 `Controller` 或者部署新的服务
   - 无版本号的走默认逻辑
2. 常规的接口升级和BUGFIX
   -  一般可以通过 `Header` 中指定版本号向下兼容
   - 在代码中进行判断即可满足要求
   - 无版本号的走默认逻辑
3. 两种方式同时使用
   - URL自带的方式实现大版本升级
   - 后续小版本迭代可以通过 `Header` 版本号进行兼容升级



# RequestCondition实现URL版本管理

## 实现方案

1. 创建注解 `@ApiVersion`，默认值为 1
2. 创建URL匹配规则 `ApiVersionCondition` 继承 `RequestCondition`
3. 创建 `ApiRequestMappingCondition` 匹配对应的请求，选择合适的版本

### 创建注解

```java
/**
 * API 版本控制
 *
 * @Version: 1.0.0
 * @Author: Administrator
 * @Date: 2023/04/17/21:06
 * @ClassName: ApiVersion
 * @Description: ApiVersion
 */
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
public @interface ApiVersion {

    /**
     * @return 版本
     */
    int value() default 1;
}
```

### URL匹配规则

继承 `RequestCondition` ，泛型为本身，作用是：

1. 正则匹配版本号(v1,v2)并提取出来，与注解里面的版本做对比，返回 `Condition` 或`null`
2. 设置优先级排序规则
3. 类和方法上都有注解的时候，将进行合并(combine)

```java

/**
 * @Version: 1.0.0
 * @Author: Administrator
 * @Date: 2023/04/17/21:23
 * @ClassName: ApiVersionCondition
 * @Description: ApiVersionCondition
 */
public class ApiVersionCondition implements RequestCondition<ApiVersionCondition> {
    private final static Pattern VERSION_PREFIX_PATTERN = Pattern.compile(".*v(\\d+).*");

    private int apiVersion;

    public ApiVersionCondition(int apiVersion) {
        this.apiVersion = apiVersion;
    }

    private int getApiVersion() {
        return apiVersion;
    }


    @Override
    public ApiVersionCondition combine(ApiVersionCondition apiVersionCondition) {
        return new ApiVersionCondition(apiVersionCondition.getApiVersion());
    }

    @Override
    public ApiVersionCondition getMatchingCondition(HttpServletRequest httpServletRequest) {
        Matcher m = VERSION_PREFIX_PATTERN.matcher(httpServletRequest.getRequestURI());
        if (m.find()) {
            Integer version = Integer.valueOf(m.group(1));
            if (version >= this.apiVersion) {
                return this;
            }
        }
        return null;
    }

    @Override
    public int compareTo(ApiVersionCondition apiVersionCondition, HttpServletRequest httpServletRequest) {
        return apiVersionCondition.getApiVersion() - this.apiVersion;
    }
}

```

### 匹配处理器

```java

/**
 * @Version: 1.0.0
 * @Author: Administrator
 * @Date: 2023/04/19/17:08
 * @ClassName: ApiRequestMappingHandlerMapping
 * @Description: ApiRequestMappingHandlerMapping
 */
public class ApiRequestMappingHandlerMapping extends RequestMappingHandlerMapping {
    private static final String VERSION_FLAG = "{version}";

    private static RequestCondition<ApiVersionCondition> createCondition(Class<?> clazz) {
        RequestMapping classRequestMapping = clazz.getAnnotation(RequestMapping.class);
        if (classRequestMapping == null) {
            return null;
        }
        StringBuilder mappingUrlBuilder = new StringBuilder();
        if (classRequestMapping.value().length > 0) {
            mappingUrlBuilder.append(classRequestMapping.value()[0]);
        }
        String mappingUrl = mappingUrlBuilder.toString();
        if (!mappingUrl.contains(VERSION_FLAG)) {
            return null;
        }
        ApiVersion apiVersion = clazz.getAnnotation(ApiVersion.class);
        return apiVersion == null ? new ApiVersionCondition(1) : new ApiVersionCondition(apiVersion.value());
    }

    @Override
    protected RequestCondition<?> getCustomMethodCondition(Method method) {
        return createCondition(method.getClass());
    }

    @Override
    protected RequestCondition<?> getCustomTypeCondition(Class<?> handlerType) {
        return createCondition(handlerType);
    }
}

```

### 注册处理器

```java
@Configuration
public class WebMvcRegistrationsConfig implements WebMvcRegistrations {
    @Override
    public RequestMappingHandlerMapping getRequestMappingHandlerMapping() {
        return new ApiRequestMappingHandlerMapping();
    }
}
```

### 测试接口

#### v1

```java
@Slf4j
@ApiVersion
@RestController
@RequestMapping("/app/demo/{version}")
public class AppDemoV1Controller {
    @RequestMapping("/test")
    public R test() {
        log.info("v1 test");
        return R.success("v1 test");
    }
    @RequestMapping("/extend")
    public R extend() {
        log.info("v1 extend");
        return R.success("v1 extend");
    }
}
```

#### v2

```java
@Slf4j
@RestController
@ApiVersion(2)
@RequestMapping("/app/demo/{version}")
public class AppDemoV2Controller {
    @RequestMapping("/test")
    public R test() {
        log.info("v2 test");
        return R.success("v2 test");
    }

}
```

### 效果如下

```powershell
# 调用 v1 test接口
curl http://127.0.0.1:8080/smilex/app/demo/v1/test
{"code":200,"msg":"v1 test","success":true}

# 调用 v2 test接口
curl http://127.0.0.1:8080/smilex/app/demo/v2/test
{"code":200,"msg":"v2 test","success":true}

# 调用 v1 extend接口
curl http://127.0.0.1:8080/smilex/app/demo/v1/extend
{"code":200,"msg":"v1 extend","success":true}

# 调用 V2 extend接口，实际匹配 v1 extend
curl http://127.0.0.1:8080/smilex/app/demo/v2/extend
{"code":200,"msg":"v1 extend","success":true}

# 调用 v3 test接口，不存在，向下兼容，实际调用 v2 test
curl http://127.0.0.1:8080/smilex/app/demo/v3/test
{"code":200,"msg":"v2 test","success":true}

```

### 小结

1. 请求正确的版本地址时，会自动匹配版本的对应接口
2. 请求版本大于当前版本时，会自动匹配当前版本
3. 当请求版本接口不存在时，会匹配之前版本的接口，即版本继承