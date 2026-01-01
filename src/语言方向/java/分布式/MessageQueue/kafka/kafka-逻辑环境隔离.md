你现在需要为 Kafka 集群创建开发（dev）和生产（prod）两个环境的专属用户，并为它们配置严格的权限隔离，确保 dev 用户只能操作开发环境的资源，prod 用户只能操作生产环境的资源。

### 一、前置条件
1. 确保你的 Kafka 集群已开启 **ACL 权限控制**（这是配置用户权限的前提）：
   - 修改 Kafka 配置文件 `server.properties`，添加以下配置并重启集群：
     ```properties
     # 开启ACL授权
     authorizer.class.name=kafka.security.authorizer.AclAuthorizer
     # 超级用户（拥有所有权限，用于运维）
     super.users=User:admin
     # 没有匹配ACL规则时，默认拒绝访问（安全最佳实践）
     allow.everyone.if.no.acl.found=false
     ```
2. 确保使用的 Kafka 版本支持 ACL（Kafka 0.11.0.0 及以上版本均支持）。

### 二、创建用户并配置权限（完整实操步骤）
Kafka 的 ACL 权限控制依赖「用户标识」，通常有两种方式定义用户：
- **简单方式**：基于客户端连接的「用户名」（适合测试/非生产环境）；
- **安全方式**：基于 SSL 证书/ SASL 认证（适合生产环境）。

下面先提供通用的「用户名+ACL」配置步骤，再补充生产环境的 SASL 认证配置。

#### 1. 基础版：基于用户名配置 ACL 权限
##### （1）创建开发环境（dev）用户权限
```bash
# 1. 授权dev用户：允许读写所有dev-前缀的主题
kafka-acls.sh --bootstrap-server <kafka集群地址>:9092 \
  --add --allow-principal User:dev \
  --operation READ,WRITE,CREATE --topic 'dev-*'

# 2. 授权dev用户：允许使用所有dev-前缀的消费者组
kafka-acls.sh --bootstrap-server <kafka集群地址>:9092 \
  --add --allow-principal User:dev \
  --operation READ --group 'dev-*'

# 3. 授权dev用户：允许描述（DESCRIBE）dev-前缀的主题（消费时需要）
kafka-acls.sh --bootstrap-server <kafka集群地址>:9092 \
  --add --allow-principal User:dev \
  --operation DESCRIBE --topic 'dev-*'
```

##### （2）创建生产环境（prod）用户权限
```bash
# 1. 授权prod用户：允许读写所有prod-前缀的主题
kafka-acls.sh --bootstrap-server <kafka集群地址>:9092 \
  --add --allow-principal User:prod \
  --operation READ,WRITE,CREATE --topic 'prod-*'

# 2. 授权prod用户：允许使用所有prod-前缀的消费者组
kafka-acls.sh --bootstrap-server <kafka集群地址>:9092 \
  --add --allow-principal User:prod \
  --operation READ --group 'prod-*'

# 3. 授权prod用户：允许描述（DESCRIBE）prod-前缀的主题
kafka-acls.sh --bootstrap-server <kafka集群地址>:9092 \
  --add --allow-principal User:prod \
  --operation DESCRIBE --topic 'prod-*'
```

##### （3）验证权限配置
```bash
# 查看所有ACL规则
kafka-acls.sh --bootstrap-server <kafka集群地址>:9092 --list

# 查看指定用户（如dev）的ACL规则
kafka-acls.sh --bootstrap-server <kafka集群地址>:9092 --list --principal User:dev
```

#### 2. 生产版：基于 SASL/PLAIN 认证创建用户（安全加固）
基础版的用户名仅做逻辑标识，生产环境建议结合 SASL 认证（用户名+密码），防止未授权客户端冒充用户：

##### （1）配置 Kafka 开启 SASL/PLAIN 认证
1. 修改 `server.properties`，添加 SASL 配置：
   ```properties
   # 开启SASL认证
   listeners=SASL_PLAINTEXT://:9092
   security.inter.broker.protocol=SASL_PLAINTEXT
   sasl.mechanism.inter.broker.protocol=PLAIN
   sasl.enabled.mechanisms=PLAIN
   
   # 指定JAAS配置文件路径
   listener.name.sasl_plaintext.plain.sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required \
     username="admin" \
     password="admin123" \
     user_dev="dev123" \
     user_prod="prod123";
   ```
   - `user_dev="dev123"`：创建 dev 用户，密码 dev123；
   - `user_prod="prod123"`：创建 prod 用户，密码 prod123；
   - `admin/admin123`：超级用户，用于运维。

2. 重启 Kafka 集群使配置生效。

##### （2）客户端连接示例（以 Java 为例）
客户端需配置 SASL 认证信息，才能以指定用户身份访问 Kafka：
```java
import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.ProducerConfig;
import org.apache.kafka.clients.producer.ProducerRecord;
import java.util.Properties;

public class DevProducer {
    public static void main(String[] args) {
        Properties props = new Properties();
        // Kafka集群地址
        props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "<kafka集群地址>:9092");
        // 序列化配置
        props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, "org.apache.kafka.common.serialization.StringSerializer");
        props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, "org.apache.kafka.common.serialization.StringSerializer");
        
        // SASL认证配置（dev用户）
        props.put("security.protocol", "SASL_PLAINTEXT");
        props.put("sasl.mechanism", "PLAIN");
        props.put("sasl.jaas.config", 
            "org.apache.kafka.common.security.plain.PlainLoginModule required " +
            "username=\"dev\" " +
            "password=\"dev123\";");
        
        // 发送消息到dev-前缀的主题（仅dev用户有权限）
        KafkaProducer<String, String> producer = new KafkaProducer<>(props);
        producer.send(new ProducerRecord<>("dev-order", "1", "test order"));
        producer.close();
    }
}
```

### 三、关键权限说明
| 操作（Operation） | 作用                                  | 适用对象       |
|-------------------|---------------------------------------|----------------|
| READ              | 消费消息、读取主题元数据              | 主题/消费者组  |
| WRITE             | 发送消息到主题                        | 主题           |
| CREATE            | 创建主题                              | 主题           |
| DESCRIBE          | 查看主题的配置、分区等信息（消费必备）| 主题           |

### 总结
1. **基础配置**：先开启 Kafka ACL 权限，为 dev/prod 用户分别授权仅能操作对应前缀的主题和消费者组，确保环境隔离；
2. **生产加固**：结合 SASL/PLAIN 认证，为用户设置密码，防止未授权访问，这是生产环境的必备安全措施；
3. **核心原则**：dev 用户只能操作 `dev-*` 资源，prod 用户只能操作 `prod-*` 资源，超级用户（admin）仅用于运维，不对外提供。