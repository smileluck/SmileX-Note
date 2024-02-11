[toc]

---



# 前言

这边文章是通过使用注解+AOP的形式，在业务代码前后做日志增强的功能。这样在不更改业务代码的情况下，利用注解作为 Pointcut 快速实现日志记录的功能。

# 准备工作

## Maven版本

这里是基于springboot-2.3.5版本，需要引入 SpringAop 库。

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

## 数据库结构

我这里采用的将接口访问日志，存储到 Mysql 数据库中，其它数据库原理一致，具体根据自己项目结构进行调整。

```sql
CREATE TABLE `sys_log` (
  `id` bigint(20) NOT NULL COMMENT 'ID',
  `log_module` varchar(50) NOT NULL COMMENT '日志模块，sys,blog',
  `log_title` varchar(50) NOT NULL COMMENT '日志标题',
  `log_value` varchar(50) NOT NULL COMMENT '日志内容',
  `log_type` tinyint(2) NOT NULL COMMENT '日志类型1:登录日志;2:操作日志;3:定时任务;4:异常日志;',
  `user_id` bigint(20) NOT NULL COMMENT '用户ID',
  `operate_type` tinyint(2) NOT NULL COMMENT '操作类型',
  `ip_address` varchar(100) DEFAULT NULL COMMENT 'IP地址',
  `method` varchar(500) DEFAULT NULL COMMENT '请求方法',
  `request_url` varchar(50) DEFAULT NULL COMMENT '请求url路径',
  `request_type` varchar(50) DEFAULT NULL COMMENT '请求类型',
  `request_params` text COMMENT '请求参数',
  `cost_time` int(11) DEFAULT NULL COMMENT '耗费时间',
  `err_msg` varchar(1000) DEFAULT NULL COMMENT '异常信息',
  `create_time` datetime DEFAULT NULL COMMENT '创建时间',
  `create_by` varchar(50) DEFAULT NULL COMMENT '创建人',
  `update_time` datetime DEFAULT NULL COMMENT '更新时间',
  `update_by` varchar(50) DEFAULT NULL COMMENT '更新人',
  `del_flag` tinyint(1) NOT NULL DEFAULT '0' COMMENT '是否删除，1是，0否',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='系统日志';
```

## 常量值

我将系统中使用的常量值单独分离了出来。

```java
public interface CommonConstant {
    /**************************SysLog 日志常量**************************/
    //查询
    public static final int SYS_LOG_OPERATE_QUERY = 1;
    //添加
    public static final int SYS_LOG_OPERATE_SAVE = 2;
    //更新
    public static final int SYS_LOG_OPERATE_UPDATE = 3;
    //删除
    public static final int SYS_LOG_OPERATE_REMOVE = 4;
    //导入
    public static final int SYS_LOG_OPERATE_IMPORT = 5;
    //导出
    public static final int SYS_LOG_OPERATE_EXPORT = 6;

    //登录
    public static final int SYS_LOG_TYPE_LOGIN = 1;
    //操作
    public static final int SYS_LOG_TYPE_OPERATE = 2;
    //定时
    public static final int SYS_LOG_TYPE_TIME = 3;
    //异常
    public static final int SYS_LOG_TYPE_ERROR = 4;
}
```



# 核心

这里的核心基于两个事物：

1. @interface 注解类
2. @Aspect 切面编程

## @SysLog

这里根据我对系统的设计划分

- module：所属模块，可以划分大模块比如(Sys,系统模块)，(Blog,博客模块)。
- title：子模块标题，可以简单划分大模块下的子模块。
- logType：日志类型，分为1:登录日志;2:操作日志;3:定时任务;4:异常日志。
- opreateType：操作类型，1查询，2添加，3修改，4删除，5导入，6导出。
- 日志内容：日志内容。

```java

/**
 * 系统日志
 */
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface SysLog {

    /**
     * 所属模块
     *
     * @return ModuleType
     */
    ModuleType module() default ModuleType.SYS;

    /**
     * 标题
     * 例如：菜单管理
     *
     * @return
     */
    String title() default "";

    /**
     * 日志类型
     *
     * @return 1:登录日志;2:操作日志;3:定时任务;4:异常日志;
     */
    int logType() default CommonConstant.SYS_LOG_TYPE_OPERATE;

    /**
     * 操作类型
     *
     * @return （1查询，2添加，3修改，4删除，5导入，6导出）
     */
    int operateType() default 0;

    /**
     * 日志内容
     *
     * @return
     */
    String value() default "";
}

```

## SysLogAspect

这里有几个需要注意的地方

1. 需要在类上添加@Component 和 @Aspect 注解，前者是为了将SysLogAspect 作为一个Bean，注入到Spring容器中；后者是为了开启切面功能
2. @Pointcut 注入点，注入点说明该切面在什么情况下生效，这里使用 "@annotation(top.zsmile.common.log.annotation.SysLog)" 表示只要是有加@SysLog注解的地方就能注入切面功能。
3. @Around，括号内使用的就是对应的切点sysLogPointcut()，这时around方法就会执行。
4. ProceedingJoinPoint joinPoint，指的是对应的执行点，当我们调用proceed()时，会调用对应的业务代码。

```java
@Slf4j
@Component
@Aspect
public class SysLogAspect {
    @Autowired
    private SysLogService sysLogService;

    @Autowired
    private CommonAuthApi commonAuthApi;

    @Pointcut("@annotation(top.zsmile.common.log.annotation.SysLog)")
    public void sysLogPointcut() {

    }

    @Around("sysLogPointcut()")
    public Object around(ProceedingJoinPoint joinPoint) throws Throwable {

        long beginTime = System.currentTimeMillis();
        try {
            //执行方法
            Object result = joinPoint.proceed();
            //执行时长(毫秒)
            long time = System.currentTimeMillis() - beginTime;
            //保存日志
            saveSysLog(joinPoint, time, result);
            return result;
        } catch (Throwable throwable) {
            // 异常日志
            long time = System.currentTimeMillis() - beginTime;
            saveErrorLog(joinPoint, time, throwable.getMessage());
            throw throwable;
        }
    }

    /**
     * 保存错误日志
     *
     * @param joinPoint
     * @param costTime
     * @param errMsg
     */
    private void saveErrorLog(ProceedingJoinPoint joinPoint, long costTime, String errMsg) {
        SysLogEntity sysLogEntity = commonLog(joinPoint, costTime, CommonConstant.SYS_LOG_TYPE_ERROR);
        sysLogEntity.setErrMsg(errMsg);
        sysLogService.save(sysLogEntity);
    }

    /**
     * 保存正常日志
     *
     * @param joinPoint
     * @param costTime
     * @param result
     */
    private void saveSysLog(ProceedingJoinPoint joinPoint, long costTime, Object result) {
        SysLogEntity sysLogEntity = commonLog(joinPoint, costTime);
        sysLogService.save(sysLogEntity);
    }

    private SysLogEntity commonLog(ProceedingJoinPoint joinPoint, long costTime) {
        return commonLog(joinPoint, costTime, 0);
    }

    private SysLogEntity commonLog(ProceedingJoinPoint joinPoint, long costTime, int logType) {
        MethodSignature signature = (MethodSignature) joinPoint.getSignature();
        Method method = signature.getMethod();

        SysLogEntity sysLogEntity = new SysLogEntity();
        SysLog sysLogAnno = method.getAnnotation(SysLog.class);
        if (sysLogAnno != null) {
            String title = sysLogAnno.title();
            ModuleType module = sysLogAnno.module();
            int operateType = sysLogAnno.operateType();
            String value = sysLogAnno.value();
            sysLogEntity.setLogTitle(title);
            sysLogEntity.setLogValue(value);
            sysLogEntity.setLogType(logType > 0 ? logType : sysLogAnno.logType());
            sysLogEntity.setOperateType(getOperateType(method.getName(), operateType));
            sysLogEntity.setLogModule(module.name());
        }

        //获取request
        HttpServletRequest request = SpringContextUtils.getHttpServletRequest();
        sysLogEntity.setIpAddress(IPUtils.getIpAddrByRequest(request));

        //请求的方法名
        String className = joinPoint.getTarget().getClass().getName();
        String methodName = signature.getName();
        sysLogEntity.setMethod(className + "." + methodName);

        // 请求类型
        sysLogEntity.setRequestType(request.getMethod());

        // 耗时
        sysLogEntity.setCostTime(costTime);

        // 请求参数
        sysLogEntity.setRequestParams(getParams(joinPoint, request));

        // 操作用户Id
        Long userId = commonAuthApi.queryUserId();
        sysLogEntity.setUserId(userId);

        return sysLogEntity;
    }

    private int getOperateType(String methodName, int operateType) {
        if (operateType > 0) {
            return operateType;
        }
        if (methodName.startsWith("query") || methodName.startsWith("list")) {
            return CommonConstant.SYS_LOG_OPERATE_QUERY;
        } else if (methodName.startsWith("save")) {
            return CommonConstant.SYS_LOG_OPERATE_SAVE;
        } else if (methodName.startsWith("update")) {
            return CommonConstant.SYS_LOG_OPERATE_UPDATE;
        } else if (methodName.startsWith("remove")) {
            return CommonConstant.SYS_LOG_OPERATE_REMOVE;
        } else if (methodName.startsWith("import")) {
            return CommonConstant.SYS_LOG_OPERATE_IMPORT;
        } else if (methodName.startsWith("export")) {
            return CommonConstant.SYS_LOG_OPERATE_EXPORT;
        }
        return CommonConstant.SYS_LOG_OPERATE_QUERY;
    }

    /**
     * 获取Request参数
     *
     * @param httpServletRequest
     * @return
     */
    private String getParams(ProceedingJoinPoint joinPoint, HttpServletRequest httpServletRequest) {
        String method = httpServletRequest.getMethod();
        if (method.equalsIgnoreCase("POST") || method.equalsIgnoreCase("PUT") || method.equalsIgnoreCase("DELET")) {
            PropertyFilter profilter = new PropertyFilter() {
                @Override
                public boolean apply(Object o, String name, Object value) {
                    if (value != null && value.toString().length() > 200) {
                        return false;
                    }
                    return true;
                }
            };
            return JSON.toJSONString(joinPoint.getArgs(), profilter);
        } else {
            return JSON.toJSONString(httpServletRequest.getParameterMap());
        }
    }
}

```

# 使用方法

```java
@SysLog(title = "角色管理", operateType = CommonConstant.SYS_LOG_OPERATE_QUERY, value = "分页查询")
```

