[toc]

---

# 前言

> 该文章基于 springboot-2.3.5-final 和 hibernate-validator.6.1.6 版本

在日常开发中，对象的校验是一个非常重要的环节，用于校验手机邮箱，用户提交的信息是否正确。

如果我们在代码里面写入了大量的if-else不仅仅复用性极差，而且维护起来也极其麻烦，我们做项目，不单单要实现功能，还要优雅。所以求求你们别用if-else做校验了。

在这时候Validator框架应运而生，就是为了减少大量的if-else校验代码，提高开发效率。



## 概论

JSR303 定义了 Bean Validation 的标准 Validation-api，但是没有提供具体实现。hibernate Validator 是对这个规范的实现，并且 spring 内置的校验框架是 hibernate-validator。

PS：虽然由hibernate字眼，但不是跟hibernate ORM框架强关联。

- **JSR303**：JSR303是一项标准，只提供规范不提供实现。定义了校验规范即校验注解如：@Null、@NotNull、@Pattern。位于：`javax.validation.constraints`包下。
- **hibernate validator**： 是对 JSR303 规范的实现并且进行了增强和扩展。并增加了注解：@Email、@Length、@Range等。 
- **Spring Validation**： 是对Hibernate Validation的二次封装。在SpringMvc模块中添加了自动校验。并将校验信息封装到特定的类中。同时提供此类CDI基础结构

# CDI

> CDI(Contexts And Dependency Injection) 是JavaEE 6标准中一个规范，将依赖注入IOC/DI上升到容器级别， 它提供了Java EE平台上服务注入的组件管理核心，简化应该是CDI的目标，让一切都可以被注解被注入。 

CDI是一种依赖注入说明：

1. spring框架中隐式提供了此类CDI基础结构
2. hibernate-validator-cdi。提供给 独立的Java应用程序，需要它来使用注释创建`HibernateValidator`  

# springboot

## 集成

我这里用maven做例子，如果使用gradle或者其它包管理工具，替换成对应写法即可

```xml
<!--对应hibernate-validator版本6.1.6-final -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
    <version>2.3.5</version>
</dependency>
```




## 配置

### 快速失败

hibernate-validator 默认时会将所有参数校验后，再抛出异常。有时我们想一旦校验失败就马上抛出异常，可以配置注入进去

```java

@Configuration
public class ValidatorConfig {
    /**
     * 快速返回校验器
     *
     * @return
     */
    @Bean
    @ConditionalOnMissingBean(value = Validator.class)
    public Validator validator() {
        //hibernate-validator 6.x没问题，7.x有问题
        ValidatorFactory validatorFactory = Validation.byProvider(HibernateValidator.class)
                .configure()
                .failFast(true)
                .buildValidatorFactory();
        return validatorFactory.getValidator();
    }

    /**
     * 设置快速校验，返回方法校验处理器。
     * 使用MethodValidationPostProcessor注入后，会启动自定义校验器
     *
     * @return
     */
    @Bean
    @ConditionalOnMissingBean(value = MethodValidationPostProcessor.class)
    public MethodValidationPostProcessor methodValidationPostProcessor() {
        MethodValidationPostProcessor postProcessor = new MethodValidationPostProcessor();
        postProcessor.setValidator(validator());
        return postProcessor;
    }

}
```

### 全局错误拦截器

```java
package top.zsmile.core.exception;

import lombok.extern.slf4j.Slf4j;
import org.springframework.validation.BindException;
import org.springframework.validation.FieldError;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;
import top.zsmile.common.core.api.R;
import top.zsmile.common.core.api.ResultCode;import top.zsmile.common.core.exception.SXException;

import java.util.List;

@Slf4j
@RestControllerAdvice
public class SXExceptionHandler {

    /**
     * 自定义异常
     */
    @ExceptionHandler(SXException.class)
    public R handleException(SXException e) {
        log.error(e.getMessage(), e);
        return R.fail(e.getMessage());
    }

    private static final String SPLINT = ",";

    @ExceptionHandler(BindException.class)
    public R BindException(BindException e) {
        log.error(e.getMessage(), e);

        List<FieldError> fieldErrors = e.getBindingResult().getFieldErrors();
        StringBuffer sb = new StringBuffer("");
        for (FieldError fieldError : fieldErrors) {
            sb.append(fieldError.getDefaultMessage()).append(SPLINT);
        }
        return R.fail(sb.substring(0, sb.length() - 1));
    }
}

```



### 自定义校验器

Bean验证主要有两部分构成：

1. 声明约束及其可配置属性的@Constraint注释。
2. constraintvalidator接口的实现，它实现了约束的行为。

这里实现了一个手机号验证。

```java
@Target({ElementType.METHOD, ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
@Constraint(validatedBy = PhoneValidator.class)
public @interface Phone {
    /**
     * 异常消息
     * @return
     */
    String message() default "手机号格式不正确";

    /**
     * 分组
     * @return
     */
    Class<?>[] groups() default {};

    Class<? extends Payload>[] payload() default {};
}

```

```java
public class PhoneValidator implements ConstraintValidator<Phone, String> {
    @Override
    public void initialize(Phone constraintAnnotation) {
		// 在这里获取一些注解上的值，比如message,require，设置为局部变量即可。
    }

    @Override
    public boolean isValid(String phone, ConstraintValidatorContext constraintValidatorContext) {
        if (StringUtils.isEmpty(phone)) {
            return true;
        }
        return ValidatorUtil.isPhone(phone);
    }
}
```

这里简单封装一个ValidatorUtil库

```java

public class ValidatorUtil {
    private static final Pattern MOBILE_PATTERN = Pattern.compile("^(13[0-9]|14[5|7|9]|15[0|1|2|3|5|6|7|8|9]|17[0|1|6|7|8]|18[0-9])\\d{8}$");

    public static boolean isPhone(String src) {
        if (StringUtils.isEmpty(src)) {
            return false;
        }
        Matcher matcher = MOBILE_PATTERN.matcher(src);
        return matcher.matches();
    }
}

```



## 使用案例

### 方案1（@Valid/@Validated+Entity）

> 使用实体类，在实体类上加校验注解，在对应接口上添加@Valid，实现接口校验。

```java
// 实体类
@Data
public class DatabaseConnEntity implements Serializable {
    private static final long serialVersionUID = -4123125798831566630L;
    /**
     * 连接类型：mysql
     */
    @NotBlank(message = "连接类型不能为空")
    private String type;
    
    @Phone
    private String phone;
}

// 接口
@PostMapping("/connect")
public R connect(@Valid DatabaseConnEntity databaseConnEntity) {
    return R.success();
}

// 接口2
@PostMapping("/connect")
public R connect(@Validated DatabaseConnEntity databaseConnEntity) {
    return R.success();
}
```

### 方案2（@Validated+Params）

> 在接受参数上加校验注解

```java
@Validated
@RestController
@RequestMapping("/generator")
public class GeneratorController {
    @GetMapping("/info")
    public R info(@NotBlank String tableName) {
        Map<String, String> maps = generatorService.queryTable(tableName);
        if (maps == null) {
            return R.fail("查询不到该表结构");
        } else {
            return R.success(maps);
        }
    }
}
```

### 方案3（使用Validator）

```java

public class ValidatorUtils {

    private final static Validator validator;

    static {
        // 这里可以使用 Factory工厂，也可使用Spring容器里的Bean对象。
        if (SpringContextUtils.getBean(Validator.class) != null) {
            validator = SpringContextUtils.getBean(Validator.class);
        } else {
            validator = Validation.buildDefaultValidatorFactory().getValidator();
        }
    }
    /**
     * 校验对象
     *
     * @param object 待校验对象
     * @param groups 待校验的组
     * @throws SXException 校验不通过，则报SXException异常
     */
    public static void validateEntity(Object object, Class<?>... groups)
            throws SXException {
        Set<ConstraintViolation<Object>> constraintViolations = validator.validate(object, groups);
        if (!constraintViolations.isEmpty()) {
            StringBuilder msg = new StringBuilder();
            for (ConstraintViolation<Object> constraint : constraintViolations) {
                msg.append(constraint.getMessage()).append("<br>");
            }
            throw new SXException(msg.toString());
        }
    }
}

// 使用方法
ValidatorUtils.validator(Phone);
```

### 分组校验

#### 组校验

1. 设置组

```java
public interface Add(){ }
public interface Update(){}
```

2. 在实体类里设置组

```java
public class user{
    @NotBlank(message = "手机不能为空",groups = {Add.class})
    @Phone(groups={Update.class})
    private String phone;
}
```

3. 校验

```java
// 接口2
@PostMapping("/connect")
public R connect(@Validated({Add.class}) DatabaseConnEntity databaseConnEntity) {
    return R.success();
}
```

#### 组序列

>  默认情况下 **不同级别的约束验证是无序的**，但是在一些情况下，顺序验证却是很重要。 
>
>  一个组可以定义为其他组的序列，使用它进行验证的时候必须符合该序列规定的顺序。在使用组序列验证的时候，如果序列前边的组验证失败，则后面的组将不再给予验证。 

```java
@GroupSequence({Add.class, Update.class})
public interface change(){}
```

用法和组校验一样，只需要设置成该分组即可

# Hibernate-validator-cdi(TODO，非本文重点，后续研究)

```xml
<dependency>
    <groupId>org.hibernate.validator</groupId>
    <artifactId>hibernate-validator</artifactId>
    <version>6.1.6-final</version>
</dependency>

<dependency>
    <groupId>org.hibernate.validator</groupId>
    <artifactId>hibernate-validator-cdi</artifactId>
    <version>6.1.6-final</version>
</dependency>
```



# 其它

## @Validated 和 @Valid

- 相同：
  1. 在Controller中校验方法参数时，使用@Valid和@Validated并无特殊差异（不需要分组校验的情况下）
  2. 都支持方法和参数注解
- 不同：

	1. @Valid是属于javax.validation包，而@Validated属于Spring下的。
	2. @Validated支持分组使用，@Valid不支持
	3. @Valid支持嵌套校验，而@Validated不支持
	4. @Valid支持成员属性、构造函数注解，而@Validated不支持


# 参考文章

- [spring-validator](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#validation-beanvalidation)
- https://www.cnblogs.com/sanye613/p/15027448.html
- https://rumenz.com/java-topic/hibernate/hibernate-validator-cdi/index.html

