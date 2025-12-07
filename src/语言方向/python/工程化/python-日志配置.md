## fastapi-日志配置
在 FastAPI 中配置日志模块可以帮助你跟踪应用运行状态、调试问题和监控系统。以下是一个完整的日志配置方案，支持不同环境（开发/生产）的日志输出格式和级别，并集成到 FastAPI 应用中。


### 一、日志配置核心需求
1. **多环境适配**：开发环境输出详细日志（DEBUG 级别），生产环境输出精简日志（INFO/WARNING 级别）。
2. **多输出目标**：同时输出到控制台和日志文件（按日期分割）。
3. **结构化日志**：生产环境使用 JSON 格式日志，便于日志收集和分析（如 ELK 栈）。
4. **集成 FastAPI**：捕获 FastAPI 内部日志、Uvicorn 服务器日志和自定义业务日志。


### 二、完整实现代码
以下是可直接使用的日志配置模块，结合了 `logging` 标准库和 `python-json-logger` 实现 JSON 日志输出。

#### 1. 安装依赖
```bash
pip install python-json-logger  # 用于生产环境的 JSON 格式日志
```


#### 2. 日志配置模块（`core/logging.py`）

    


#### 3. 配置集成（结合环境配置）
将日志配置与之前的环境配置（`settings.py`）结合，根据当前环境自动调整日志行为：


    


#### 4. 在 FastAPI 中使用
在应用入口（`main.py`）中集成日志，并在路由中使用：


    


### 三、关键配置说明

#### 1. 日志级别控制
- 通过 `LOG_LEVEL` 环境变量设置（默认 `INFO`）：
  - 开发环境：建议设为 `DEBUG`（输出所有日志）。
  - 生产环境：建议设为 `INFO` 或 `WARNING`（减少日志量）。

- 级别优先级（从低到高）：`DEBUG < INFO < WARNING < ERROR < CRITICAL`


#### 2. 日志输出格式
- **开发环境**：
  - 控制台：`时间 - 日志名 - 级别 - 消息`（简洁易读）。
  - 文件：额外包含 `模块名:行号`（便于调试）。

- **生产环境**：
  - 所有日志采用 JSON 格式，包含时间、模块、行号等元数据，示例：
    ```json
    {"asctime": "2023-10-01 12:00:00", "name": "fastapi-app", "module": "main", "lineno": 15, "levelname": "INFO", "message": "访问物品: 100"}
    ```


#### 3. 日志文件管理
- 日志文件保存在 `logs/` 目录下，自动创建。
- 按天分割（`TimedRotatingFileHandler`），保留最近 7 天的日志。
- 生产环境中，错误日志单独输出到 `logs/error.log`，便于问题排查。


#### 4. 捕获 Uvicorn 日志
Uvicorn 服务器的日志（如启动信息、请求访问日志）默认不会被 FastAPI 的日志器捕获，可通过以下方式统一管理：

```bash
# 启动时指定 Uvicorn 日志级别和格式
uvicorn main:app --host 0.0.0.0 --port 8000 --log-config logging.conf
```

或在代码中配置 Uvicorn 日志：

```python
import uvicorn
from core.config import settings

if __name__ == "__main__":
    uvicorn.run(
        "main:app",
        host="0.0.0.0",
        port=8000,
        log_level=settings.log_level.lower(),  # 与应用日志级别一致
    )
```


### 四、环境变量配置示例
创建 `.env` 文件控制日志行为：

```ini
# 开发环境配置（.env.dev）
ENV=dev
LOG_LEVEL=DEBUG  # 输出详细日志

# 生产环境配置（.env.prod）
ENV=prod
LOG_LEVEL=INFO   # 只输出重要日志
```


### 五、最佳实践
1. **敏感信息过滤**：日志中避免记录密码、Token 等敏感信息，可通过自定义过滤器实现：
   ```python
   class SensitiveInfoFilter(logging.Filter):
       def filter(self, record):
           # 替换日志消息中的敏感信息
           record.msg = str(record.msg).replace("password", "***")
           return True

   # 添加到日志处理器
   console_handler.addFilter(SensitiveInfoFilter())
   ```

2. **异步日志**：高并发场景下，使用 `QueueHandler` 和 `QueueListener` 实现异步日志，避免阻塞主线程。

3. **日志监控**：生产环境结合 ELK（Elasticsearch + Logstash + Kibana）或 Grafana Loki 进行日志收集和可视化分析。


通过以上配置，你的 FastAPI 应用将拥有灵活、规范的日志系统，既能满足开发调试需求，也能适应生产环境的稳定性和可观测性要求。