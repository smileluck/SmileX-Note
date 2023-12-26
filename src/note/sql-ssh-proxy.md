[toc]

---

# SpringBoot 配置ssh连接数据库

## 前言

有时候我们无法直连上数据库，这时就需要通过代理的形式。



## 方案1

1. 导入依赖包

```xml
<dependency>
    <groupId>com.jcraft</groupId>
    <artifactId>jsch</artifactId>
    <version>0.1.50</version>
</dependency>
```

2. 添加ssh配置

```yaml
ssh.enabled: true
ssh.remote.ip: 192.168.0.1
ssh.remote.port: 22
ssh.remote.username: root
ssh.remote.password: root
ssh.remote.target_host: 192.168.0.2
ssh.remote.target_port: 3306
ssh.local.resource_host: 127.0.0.1
ssh.local.resource_port: 3307
```

3. properties配置类

```java
@Configuration
@Data
@PropertySource(value = "classpath:/config/remote-ssh.properties", encoding = "utf-8", ignoreResourceNotFound = true)
@ConditionalOnResource(resources = {"classpath:/config/remote-ssh.properties"})
public class SshProperties {
    @Value("${ssh.enabled}")
    private boolean enabled;
    @Value("${ssh.remote.ip}")
    private String ip;
    @Value("${ssh.remote.port}")
    private int port;
    @Value("${ssh.remote.username}")
    private String username;
    @Value("${ssh.remote.password}")
    private String passowrd;
    @Value("${ssh.remote.target_host}")
    private String targetHost;
    @Value("${ssh.remote.target_port}")
    private int targetPort;
    @Value("${ssh.local.resource_host}")
    private String resourceHost;
    @Value("${ssh.local.resource_port}")
    private int resourcePort;
}

// 或者这样的方式
@Configuration
@Data
@PropertySource(value = "classpath:/config/remote-ssh.properties", encoding = "utf-8", ignoreResourceNotFound = true)
@ConditionalOnResource(resources = {"classpath:/config/remote-ssh.properties"})
@ConfigurationProperties(prefix = "ssh")
public class SshProperties {
    //    @Value("${ssh.enabled:false}")
    private boolean enabled;

    @Bean
    public Remote getRemote() {
        return new Remote();
    }

    @Bean
    public Local getLocal() {
        return new Local();
    }

    @Data
    @Configuration
    @ConfigurationProperties("ssh.remote")
    public class Remote {
        //    @Value("${ssh.remote.ip}")
        private String ip;
        //    @Value("${ssh.remote.port}")
        private int port;
        //    @Value("${ssh.remote.username}")
        private String username;
        //    @Value("${ssh.remote.password}")
        private String passowrd;
        //    @Value("${ssh.remote.target_host}")
        private String targetHost;
        //    @Value("${ssh.remote.target_port}")
        private int targetPort;
    }

    @Data
    @Configuration
    @ConfigurationProperties("ssh.local")
    public class Local {
        //    @Value("${ssh.local.resource_host}")
        private String resourceHost;
        //    @Value("${ssh.local.resource_port}")
        private int resourcePort;
    }
}
```

4. 使用jsch加载SshProperties

```java
@Component
@Slf4j
@ConditionalOnBean(SshProperties.class)
public class SshConfig implements ServletContextInitializer {
    private static Session session;

    @Autowired(required = false)
    private SshProperties sshProperties;


    public Session initSession() {
        try {
            //如果配置文件包含ssh.enabled属性，则使用ssh转发
            if (sshProperties.isEnabled()) {
                Session session = new JSch().getSession(sshProperties.getRemote().getUsername(), sshProperties.getRemote().getIp(), sshProperties.getRemote().getPort());
                session.setConfig("StrictHostKeyChecking", "no");
                session.setPassword(sshProperties.getRemote().getPassowrd());
                session.connect();
                //将本地端口的请求转发到目标地址的端口
                session.setPortForwardingL(sshProperties.getLocal().getResourceHost(), sshProperties.getLocal().getResourcePort(), sshProperties.getRemote().getTargetHost(), sshProperties.getRemote().getTargetPort());
                log.info("ssh forward is open.");
                return session;
            } else {
                log.info("ssh forward is closed.");
            }
        } catch (JSchException e) {
            log.error("ssh JSchException failed.", e);
        } catch (Exception e) {
            log.error("ssh settings is failed. skip!", e);
        }
        return null;
    }

    @Override
    public void onStartup(ServletContext servletContext) throws ServletException {
        session = initSession();
    }

    /**
     * 断开SSH连接
     */
    public void destroy() {
        this.session.disconnect();
    }
}
```

到这一步就大功告成了。

## 参考文章

- [SpringBoot 通过SSH跳板机连接不能直达的IP+端口](https://blog.csdn.net/qq_39706515/article/details/122539650?utm_medium=distribute.pc_aggpage_search_result.none-task-blog-2~aggregatepage~first_rank_ecpm_v1~rank_v31_ecpm-1-122539650.pc_agg_new_rank&utm_term=HikariCP+ssh%E9%80%9A%E9%81%93&spm=1000.2123.3001.4430)