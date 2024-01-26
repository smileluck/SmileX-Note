[toc]

---

# 关于配置动态更新不起效果问题

```yaml
spring:
  cloud:
    nacos:
      discovery:
        # 命名空间
        namespace: dev
        # 注册中鼎
        server-addr: 127.0.0.1:8848
      config:
        # 命名空间
        namespace: dev
        # 配置中心地址
        server-addr: 127.0.0.1:8848
        # 配置文件格式
        file-extension: yml
        # 共享配置
        shared-configs:
          - application-${spring.profiles.active}.${spring.cloud.nacos.config.file-extension}
 
 # 换成这种就可以拉取，但是动态更新会异常
#           - ${spring.application.name}-${spring.profiles.active}.${spring.cloud.nacos.config.file-extension}



```

# 关于jmenv.tbsite.net错误

- 问题描述：nacos未配置cluster.conf，直接执行 ./startup ，表示使用集群模式启动，导致异常
- 解决办法：
  - 方法1：使用单机模式启动 ，执行命令 `./startup -m standalone`。
  - 方法2：创建`cluster.conf`（可不配置）。

- 关键错误信息：

  ```shell
  [com.alibaba.nacos.core.cluster.ServerMemberManager]: Constructor threw exception; nested exception is ErrCode:500, ErrMsg:jmenv.tbsite.net
  ```

- 完整错误信息：

  ```shell
  "nacos is starting with cluster"
  
           ,--.
         ,--.'|
     ,--,:  : |                                           Nacos 2.2.2
  ,`--.'`|  ' :                       ,---.               Running in cluster mode, All function modules
  |   :  :  | |                      '   ,'\   .--.--.    Port: 8848
  :   |   \ | :  ,--.--.     ,---.  /   /   | /  /    '   Pid: 6824
  |   : '  '; | /       \   /     \.   ; ,. :|  :  /`./   Console: http://192.168.56.1:8848/nacos/index.html
  '   ' ;.    ;.--.  .-. | /    / ''   | |: :|  :  ;_
  |   | | \   | \__\/: . ..    ' / '   | .; : \  \    `.      https://nacos.io
  '   : |  ; .' ," .--.; |'   ; :__|   :    |  `----.   \
  |   | '`--'  /  /  ,.  |'   | '.'|\   \  /  /  /`--'  /
  '   : |     ;  :   .'   \   :    : `----'  '--'.     /
  ;   |.'     |  ,     .-./\   \  /            `--'---'
  '---'        `--`---'     `----'
  
  2024-01-09 11:24:11,379 INFO The server IP list of Nacos is []
  
  2024-01-09 11:24:12,394 INFO Nacos is starting...
  
  2024-01-09 11:24:13,414 INFO Nacos is starting...
  
  2024-01-09 11:24:14,421 INFO Nacos is starting...
  
  2024-01-09 11:24:15,429 INFO Nacos is starting...
  
  2024-01-09 11:24:16,435 INFO Nacos is starting...
  
  2024-01-09 11:24:17,442 INFO Nacos is starting...
  
  2024-01-09 11:24:18,449 INFO Nacos is starting...
  
  2024-01-09 11:24:19,462 INFO Nacos is starting...
  
  2024-01-09 11:24:20,463 INFO Nacos is starting...
  
  2024-01-09 11:24:21,473 INFO Nacos is starting...
  
  2024-01-09 11:24:21,799 INFO Nacos Log files: D:\dev\nacos-server-2.2.2\nacos\logs
  
  2024-01-09 11:24:21,800 INFO Nacos Log files: D:\dev\nacos-server-2.2.2\nacos\conf
  
  2024-01-09 11:24:21,800 INFO Nacos Log files: D:\dev\nacos-server-2.2.2\nacos\data
  
  2024-01-09 11:24:21,803 ERROR Startup errors :
  
  org.springframework.context.ApplicationContextException: Unable to start web server; nested exception is org.springframework.boot.web.server.WebServerException: Unable to start embedded Tomcat
          at org.springframework.boot.web.servlet.context.ServletWebServerApplicationContext.onRefresh(ServletWebServerApplicationContext.java:163)
          at org.springframework.context.support.AbstractApplicationContext.refresh(AbstractApplicationContext.java:577)
          at org.springframework.boot.web.servlet.context.ServletWebServerApplicationContext.refresh(ServletWebServerApplicationContext.java:145)
          at org.springframework.boot.SpringApplication.refresh(SpringApplication.java:745)
          at org.springframework.boot.SpringApplication.refreshContext(SpringApplication.java:423)
          at org.springframework.boot.SpringApplication.run(SpringApplication.java:307)
          at org.springframework.boot.SpringApplication.run(SpringApplication.java:1317)
          at org.springframework.boot.SpringApplication.run(SpringApplication.java:1306)
          at com.alibaba.nacos.Nacos.main(Nacos.java:35)
          at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
          at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
          at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
          at java.lang.reflect.Method.invoke(Method.java:498)
          at org.springframework.boot.loader.MainMethodRunner.run(MainMethodRunner.java:49)
          at org.springframework.boot.loader.Launcher.launch(Launcher.java:108)
          at org.springframework.boot.loader.Launcher.launch(Launcher.java:58)
          at org.springframework.boot.loader.PropertiesLauncher.main(PropertiesLauncher.java:467)
  Caused by: org.springframework.boot.web.server.WebServerException: Unable to start embedded Tomcat
          at org.springframework.boot.web.embedded.tomcat.TomcatWebServer.initialize(TomcatWebServer.java:142)
          at org.springframework.boot.web.embedded.tomcat.TomcatWebServer.<init>(TomcatWebServer.java:104)
          at org.springframework.boot.web.embedded.tomcat.TomcatServletWebServerFactory.getTomcatWebServer(TomcatServletWebServerFactory.java:479)
          at org.springframework.boot.web.embedded.tomcat.TomcatServletWebServerFactory.getWebServer(TomcatServletWebServerFactory.java:211)
          at org.springframework.boot.web.servlet.context.ServletWebServerApplicationContext.createWebServer(ServletWebServerApplicationContext.java:182)
          at org.springframework.boot.web.servlet.context.ServletWebServerApplicationContext.onRefresh(ServletWebServerApplicationContext.java:160)
          ... 16 common frames omitted
  Caused by: org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'distroFilterRegistration' defined in class path resource [com/alibaba/nacos/naming/web/NamingConfig.class]: Bean instantiation via factory method failed; nested exception is org.springframework.beans.BeanInstantiationException: Failed to instantiate [org.springframework.boot.web.servlet.FilterRegistrationBean]: Factory method 'distroFilterRegistration' threw exception; nested exception is org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'distroFilter': Unsatisfied dependency expressed through field 'distroMapper'; nested exception is org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'distroMapper' defined in URL [jar:file:/D:/dev/nacos-server-2.2.2/nacos/target/nacos-server.jar!/BOOT-INF/lib/nacos-naming-2.2.2.jar!/com/alibaba/nacos/naming/core/DistroMapper.class]: Unsatisfied dependency expressed through constructor parameter 0; nested exception is org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'serverMemberManager' defined in URL [jar:file:/D:/dev/nacos-server-2.2.2/nacos/target/nacos-server.jar!/BOOT-INF/lib/nacos-core-2.2.2.jar!/com/alibaba/nacos/core/cluster/ServerMemberManager.class]: Bean instantiation via constructor failed; nested exception is org.springframework.beans.BeanInstantiationException: Failed to instantiate [com.alibaba.nacos.core.cluster.ServerMemberManager]: Constructor threw exception; nested exception is ErrCode:500, ErrMsg:jmenv.tbsite.net
          at org.springframework.beans.factory.support.ConstructorResolver.instantiate(ConstructorResolver.java:658)
          at org.springframework.beans.factory.support.ConstructorResolver.instantiateUsingFactoryMethod(ConstructorResolver.java:486)
          at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.instantiateUsingFactoryMethod(AbstractAutowireCapableBeanFactory.java:1352)
          at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBeanInstance(AbstractAutowireCapableBeanFactory.java:1195)
          at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.doCreateBean(AbstractAutowireCapableBeanFactory.java:582)
          at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBean(AbstractAutowireCapableBeanFactory.java:542)
          at org.springframework.beans.factory.support.AbstractBeanFactory.lambda$doGetBean$0(AbstractBeanFactory.java:335)
          at org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.getSingleton(DefaultSingletonBeanRegistry.java:234)
          at org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:333)
          at org.springframework.beans.factory.support.AbstractBeanFactory.getBean(AbstractBeanFactory.java:213)
          at org.springframework.boot.web.servlet.ServletContextInitializerBeans.getOrderedBeansOfType(ServletContextInitializerBeans.java:212)
          at org.springframework.boot.web.servlet.ServletContextInitializerBeans.getOrderedBeansOfType(ServletContextInitializerBeans.java:203)
          at org.springframework.boot.web.servlet.ServletContextInitializerBeans.addServletContextInitializerBeans(ServletContextInitializerBeans.java:97)
          at org.springframework.boot.web.servlet.ServletContextInitializerBeans.<init>(ServletContextInitializerBeans.java:86)
          at org.springframework.boot.web.servlet.context.ServletWebServerApplicationContext.getServletContextInitializerBeans(ServletWebServerApplicationContext.java:260)
          at org.springframework.boot.web.servlet.context.ServletWebServerApplicationContext.selfInitialize(ServletWebServerApplicationContext.java:234)
          at org.springframework.boot.web.embedded.tomcat.TomcatStarter.onStartup(TomcatStarter.java:53)
          at org.apache.catalina.core.StandardContext.startInternal(StandardContext.java:5211)
          at org.apache.catalina.util.LifecycleBase.start(LifecycleBase.java:183)
          at org.apache.catalina.core.ContainerBase$StartChild.call(ContainerBase.java:1393)
          at org.apache.catalina.core.ContainerBase$StartChild.call(ContainerBase.java:1383)
          at java.util.concurrent.FutureTask.run(FutureTask.java:266)
          at org.apache.tomcat.util.threads.InlineExecutorService.execute(InlineExecutorService.java:75)
          at java.util.concurrent.AbstractExecutorService.submit(AbstractExecutorService.java:134)
          at org.apache.catalina.core.ContainerBase.startInternal(ContainerBase.java:916)
          at org.apache.catalina.core.StandardHost.startInternal(StandardHost.java:835)
          at org.apache.catalina.util.LifecycleBase.start(LifecycleBase.java:183)
          at org.apache.catalina.core.ContainerBase$StartChild.call(ContainerBase.java:1393)
          at org.apache.catalina.core.ContainerBase$StartChild.call(ContainerBase.java:1383)
          at java.util.concurrent.FutureTask.run(FutureTask.java:266)
          at org.apache.tomcat.util.threads.InlineExecutorService.execute(InlineExecutorService.java:75)
          at java.util.concurrent.AbstractExecutorService.submit(AbstractExecutorService.java:134)
          at org.apache.catalina.core.ContainerBase.startInternal(ContainerBase.java:916)
          at org.apache.catalina.core.StandardEngine.startInternal(StandardEngine.java:265)
          at org.apache.catalina.util.LifecycleBase.start(LifecycleBase.java:183)
          at org.apache.catalina.core.StandardService.startInternal(StandardService.java:430)
          at org.apache.catalina.util.LifecycleBase.start(LifecycleBase.java:183)
          at org.apache.catalina.core.StandardServer.startInternal(StandardServer.java:930)
          at org.apache.catalina.util.LifecycleBase.start(LifecycleBase.java:183)
          at org.apache.catalina.startup.Tomcat.start(Tomcat.java:486)
          at org.springframework.boot.web.embedded.tomcat.TomcatWebServer.initialize(TomcatWebServer.java:123)
          ... 21 common frames omitted
  Caused by: org.springframework.beans.BeanInstantiationException: Failed to instantiate [org.springframework.boot.web.servlet.FilterRegistrationBean]: Factory method 'distroFilterRegistration' threw exception; nested exception is org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'distroFilter': Unsatisfied dependency expressed through field 'distroMapper'; nested exception is org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'distroMapper' defined in URL [jar:file:/D:/dev/nacos-server-2.2.2/nacos/target/nacos-server.jar!/BOOT-INF/lib/nacos-naming-2.2.2.jar!/com/alibaba/nacos/naming/core/DistroMapper.class]: Unsatisfied dependency expressed through constructor parameter 0; nested exception is org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'serverMemberManager' defined in URL [jar:file:/D:/dev/nacos-server-2.2.2/nacos/target/nacos-server.jar!/BOOT-INF/lib/nacos-core-2.2.2.jar!/com/alibaba/nacos/core/cluster/ServerMemberManager.class]: Bean instantiation via constructor failed; nested exception is org.springframework.beans.BeanInstantiationException: Failed to instantiate [com.alibaba.nacos.core.cluster.ServerMemberManager]: Constructor threw exception; nested exception is ErrCode:500, ErrMsg:jmenv.tbsite.net
          at org.springframework.beans.factory.support.SimpleInstantiationStrategy.instantiate(SimpleInstantiationStrategy.java:185)
          at org.springframework.beans.factory.support.ConstructorResolver.instantiate(ConstructorResolver.java:653)
          ... 61 common frames omitted
  Caused by: org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'distroFilter': Unsatisfied dependency expressed through field 'distroMapper'; nested exception is org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'distroMapper' defined in URL [jar:file:/D:/dev/nacos-server-2.2.2/nacos/target/nacos-server.jar!/BOOT-INF/lib/nacos-naming-2.2.2.jar!/com/alibaba/nacos/naming/core/DistroMapper.class]: Unsatisfied dependency expressed through constructor parameter 0; nested exception is org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'serverMemberManager' defined in URL [jar:file:/D:/dev/nacos-server-2.2.2/nacos/target/nacos-server.jar!/BOOT-INF/lib/nacos-core-2.2.2.jar!/com/alibaba/nacos/core/cluster/ServerMemberManager.class]: Bean instantiation via constructor failed; nested exception is org.springframework.beans.BeanInstantiationException: Failed to instantiate [com.alibaba.nacos.core.cluster.ServerMemberManager]: Constructor threw exception; nested exception is ErrCode:500, ErrMsg:jmenv.tbsite.net
          at org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor$AutowiredFieldElement.resolveFieldValue(AutowiredAnnotationBeanPostProcessor.java:660)
          at org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor$AutowiredFieldElement.inject(AutowiredAnnotationBeanPostProcessor.java:640)
          at org.springframework.beans.factory.annotation.InjectionMetadata.inject(InjectionMetadata.java:119)
          at org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor.postProcessProperties(AutowiredAnnotationBeanPostProcessor.java:399)
          at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.populateBean(AbstractAutowireCapableBeanFactory.java:1431)
          at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.doCreateBean(AbstractAutowireCapableBeanFactory.java:619)
          at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBean(AbstractAutowireCapableBeanFactory.java:542)
          at org.springframework.beans.factory.support.AbstractBeanFactory.lambda$doGetBean$0(AbstractBeanFactory.java:335)
          at org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.getSingleton(DefaultSingletonBeanRegistry.java:234)
          at org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:333)
          at org.springframework.beans.factory.support.AbstractBeanFactory.getBean(AbstractBeanFactory.java:208)
          at org.springframework.context.annotation.ConfigurationClassEnhancer$BeanMethodInterceptor.resolveBeanReference(ConfigurationClassEnhancer.java:362)
          at org.springframework.context.annotation.ConfigurationClassEnhancer$BeanMethodInterceptor.intercept(ConfigurationClassEnhancer.java:334)
          at com.alibaba.nacos.naming.web.NamingConfig$$EnhancerBySpringCGLIB$$78c17639.distroFilter(<generated>)
          at com.alibaba.nacos.naming.web.NamingConfig.distroFilterRegistration(NamingConfig.java:44)
          at com.alibaba.nacos.naming.web.NamingConfig$$EnhancerBySpringCGLIB$$78c17639.CGLIB$distroFilterRegistration$3(<generated>)
          at com.alibaba.nacos.naming.web.NamingConfig$$EnhancerBySpringCGLIB$$78c17639$$FastClassBySpringCGLIB$$886156dc.invoke(<generated>)
          at org.springframework.cglib.proxy.MethodProxy.invokeSuper(MethodProxy.java:244)
          at org.springframework.context.annotation.ConfigurationClassEnhancer$BeanMethodInterceptor.intercept(ConfigurationClassEnhancer.java:331)
          at com.alibaba.nacos.naming.web.NamingConfig$$EnhancerBySpringCGLIB$$78c17639.distroFilterRegistration(<generated>)
          at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
          at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
          at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
          at java.lang.reflect.Method.invoke(Method.java:498)
          at org.springframework.beans.factory.support.SimpleInstantiationStrategy.instantiate(SimpleInstantiationStrategy.java:154)
          ... 62 common frames omitted
  Caused by: org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'distroMapper' defined in URL [jar:file:/D:/dev/nacos-server-2.2.2/nacos/target/nacos-server.jar!/BOOT-INF/lib/nacos-naming-2.2.2.jar!/com/alibaba/nacos/naming/core/DistroMapper.class]: Unsatisfied dependency expressed through constructor parameter 0; nested exception is org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'serverMemberManager' defined in URL [jar:file:/D:/dev/nacos-server-2.2.2/nacos/target/nacos-server.jar!/BOOT-INF/lib/nacos-core-2.2.2.jar!/com/alibaba/nacos/core/cluster/ServerMemberManager.class]: Bean instantiation via constructor failed; nested exception is org.springframework.beans.BeanInstantiationException: Failed to instantiate [com.alibaba.nacos.core.cluster.ServerMemberManager]: Constructor threw exception; nested exception is ErrCode:500, ErrMsg:jmenv.tbsite.net
          at org.springframework.beans.factory.support.ConstructorResolver.createArgumentArray(ConstructorResolver.java:800)
          at org.springframework.beans.factory.support.ConstructorResolver.autowireConstructor(ConstructorResolver.java:229)
          at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.autowireConstructor(AbstractAutowireCapableBeanFactory.java:1372)
          at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBeanInstance(AbstractAutowireCapableBeanFactory.java:1222)
          at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.doCreateBean(AbstractAutowireCapableBeanFactory.java:582)
          at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBean(AbstractAutowireCapableBeanFactory.java:542)
          at org.springframework.beans.factory.support.AbstractBeanFactory.lambda$doGetBean$0(AbstractBeanFactory.java:335)
          at org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.getSingleton(DefaultSingletonBeanRegistry.java:234)
          at org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:333)
          at org.springframework.beans.factory.support.AbstractBeanFactory.getBean(AbstractBeanFactory.java:208)
          at org.springframework.beans.factory.config.DependencyDescriptor.resolveCandidate(DependencyDescriptor.java:276)
          at org.springframework.beans.factory.support.DefaultListableBeanFactory.doResolveDependency(DefaultListableBeanFactory.java:1391)
          at org.springframework.beans.factory.support.DefaultListableBeanFactory.resolveDependency(DefaultListableBeanFactory.java:1311)
          at org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor$AutowiredFieldElement.resolveFieldValue(AutowiredAnnotationBeanPostProcessor.java:657)
          ... 86 common frames omitted
  Caused by: org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'serverMemberManager' defined in URL [jar:file:/D:/dev/nacos-server-2.2.2/nacos/target/nacos-server.jar!/BOOT-INF/lib/nacos-core-2.2.2.jar!/com/alibaba/nacos/core/cluster/ServerMemberManager.class]: Bean instantiation via constructor failed; nested exception is org.springframework.beans.BeanInstantiationException: Failed to instantiate [com.alibaba.nacos.core.cluster.ServerMemberManager]: Constructor threw exception; nested exception is ErrCode:500, ErrMsg:jmenv.tbsite.net
          at org.springframework.beans.factory.support.ConstructorResolver.instantiate(ConstructorResolver.java:315)
          at org.springframework.beans.factory.support.ConstructorResolver.autowireConstructor(ConstructorResolver.java:296)
          at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.autowireConstructor(AbstractAutowireCapableBeanFactory.java:1372)
          at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBeanInstance(AbstractAutowireCapableBeanFactory.java:1222)
          at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.doCreateBean(AbstractAutowireCapableBeanFactory.java:582)
          at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBean(AbstractAutowireCapableBeanFactory.java:542)
          at org.springframework.beans.factory.support.AbstractBeanFactory.lambda$doGetBean$0(AbstractBeanFactory.java:335)
          at org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.getSingleton(DefaultSingletonBeanRegistry.java:234)
          at org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:333)
          at org.springframework.beans.factory.support.AbstractBeanFactory.getBean(AbstractBeanFactory.java:208)
          at org.springframework.beans.factory.config.DependencyDescriptor.resolveCandidate(DependencyDescriptor.java:276)
          at org.springframework.beans.factory.support.DefaultListableBeanFactory.doResolveDependency(DefaultListableBeanFactory.java:1391)
          at org.springframework.beans.factory.support.DefaultListableBeanFactory.resolveDependency(DefaultListableBeanFactory.java:1311)
          at org.springframework.beans.factory.support.ConstructorResolver.resolveAutowiredArgument(ConstructorResolver.java:887)
          at org.springframework.beans.factory.support.ConstructorResolver.createArgumentArray(ConstructorResolver.java:791)
          ... 99 common frames omitted
  Caused by: org.springframework.beans.BeanInstantiationException: Failed to instantiate [com.alibaba.nacos.core.cluster.ServerMemberManager]: Constructor threw exception; nested exception is ErrCode:500, ErrMsg:jmenv.tbsite.net
          at org.springframework.beans.BeanUtils.instantiateClass(BeanUtils.java:224)
          at org.springframework.beans.factory.support.SimpleInstantiationStrategy.instantiate(SimpleInstantiationStrategy.java:117)
          at org.springframework.beans.factory.support.ConstructorResolver.instantiate(ConstructorResolver.java:311)
          ... 113 common frames omitted
  Caused by: com.alibaba.nacos.api.exception.NacosException: java.net.UnknownHostException: jmenv.tbsite.net
          at com.alibaba.nacos.core.cluster.lookup.AddressServerMemberLookup.run(AddressServerMemberLookup.java:152)
          at com.alibaba.nacos.core.cluster.lookup.AddressServerMemberLookup.doStart(AddressServerMemberLookup.java:100)
          at com.alibaba.nacos.core.cluster.AbstractMemberLookup.start(AbstractMemberLookup.java:55)
          at com.alibaba.nacos.core.cluster.ServerMemberManager.initAndStartLookup(ServerMemberManager.java:224)
          at com.alibaba.nacos.core.cluster.ServerMemberManager.init(ServerMemberManager.java:171)
          at com.alibaba.nacos.core.cluster.ServerMemberManager.<init>(ServerMemberManager.java:152)
          at sun.reflect.NativeConstructorAccessorImpl.newInstance0(Native Method)
          at sun.reflect.NativeConstructorAccessorImpl.newInstance(NativeConstructorAccessorImpl.java:62)
          at sun.reflect.DelegatingConstructorAccessorImpl.newInstance(DelegatingConstructorAccessorImpl.java:45)
          at java.lang.reflect.Constructor.newInstance(Constructor.java:423)
          at org.springframework.beans.BeanUtils.instantiateClass(BeanUtils.java:211)
          ... 115 common frames omitted
  Caused by: java.net.UnknownHostException: jmenv.tbsite.net
          at java.net.AbstractPlainSocketImpl.connect(AbstractPlainSocketImpl.java:184)
          at java.net.PlainSocketImpl.connect(PlainSocketImpl.java:172)
          at java.net.SocksSocketImpl.connect(SocksSocketImpl.java:392)
          at java.net.Socket.connect(Socket.java:589)
          at sun.net.NetworkClient.doConnect(NetworkClient.java:175)
          at sun.net.www.http.HttpClient.openServer(HttpClient.java:463)
          at sun.net.www.http.HttpClient.openServer(HttpClient.java:558)
          at sun.net.www.http.HttpClient.<init>(HttpClient.java:242)
          at sun.net.www.http.HttpClient.New(HttpClient.java:339)
          at sun.net.www.http.HttpClient.New(HttpClient.java:357)
          at sun.net.www.protocol.http.HttpURLConnection.getNewHttpClient(HttpURLConnection.java:1220)
          at sun.net.www.protocol.http.HttpURLConnection.plainConnect0(HttpURLConnection.java:1156)
          at sun.net.www.protocol.http.HttpURLConnection.plainConnect(HttpURLConnection.java:1050)
          at sun.net.www.protocol.http.HttpURLConnection.connect(HttpURLConnection.java:984)
          at com.alibaba.nacos.common.http.client.request.JdkHttpClientRequest.execute(JdkHttpClientRequest.java:114)
          at com.alibaba.nacos.common.http.client.NacosRestTemplate.execute(NacosRestTemplate.java:482)
          at com.alibaba.nacos.common.http.client.NacosRestTemplate.get(NacosRestTemplate.java:72)
          at com.alibaba.nacos.core.cluster.lookup.AddressServerMemberLookup.syncFromAddressUrl(AddressServerMemberLookup.java:175)
          at com.alibaba.nacos.core.cluster.lookup.AddressServerMemberLookup.run(AddressServerMemberLookup.java:143)
          ... 125 common frames omitted
  2024-01-09 11:24:21,806 WARN [WatchFileCenter] start close
  
  2024-01-09 11:24:21,806 WARN [WatchFileCenter] start to shutdown this watcher which is watch : D:\dev\nacos-server-2.2.2\nacos\conf
  
  2024-01-09 11:24:21,808 WARN [WatchFileCenter] already closed
  
  2024-01-09 11:24:21,808 WARN [NotifyCenter] Start destroying Publisher
  
  2024-01-09 11:24:21,809 WARN [NotifyCenter] Destruction of the end
  
  2024-01-09 11:24:21,810 ERROR Nacos failed to start, please see D:\dev\nacos-server-2.2.2\nacos\logs\nacos.log for more details.
  
  2024-01-09 11:24:21,876 WARN [ThreadPoolManager] Start destroying ThreadPool
  
  2024-01-09 11:24:21,877 WARN [ThreadPoolManager] Destruction of the end
  ```

  