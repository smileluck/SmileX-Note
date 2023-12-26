[toc]

---

# 什么是跨域
同源策略是由 Netscape 提出的一个著名的安全策略，它是浏览器最核心也最基本的安全功能，现在所有支持 JavaScript 的浏览器都会使用这个策略。所谓同源是指协议、域名以及端口要相同。

同源策略是基于安全方面的考虑提出来的，这个策略本身没问题，但是我们在实际开发中，由于各种原因又经常有跨域的需求，传统的跨域方案是 JSONP，JSONP 虽然能解决跨域但是有一个很大的局限性，那就是只支持 GET 请求，不支持其他类型的请求，在 RESTful 时代这几乎就没什么用。

# 什么是CORS
CORS，Cross-origin resource sharing是一个 W3C 标准，提供了 Web 服务从不同网域传来沙盒脚本的方法，以避开浏览器的同源策略，这是 JSONP 模式的现代版。
- 域，指的是一个站点，由protocal、host和port三部分组成，其中host可以是域名，也可以是 ip:port。如果没有指明，则是使用protocal的默认端口
- 同源策略， 指的是为了防止 XSS， 浏览器、客户端应该仅请求与当前页面来自同一个域的资源， 请求其他域的资源需要通过验证。



 CORS需要浏览器和服务器同时支持。目前，所有浏览器都支持该功能，IE浏览器不能低于IE10。 

 浏览器一旦发现AJAX请求跨源，就会自动添加一些附加的头信息，有时还会多出一次附加的请求，但用户不会有感觉。 

 实现CORS通信的关键是服务器。只要服务器实现了CORS接口，就可以跨源通信。 



关于CORS可以看一下这篇[文章](https://www.ruanyifeng.com/blog/2016/04/cors.html)



# SpringBoot下3种常见的cors方案
1. @CrossOrigin。针对单个类或者方法。
```java
@CrossOrigin(value = "http://localhost:8081")
@PostMapping("/hello")
public String hello() {
    return "post hello";
}
```



2. WebMvcConfigurer。全局配置

```java 
@Configuration
public class WebConfig implements WebMvcConfigurer {
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**")
                .allowedOrigins("*")
                .allowCredentials(true)
                .allowedMethods("GET", "POST", "PUT", "DELETE", "OPTIONS")
                .maxAge(3600);
    }

    @Override
    public void addFormatters(FormatterRegistry registry) {
        registry.addFormatter(new DateFormatter("yyyy-MM-dd HH:mm:ss"));
    }
}
```
3. 注入CorsFilter。过滤器

```java
@Configuration
public class CorsConfig {

    private CorsConfiguration corsConfig() {
        CorsConfiguration corsConfiguration = new CorsConfiguration();
        corsConfiguration.addAllowedOrigin("*");
        corsConfiguration.addAllowedHeader("*");
        corsConfiguration.addAllowedMethod("*");
        corsConfiguration.setAllowCredentials(true);
        corsConfiguration.setMaxAge(3600L);
        return corsConfiguration;
    }
    @Bean
    public CorsFilter corsFilter() {
        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/**", corsConfig());
        return new CorsFilter(source);
    }
}
```

4. FilterRegistrationBean。也是基于过滤器

# 参考

- https://www.ruanyifeng.com/blog/2016/04/cors.html
- https://blog.51cto.com/u_15080000/2593305
- https://mp.weixin.qq.com/s?__biz=MzI1NDY0MTkzNQ==&mid=2247488989&idx=1&sn=00881de1a77c2e4027ee948164644485&scene=21#wechat_redirect



