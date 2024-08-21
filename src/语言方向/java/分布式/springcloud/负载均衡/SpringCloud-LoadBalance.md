[TOC]

---

# LoadBalancer

## 前言

在 2020 年以前的 SpringCloud 采用 [Ribbon](https://so.csdn.net/so/search?q=Ribbon&spm=1001.2101.3001.7020) 作为负载均衡，但是 2020 年之后，SpringCloud 吧 Ribbon 移除了，而是使用自己编写的 LoadBalancer 替代.

## 负载策略

LoadBalancer默认提供了两种负载均衡策略：

- RandomLoadBalancer - 随机分配策略
- **(默认)** RoundRobinLoadBalancer - 轮询分配策略

这里提供更改为随机分配策略的配置代码。

```java
 // 不需要 @Configuration 
  public class LoadBalancerConfig {
      //将官方提供的 RandomLoadBalancer 注册为Bean
      @Bean
      public ReactorLoadBalancer<ServiceInstance> randomLoadBalancer(Environment environment, LoadBalancerClientFactory loadBalancerClientFactory){
          String name = environment.getProperty(LoadBalancerClientFactory.PROPERTY_NAME);
          return new RandomLoadBalancer(loadBalancerClientFactory.getLazyProvider(name, ServiceInstanceListSupplier.class), name);
      }
  }
```

@LoadBalancerClient(value = "服务名", configuration = LoadBalancerConfig.class)  指定负载均衡策略为随机。

```java
@FeignClient("article")
@LoadBalancerClient(value = "article", configuration = LoadBalancerConfig.class) //指定负载均衡策略为随机
public interface ArticleClient {
 
    // @LoadBalanced(可以写，也可以不用写，默认所有方法都自动加 @LoadBalanced)
    @GetMapping("/article/publish")
    String articlePublish();
 
}
```

之所以不用加 `@Configuration`，可以查看一下 `@LoadBalancerClient` 源码说明

```java
/**
 * Declarative configuration for a load balancer client. Add this annotation to any
 * <code>@Configuration</code> and then inject a {@link LoadBalancerClientFactory} to
 * access the client that is created.
 *
 * @author Dave Syer
 */
@Configuration(proxyBeanMethods = false)
@Import(LoadBalancerClientConfigurationRegistrar.class)
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface LoadBalancerClient {

	/**
	 * Synonym for name (the name of the client).
	 *
	 * @see #name()
	 * @return the name of the load balancer client
	 */
	@AliasFor("name")
	String value() default "";

	/**
	 * The name of the load balancer client, uniquely identifying a set of client
	 * resources, including a load balancer.
	 * @return the name of the load balancer client
	 */
	@AliasFor("value")
	String name() default "";

	/**
	 * A custom <code>@Configuration</code> for the load balancer client. Can contain
	 * override <code>@Bean</code> definition for the pieces that make up the client.
	 *
	 * @see LoadBalancerClientConfiguration for the defaults
	 * @return configuration classes for the load balancer client.
	 */
	Class<?>[] configuration() default {};

}
```

## 底层实现

### @LoadBalanced 注解拦截

添加了 @LoadBalanced 注解之后，会启用拦截器对我们发起的服务调用请求进行拦截（注意，这里是针对我们发起的请求进行拦截）

```java
@FunctionalInterface
public interface ClientHttpRequestInterceptor {
    ClientHttpResponse intercept(HttpRequest request, byte[] body, ClientHttpRequestExecution execution) throws IOException;
}
```

**LoadBalancerInterceptor.intercept** 实现 上面接口

```java
public class LoadBalancerInterceptor implements ClientHttpRequestInterceptor {
    private LoadBalancerClient loadBalancer;
    private LoadBalancerRequestFactory requestFactory;

    public LoadBalancerInterceptor(LoadBalancerClient loadBalancer, LoadBalancerRequestFactory requestFactory) {
        this.loadBalancer = loadBalancer;
        this.requestFactory = requestFactory;
    }

    public LoadBalancerInterceptor(LoadBalancerClient loadBalancer) {
        this(loadBalancer, new LoadBalancerRequestFactory(loadBalancer));
    }

    public ClientHttpResponse intercept(final HttpRequest request, final byte[] body, final ClientHttpRequestExecution execution) throws IOException {
        URI originalUri = request.getURI();
        String serviceName = originalUri.getHost();
        Assert.state(serviceName != null, "Request URI does not contain a valid hostname: " + originalUri);
        return (ClientHttpResponse)this.loadBalancer.execute(serviceName, this.requestFactory.createRequest(request, body, execution));
    }
}
```

### 服务选择

```java
/**
 * Implemented by classes which use a load balancer to choose a server to send a request
 * to.
 *
 * @author Ryan Baxter
 * @author Olga Maciaszek-Sharma
 */
public interface ServiceInstanceChooser {

    /**
     * Chooses a ServiceInstance from the LoadBalancer for the specified service.
     * @param serviceId The service ID to look up the LoadBalancer.
     * @return A ServiceInstance that matches the serviceId.
     */
    ServiceInstance choose(String serviceId);

    /**
     * Chooses a ServiceInstance from the LoadBalancer for the specified service and
     * LoadBalancer request.
     * @param serviceId The service ID to look up the LoadBalancer.
     * @param request The request to pass on to the LoadBalancer
     * @param <T> The type of the request context.
     * @return A ServiceInstance that matches the serviceId.
     */
    <T> ServiceInstance choose(String serviceId, Request<T> request);

}
```

```java
    @Override
    public <T> T execute(String serviceId, ServiceInstance serviceInstance, LoadBalancerRequest<T> request)
            throws IOException {
        if (serviceInstance == null) {
            throw new IllegalArgumentException("Service Instance cannot be null");
        }
        DefaultResponse defaultResponse = new DefaultResponse(serviceInstance);
        Set<LoadBalancerLifecycle> supportedLifecycleProcessors = getSupportedLifecycleProcessors(serviceId);
        Request lbRequest = request instanceof Request ? (Request) request : new DefaultRequest<>();
        supportedLifecycleProcessors
                .forEach(lifecycle -> lifecycle.onStartRequest(lbRequest, new DefaultResponse(serviceInstance)));
        try {
            T response = request.apply(serviceInstance);
            LoadBalancerProperties properties = loadBalancerClientFactory.getProperties(serviceId);
            Object clientResponse = getClientResponse(response, properties.isUseRawStatusCodeInResponseData());
            supportedLifecycleProcessors
                    .forEach(lifecycle -> lifecycle.onComplete(new CompletionContext<>(CompletionContext.Status.SUCCESS,
                            lbRequest, defaultResponse, clientResponse)));
            return response;
        }
        catch (IOException iOException) {
            supportedLifecycleProcessors.forEach(lifecycle -> lifecycle.onComplete(
                    new CompletionContext<>(CompletionContext.Status.FAILED, iOException, lbRequest, defaultResponse)));
            throw iOException;
        }
        catch (Exception exception) {
            supportedLifecycleProcessors.forEach(lifecycle -> lifecycle.onComplete(
                    new CompletionContext<>(CompletionContext.Status.FAILED, exception, lbRequest, defaultResponse)));
            ReflectionUtils.rethrowRuntimeException(exception);
        }
        return null;
    }
```

# 异常

## 依赖提示 no feign client for loadbalance defined.

这就是在告诉你 **LoadBalancing是未定义的（OpenFeign 中引入的依赖会使用 LoadBalancing），然后问你是不是忘记加入 spring-cloud-starter-loadbalancer 依赖.**
