- [架构总设计](#架构总设计)
  - [1. 总体架构设计](#1-总体架构设计)
    - [1.1 技术栈选型](#11-技术栈选型)
  - [2. 存储方案设计：JuiceFS + MinIO + TiKV](#2-存储方案设计juicefs--minio--tikv)
  - [3. 数据流转与业务流程](#3-数据流转与业务流程)
    - [3.1 设备采集端上传流程](#31-设备采集端上传流程)
    - [3.2 数据清洗与处理（Spark/Hadoop）](#32-数据清洗与处理sparkhadoop)
    - [3.3 语义流程与 Embedding 索引](#33-语义流程与-embedding-索引)
  - [4. 详细流程表](#4-详细流程表)
  - [5. 架构优势](#5-架构优势)
- [数据清理](#数据清理)
  - [数据清洗的三个触发时机](#数据清洗的三个触发时机)
    - [1. 准实时触发（Event-Driven）](#1-准实时触发event-driven)
    - [2. 定时批处理（Batch Processing - 最主流）](#2-定时批处理batch-processing---最主流)
    - [3. 专家介入清洗（Active Learning）](#3-专家介入清洗active-learning)
  - [结合 JuiceFS 的清洗全生命周期图](#结合-juicefs-的清洗全生命周期图)
  - [为什么这种架构下“清洗”很快？](#为什么这种架构下清洗很快)
  - [数据清理的两种方式](#数据清理的两种方式)
    - [1. Spark 清洗 ROSBag 数据的代码框架](#1-spark-清洗-rosbag-数据的代码框架)
      - [核心清洗逻辑：时空对齐（Spatial-Temporal Alignment）](#核心清洗逻辑时空对齐spatial-temporal-alignment)
    - [2. MinIO 自动触发清洗的机制](#2-minio-自动触发清洗的机制)
      - [触发流程说明：](#触发流程说明)
      - [MinIO 配置示例 (JSON):](#minio-配置示例-json)
    - [3. Embedding 后的语义流程（Milvus/Qdrant）](#3-embedding-后的语义流程milvusqdrant)
      - [语义化流程步骤：](#语义化流程步骤)
    - [架构选型对比表](#架构选型对比表)
    - [总结与建议](#总结与建议)

---
# 架构总设计

## 1. 总体架构设计

本系统采用分层架构设计，确保从端侧采集到模型训练的高效流转。

### 1.1 技术栈选型

* **存储层**：**JuiceFS**（文件系统接口）+ **MinIO**（数据块存储）+ **TiKV**（元数据引擎）。
* **计算层**：**Hadoop/Spark**（大规模离线清洗、特征提取）。
* **向量检索层**：**Milvus**（支持海量向量的高效语义检索）。
* **采集端**：边缘网关（Edge Gateway）+ MQTT/S3-Proxy。

---

## 2. 存储方案设计：JuiceFS + MinIO + TiKV

具身智能数据（如 ROSBag 文件）具有“大文件连续写入、小文件随机读取”的特点。

* **JuiceFS (POSIX/HDFS Interface)**：作为统一存储层，为 Spark 提供 HDFS 接口，为模型训练（PyTorch/TensorFlow）提供 POSIX 挂载，解决海量小文件读取瓶颈。
* **MinIO (Data Persistence)**：作为底层对象存储，存储实际的数据块（Chunks），利用其 S3 协议的通用性。
* **TiKV (Metadata Management)**：具身智能模型训练涉及数亿级文件，TiKV 作为分布式 KV 数据库，负责存储 JuiceFS 的元数据，确保在高并发下的元数据查询性能与强一致性。

---

## 3. 数据流转与业务流程

### 3.1 设备采集端上传流程

1. **数据打包**：机器人/传感器端将原始数据（点云、视频流、IMU、控制指令）封装为 ROSBag 或自定义压缩包。
2. **断点续传**：采集端通过 **S3 SDK** 或 **Edge Proxy** 将数据推送到 MinIO 临时 Bucket，或直接写入挂载的 JuiceFS 边缘节点。
3. **心跳同步**：采集端实时上传设备状态及任务 Metadata 到管理后台。

### 3.2 数据清洗与处理（Spark/Hadoop）

* **数据接入**：Spark 任务通过 JuiceFS 的 HDFS 接口直接读取原始数据。
* **清洗规则**：
* **异常值过滤**：剔除传感器漂移、无效丢包数据。
* **时空对齐**：将视觉（Camera）与触觉（Tactile）、运动指令（Control）进行时间戳对齐。
* **隐私脱敏**：对视频中的人脸、敏感信息进行遮挡。


* **结果回写**：清洗后的高质量数据集（如 TFRecord 或 Parquet）回写至 JuiceFS。

### 3.3 语义流程与 Embedding 索引

为了实现“通过语言指令寻找对应动作片段”，需要进行向量化处理：

1. **特征提取**：利用多模态大模型（如 CLIP）提取清洗后视频帧或点云的特征向量（Embedding）。
2. **向量存储**：将特征向量写入 **Milvus**，同时在 Milvus 的 `Scalar Field` 中存储指向 JuiceFS 原始路径的 URI。
3. **语义检索**：训练或推理阶段，通过文本指令在 Milvus 中进行向量相似度搜索，快速筛选出相关的训练素材。

---

## 4. 详细流程表

| 阶段 | 核心组件 | 关键动作 | 输出结果 |
| --- | --- | --- | --- |
| **数据采集** | 机器人/传感器/S3-Proxy | 原始流打包、加密上传 | MinIO 中的 Raw Data |
| **数据入湖** | JuiceFS (TiKV Meta) | 自动分块归档、目录树构建 | 可挂载的 POSIX 路径 |
| **数据清洗** | Spark + Python SDK | 降噪、时空同步、质量评分 | Cleaned Dataset (JuiceFS) |
| **向量化** | GPU Cluster + Milvus | 特征提取、向量upsert、索引构建 | Semantic Vector Library |
| **模型训练** | PyTorch + JuiceFS | POSIX 零拷贝读取、分布式训练 | AI Model Checkpoints |

---

## 5. 架构优势

1. **高性能读取**：JuiceFS 的客户端缓存机制可将训练样本缓存至本地 SSD，极大降低模型训练时的 I/O 等待。
2. **无限扩展**：TiKV 与 MinIO 均支持线性水平扩展，足以应对 PB 级的具身智能数据。
3. **语义化管理**：通过 Milvus，研发人员可以通过自然语言（如“抓取瓶子的动作”）快速检索并调取存储在 JuiceFS 中的对应视频与指令数据。

---

# 数据清理

## 数据清洗的三个触发时机

### 1. 准实时触发（Event-Driven）

当设备端完成一个文件（如 `.mcap` 或 `.bag`）的上传并关闭文件时，MinIO 可以通过 **Bucket Notification**（Webhook 或 Kafka）触发清洗任务。

* **适用场景**：元数据提取、数据质量初步检查（是否损坏）、小片段截取。
* **流程**：`Upload to MinIO` -> `MinIO Event` -> `Task Gateway` -> `Spark Streaming / Serverless Job`。

### 2. 定时批处理（Batch Processing - 最主流）

这是 Hadoop/Spark 架构最擅长的部分。通常采用“数据湖”经典的 **Medallion Architecture（奖章架构）**：

* **Raw 层 (ODS)**：采集端原始数据直接推入 MinIO。
* **Bronze 层 (清洗发生点)**：Spark 任务定时扫描 JuiceFS 挂载路径下的新文件。
* **工作内容**：
* **降采样**：高频传感器数据减采样。
* **对齐**：将不同频率的 LiDAR 和 Camera 数据按时间戳对齐。
* **过滤**：剔除机器人静止不动、光线过暗或无意义的重复数据。



### 3. 专家介入清洗（Active Learning）

在 Embedding 流程之后，通过 Milvus 发现聚类异常的数据，由算法工程师手动或半自动发起针对性的“精清洗”任务。

---

## 结合 JuiceFS 的清洗全生命周期图

由于你采用了 **JuiceFS + MinIO**，清洗过程中的 I/O 效率会非常高：

| 步骤 | 状态 | 动作描述 | 存储路径 (JuiceFS) |
| --- | --- | --- | --- |
| **Step 1** | **Landing** | 设备端通过 S3 协议或 POSIX 写入数据。 | `/jfs/raw/2026-03-10/` |
| **Step 2** | **Trigger** | 调度系统（如 Airflow/DolphinScheduler）检测到 Raw 目录更新。 | - |
| **Step 3** | **Cleaning** | **Spark 任务** 启动，利用 JuiceFS 的本地缓存加速读取，进行数据脱敏和对齐。 | `/jfs/tmp/cleaning_cache` |
| **Step 4** | **Sink** | 清洗后的 Parquet 或 TFRecord 格式数据写入 Clean 目录。 | `/jfs/processed/robot_v1/` |
| **Step 5** | **Index** | 启动 **Embedding 脚本**，读取 Clean 数据提取向量并存入 Milvus。 | `/jfs/embeddings/` |

---

## 为什么这种架构下“清洗”很快？

1. **零拷贝感官**：Spark 在清洗时，虽然底层是在读 MinIO，但通过 JuiceFS 挂载，它看起来像是在读本地硬盘。
2. **元数据分离**：由于元数据在 **TiKV** 里，Spark 在扫描数百万个原始文件碎片时，不需要频繁请求 MinIO API，响应时间从秒级降至毫秒级。
3. **计算下推**：清洗任务可以直接跑在靠近存储的计算节点上，利用 JuiceFS 的本地缓存（Local Cache）避免重复从云端拉取 Raw 数据。

**针对您的场景：**
如果您的具身智能数据包含大量的 **视频/点云流**，我建议采用 **“按需清洗”** 模式：

* 先将所有数据全量存入 MinIO（作为冷归档）。
* 通过 Spark 提取关键帧并做 Embedding。
* **仅在训练需要时**，根据 Milvus 返回的语义索引，反向调取原始数据进行深度清洗。


## 数据清理的两种方式

---

### 1. Spark 清洗 ROSBag 数据的代码框架

具身智能的核心数据通常是 ROSBag（包含 LiDAR 点云、视频流、IMU、控制指令）。在 Spark 中，我们通常不直接读二进制，而是通过 Python 环境（PySpark）调用专用的解析库。

#### 核心清洗逻辑：时空对齐（Spatial-Temporal Alignment）

具身智能最怕的是：摄像头频率 30Hz，LiDAR 10Hz，机械臂反馈 100Hz。**清洗的首要任务就是把它们对齐到同一个时间轴上。**

```python
# 伪代码示例：基于 PySpark 的分布式清洗
from pyspark.sql import SparkSession
from rosbags.highlevel import AnyReader # 假设使用 rosbags 库解析

def process_bag(bag_binary_data):
    # 1. 将二进制流转为可读取对象
    with AnyReader(bag_binary_data) as reader:
        aligned_data = []
        # 2. 核心逻辑：以某一频率为主轴进行插值或对齐
        # 比如：每 100ms 采样一次全量传感器状态
        for connection, timestamp, rawdata in reader.messages():
            # 进行解包、降噪、坐标系变换 (RT 矩阵运算)
            # ...
            aligned_data.append(processed_record)
    return aligned_data

# Spark 任务入口
spark = SparkSession.builder.appName("EmbodiedDataCleaning").getOrCreate()

# 从 JuiceFS (HDFS 接口) 读取原始 .bag 文件
raw_df = spark.read.format("binaryFile").load("jfs://raw_data/*.bag")

# 分布式并行解析并清洗
clean_df = raw_df.rdd.flatMap(lambda row: process_bag(row.content)).toDF()

# 存为 Parquet 格式，方便后续模型训练快速加载
clean_df.write.partitionBy("robot_id", "date").parquet("jfs://processed_data/")

```

---

### 2. MinIO 自动触发清洗的机制

在企业级架构中，人工手动启任务太低效。我们利用 **MinIO 的 Bucket Notification** 配合 **消息队列** 实现自动化。

#### 触发流程说明：

1. **上传动作**：采集端完成一个大文件的上传。
2. **事件外发**：MinIO 监测到 `s3:ObjectCreated:Put` 事件。
3. **消息队列**：MinIO 将事件推送到 **Kafka** 或 **Redis**。消息包含文件路径（如 `/raw/robot_01/task_001.bag`）。
4. **调度决策**：
* **小任务**：直接触发 Serverless 函数（如 Knative 或 AWS Lambda 风格的容器）进行初步检查。
* **大任务**：**Airflow** 或 **Argo Workflows** 监听 Kafka，当累积到一定量（或定时）后，拉起 Spark 集群进行大规模清洗。



#### MinIO 配置示例 (JSON):

```json
{
    "QueueConfigurations": [
        {
            "Queue": "arn:minio:sqs::PRIMARY:kafka",
            "Events": ["s3:ObjectCreated:*"],
            "Filter": { "Key": { "FilterRules": [{ "Name": "suffix", "Value": ".bag" }] } }
        }
    ]
}

```

---

### 3. Embedding 后的语义流程（Milvus/Qdrant）

清洗完的数据不是直接丢给模型就完了，为了让模型能“听懂人话”去干活，我们需要构建一个**多模态语义索引**。

#### 语义化流程步骤：

1. **特征提取 (Encoder)**：
* 使用视觉模型（如 CLIP, ViT）提取视频帧特征。
* 使用文本模型提取对应的操作指令特征。


2. **向量入库**：
* **Vector**: `[0.12, -0.05, 0.88, ...]` (特征)
* **Scalar (Metadata)**: `{ "jfs_path": "/jfs/proc/v1.parquet", "timestamp": 1741584000, "action": "pick_up_bottle" }`


3. **语义检索 (Search)**：
* **场景 A (数据筛选)**：科研人员搜索“平滑的抓取动作”，系统从 Milvus 找出相似向量，并根据 `jfs_path` 直接从 JuiceFS 挂载点拉取数据送入显卡训练。
* **场景 B (长短期记忆)**：机器人运行中，将当前观察到的画面转为向量，在 Milvus 中检索历史相似场景，学习当时的成功经验。



---

### 架构选型对比表

| 维度 | Milvus (推荐) | Qdrant |
| --- | --- | --- |
| **数据量级** | 10亿级以上，分布式扩展性极强 | 亿级以下表现极其出色，轻量化 |
| **云原生支持** | 完美支持 K8s 部署 | 部署非常简单，适合快速启动 |
| **存储适配** | 支持 S3/MinIO 作为第三方存储 | 主要是本地磁盘存储 |
| **适用场景** | **超大规模具身智能集群数据湖** | 机器人单机或中小型研发团队 |

---

### 总结与建议

这一套架构的核心价值在于：**JuiceFS 解决了存储的“量”与“速”，Spark 解决了数据的“质”，而 Milvus 赋予了数据“语义”。**

**下一步建议：**
你现在的项目处于什么阶段？如果你正在进行**私有化部署**，我可以为你提供一份 **JuiceFS 在 TiKV 模式下的核心配置文件**，或者帮你写一个 **Milvus 的 Collection Schema 设计方案**（专门针对具身智能数据的）。你想先看哪一个？