[toc]

---

# springboot中使用undertow替换tomcat

## 引入maven

```xml
 <dependency>
     <groupId>org.springframework.boot</groupId>
     <artifactId>spring-boot-starter-web</artifactId>
     <!-- 排除Tomcat容器 -->
     <exclusions>
         <exclusion>
             <groupId>org.springframework.boot</groupId>
             <artifactId>spring-boot-starter-tomcat</artifactId>
         </exclusion>
     </exclusions>
</dependency>
<!-- 引入undertow容器 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-undertow</artifactId>
    <exclusions>
        <exclusion>
            <!-- 排除websocket模块-->
            <groupId>io.undertow</groupId>
            <artifactId>undertow-websockets-jsr</artifactId>
        </exclusion>
    </exclusions>
</dependency>

```

## 基本配置

```properties
# === undertow 日志 ========
# 是否打开 undertow 日志，默认为 false
server.undertow.accesslog.enabled=false
# 设置访问日志所在目录
server.undertow.accesslog.dir=logs
# 日志格式
server.undertow.accesslog.pattern=common
# 日志文件前缀
server.undertow.accesslog.prefix=access_log.
# 日志文件后缀
server.undertow.accesslog.suffix=log

# 指定工作者线程的 I/0 线程数，默认为 2 或者 CPU 的个数
server.undertow.threads.io=2
# 指定工作者线程个数，默认为 I/O 线程个数的 8 倍
server.undertow.threads.worker=8
# 设置 HTTP POST 内容的最大长度，默认不做限制
server.undertow.max-http-post-size=4MB
# 以下的配置会影响buffer,这些buffer会用于服务器连接的IO操作,有点类似netty的池化内存管理；
server.undertow.buffer-size=1024
# 是否分配的直接内存(NIO直接分配的堆外内存)
server.undertow.direct-buffers=true
```

