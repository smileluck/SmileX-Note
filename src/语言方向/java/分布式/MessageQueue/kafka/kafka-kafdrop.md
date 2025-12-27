- [核心特点](#核心特点)
- [适用场景](#适用场景)
- [与其他工具对比](#与其他工具对比)
- [`start-kafdrop.sh` 脚本](#start-kafdropsh-脚本)
- [使用说明](#使用说明)
  - [1. 准备工作](#1-准备工作)
  - [2. 配置修改](#2-配置修改)
  - [3. 赋予执行权限并启动](#3-赋予执行权限并启动)
  - [4. 停止 Kafdrop](#4-停止-kafdrop)
  - [5. 开机自启（可选）](#5-开机自启可选)
- [脚本功能说明](#脚本功能说明)


### 核心特点
1. **轻量易用**  
   基于 Spring Boot 开发，以 JAR 包形式部署，无需复杂配置，启动命令简单，支持单文件运行。

2. **核心功能**  
   - **集群概览**：展示 Kafka 集群节点（Broker）状态、版本、分区分布等。  
   - **Topic 管理**：查看所有 Topic 的分区数、副本数、留存策略，支持浏览消息内容（支持 JSON、文本等格式）。  
   - **消费者组监控**：查看消费组列表、各分区的消费偏移量（Offset）、滞后消息数（Lag），直观定位消费延迟问题。  
   - **消息查询**：按分区、偏移量、时间范围筛选消息，支持消息内容格式化展示（如解析 JSON 结构）。  

3. **兼容性**  
   支持 Kafka 0.11.0 及以上版本，兼容 KRaft 模式（无 Zookeeper）和传统 Zookeeper 模式集群。

4. **扩展性**  
   可通过配置启用 Basic Auth 认证、HTTPS 加密，支持集成 Prometheus 监控指标。


### 适用场景
- **开发调试**：快速验证消息生产/消费是否正常，查看消息格式是否符合预期。  
- **运维监控**：实时监控消费者组滞后情况，排查分区副本异常（如 Leader 不可用）。  
- **教学演示**：直观展示 Kafka 核心概念（Topic、分区、消费组）的运行状态。


### 与其他工具对比
| 工具         | 优势                                  | 劣势                                  |
|--------------|---------------------------------------|---------------------------------------|
| Kafdrop      | 轻量、部署快、专注基础可视化          | 无集群管理（如创建 Topic）、企业级功能弱 |
| Kafka Tool   | 桌面工具，支持消息编辑、偏移量调整    | 需安装客户端，不适合集群共享访问      |
| AKHQ         | 功能更全，支持 Topic 创建、权限管理   | 部署相对复杂，依赖更多组件            |
| Confluent CC | 企业级监控、数据流追踪                | 收费，重量级，适合大规模集群          |


### `start-kafdrop.sh` 脚本
```bash
#!/bin/bash
# Kafdrop 启动脚本
# 依赖：已安装 Java 11+（Kafdrop 运行环境）

# ==================== 配置参数（根据实际环境修改） ====================
# Kafdrop JAR 包路径（请替换为实际路径）
KAFDROP_JAR="/opt/kafdrop/kafdrop-4.0.1.jar"
# Kafka 集群 bootstrap-server 地址（与 Kafka 的 advertised.listeners 一致）
KAFKA_BROKER="47.112.193.109:9092"
# Kafdrop 监听端口（默认 9000，可修改避免冲突）
KAFDROP_PORT=9000
# 日志输出目录
LOG_DIR="/var/log/kafdrop"
# 日志文件保留天数（默认 7 天）
LOG_RETENTION_DAYS=7
# ==================================================================

# 检查 Java 环境
if ! command -v java &> /dev/null; then
    echo "错误：未找到 Java 环境，请先安装 JDK 11 及以上版本。"
    exit 1
fi

# 检查 Kafdrop JAR 包是否存在
if [ ! -f "$KAFDROP_JAR" ]; then
    echo "错误：Kafdrop JAR 包不存在，请检查路径：$KAFDROP_JAR"
    exit 1
fi

# 创建日志目录
mkdir -p "$LOG_DIR"
if [ $? -ne 0 ]; then
    echo "错误：无法创建日志目录 $LOG_DIR，请检查权限。"
    exit 1
fi

# 日志轮转（删除过期日志）
find "$LOG_DIR" -name "kafdrop-*.log" -type f -mtime +$LOG_RETENTION_DAYS -delete

# 启动 Kafdrop
echo "===== 启动 Kafdrop ====="
echo "Kafka 集群地址：$KAFKA_BROKER"
echo "Kafdrop 访问端口：$KAFDROP_PORT"
echo "日志输出路径：$LOG_DIR/kafdrop-$(date +%Y%m%d).log"

nohup java -jar "$KAFDROP_JAR" \
  --server.port="$KAFDROP_PORT" \
  --kafka.brokerConnect="$KAFKA_BROKER" \
  > "$LOG_DIR/kafdrop-$(date +%Y%m%d).log" 2>&1 &

# 检查启动是否成功
sleep 3
if pgrep -f "$KAFDROP_JAR" > /dev/null; then
    echo "Kafdrop 启动成功！访问地址：http://$(hostname -I | awk '{print $1}'):$KAFDROP_PORT"
else
    echo "Kafdrop 启动失败，请查看日志：$LOG_DIR/kafdrop-$(date +%Y%m%d).log"
    exit 1
fi
```


### 使用说明

#### 1. 准备工作
- **下载 Kafdrop JAR**：从 [GitHub  releases](https://github.com/obsidiandynamics/kafdrop/releases) 下载最新版本（如 `kafdrop-4.0.1.jar`），放到 `/opt/kafdrop/` 目录（或修改脚本中的 `KAFDROP_JAR` 路径）。
- **安装 Java**：确保已安装 JDK 11+（可参考前文 JDK 安装步骤）。


#### 2. 配置修改
根据实际环境调整脚本中的核心参数：
- `KAFDROP_JAR`：Kafdrop JAR 包的绝对路径。
- `KAFKA_BROKER`：Kafka 集群的 `advertised.listeners` 地址（如公网 IP:9092）。
- `KAFDROP_PORT`：Kafdrop 自身的 Web 访问端口（默认 9000，若冲突可改为 8080 等）。
- `LOG_DIR`：日志存储目录（建议设为 `/var/log/kafdrop`）。


#### 3. 赋予执行权限并启动
```bash
# 赋予执行权限
chmod +x start-kafdrop.sh

# 启动 Kafdrop
./start-kafdrop.sh
```


#### 4. 停止 Kafdrop
如需停止，可使用以下命令：
```bash
# 查找 Kafdrop 进程并杀死
pgrep -f "kafdrop-.*.jar" | xargs -r kill -9
echo "Kafdrop 已停止"
```


#### 5. 开机自启（可选）
通过 systemd 配置开机自启：
```bash
# 创建服务文件
sudo tee /etc/systemd/system/kafdrop.service << 'EOF'
[Unit]
Description=Kafdrop Kafka Web UI
After=network.target kafka.service  # 确保 Kafka 启动后再启动 Kafdrop

[Service]
User=ubuntu  # 替换为实际用户名
Group=ubuntu
ExecStart=/opt/kafdrop/start-kafdrop.sh  # 脚本绝对路径
Restart=on-failure  # 故障自动重启
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF

# 启用并启动服务
sudo systemctl daemon-reload
sudo systemctl enable kafdrop
sudo systemctl start kafdrop
```


### 脚本功能说明
- **环境检查**：自动验证 Java 环境和 JAR 包是否存在，避免启动失败。
- **日志管理**：按日期生成日志文件，并自动删除超过保留天数的旧日志。
- **参数配置**：集中管理 Kafka 地址、端口等核心参数，方便修改。
- **启动反馈**：输出启动状态和访问地址，失败时提示查看日志。
