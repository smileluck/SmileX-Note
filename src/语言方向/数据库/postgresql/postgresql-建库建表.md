- [新建数据库](#新建数据库)
  - [一、前期规划](#一前期规划)
  - [二、安全配置](#二安全配置)
  - [三、性能与资源控制](#三性能与资源控制)
  - [四、扩展与兼容性](#四扩展与兼容性)
  - [五、备份与维护](#五备份与维护)
  - [六、常见错误规避](#六常见错误规避)
- [新建数据表](#新建数据表)
  - [一、常规表（Regular Table）](#一常规表regular-table)
    - [适用场景：](#适用场景)
    - [创建示例：](#创建示例)
    - [注意事项：](#注意事项)
  - [二、外部表（External Table）](#二外部表external-table)
    - [适用场景：](#适用场景-1)
    - [创建示例（以访问 CSV 文件为例）：](#创建示例以访问-csv-文件为例)
    - [注意事项：](#注意事项-1)
  - [三、分区表（Partitioned Table）](#三分区表partitioned-table)
    - [适用场景：](#适用场景-2)
    - [分区类型及创建示例：](#分区类型及创建示例)
      - [1. 范围分区（按时间分区，适用于日志、订单等）：](#1-范围分区按时间分区适用于日志订单等)
      - [2. 列表分区（按枚举值分区，如按地区）：](#2-列表分区按枚举值分区如按地区)
    - [注意事项：](#注意事项-2)
  - [总结：如何选择？](#总结如何选择)


## 新建数据库
在 PostgreSQL 中新建数据库时，除了字符集等核心配置外，还有一些关键注意事项直接影响数据库的安全性、性能和可维护性，以下是需要重点关注的内容：


### 一、前期规划
1. **命名规范**  
   - 数据库名避免使用特殊字符（如空格、标点）和 PostgreSQL 关键字（如 `user`、`table`），建议使用小写字母 + 下划线（如 `order_db`）。  
   - 名称需体现业务含义（如 `ecommerce_db` 对应电商业务），便于后期管理。

2. **业务隔离**  
   - 不同业务模块建议使用独立数据库（而非同一库中的不同 schema），避免权限混乱和资源竞争（如日志业务与交易业务分离）。  
   - 若需轻度隔离，可在同一库中使用 schema 区分（如 `public` 用于公共表，`user_schema` 用于用户相关表）。


### 二、安全配置
1. **最小权限原则**  
   - 数据库所有者（`OWNER`）使用专用业务账号（如 `shop_app`），而非超级用户 `postgres`。  
   - 新建用户时仅授予必要权限（如 `CONNECT`、`SELECT`、`INSERT`），避免直接赋予 `SUPERUSER` 或 `CREATEDB` 权限。  
     ```sql
     -- 示例：创建受限用户
     CREATE USER app_user WITH PASSWORD 'secure_password';
     GRANT CONNECT ON DATABASE mydb TO app_user;
     ```

2. **密码策略**  
   - 强制用户密码复杂度（长度 ≥ 10，包含大小写、数字和特殊字符）。  
   - 定期通过 `ALTER USER app_user WITH PASSWORD 'new_password';` 更新密码。


### 三、性能与资源控制
1. **连接限制**  
   - 根据服务器配置（内存、CPU）设置合理的 `CONNECTION LIMIT`（如 500 并发），避免连接数过多导致资源耗尽。  
   - 配合连接池工具（如 PgBouncer）使用，减少频繁创建连接的开销。

2. **表空间规划**  
   - 非默认表空间需提前创建，并指定独立的磁盘路径（建议使用高性能存储，如 SSD）：  
     ```sql
     CREATE TABLESPACE fast_space LOCATION '/data/pg_fast';
     -- 建库时指定
     CREATE DATABASE mydb WITH TABLESPACE = fast_space;
     ```
   - 大表（如历史订单表）可单独指定表空间，避免与系统表共享磁盘 I/O。

3. **模板库选择**  
   - 必须使用 `TEMPLATE = template0`，因为 `template1` 可能被预配置了非默认参数（如编码、扩展），导致新建库继承不必要的设置。  
   - 若需自定义模板（如预设扩展、表结构），可基于 `template0` 创建新模板库：  
     ```sql
     CREATE DATABASE my_template WITH TEMPLATE = template0 ENCODING = 'UTF8';
     -- 后续基于自定义模板建库
     CREATE DATABASE new_db WITH TEMPLATE = my_template;
     ```


### 四、扩展与兼容性
1. **必要扩展预安装**  
   - 建库后立即安装常用扩展（如 `pg_trgm` 用于模糊查询，`uuid-ossp` 生成 UUID）：  
     ```sql
     \c mydb  -- 连接到目标库
     CREATE EXTENSION IF NOT EXISTS pg_trgm;
     ```
   - 扩展需在具体数据库中安装，无法跨库共享。

2. **字符集兼容性检查**  
   - 确保应用程序使用的编码与数据库 `ENCODING` 一致（均为 UTF8），避免插入数据时出现 `invalid byte sequence for encoding "UTF8"` 错误。  
   - 若应用使用其他编码（如 GBK），需在连接字符串中指定 `options=-c client_encoding=GBK`，由 PostgreSQL 自动转换。


### 五、备份与维护
1. **初始备份策略**  
   - 建库后立即配置定期备份（如使用 `pg_dump` 做逻辑备份，或 `pg_basebackup` 做物理备份）。  
   - 测试备份恢复流程，确保数据可恢复性。

2. **参数优化**  
   - 根据业务场景调整 `postgresql.conf` 中的核心参数（如 `shared_buffers`、`work_mem`、`max_connections`），避免使用默认值（默认值适合测试环境，不适合生产）。  
   - 示例（适用于 8GB 内存服务器）：  
     ```ini
     shared_buffers = 2GB  # 通常设为物理内存的 1/4
     max_connections = 200
     work_mem = 16MB       # 每个排序/哈希操作的内存
     ```


### 六、常见错误规避
1. **创建后无法修改的参数**  
   - `ENCODING`、`LC_COLLATE`、`LC_CTYPE` 一旦设置无法修改，若需变更只能通过 `pg_dump` 导出数据后重建库。  
   - 建库前务必确认这些参数（可先创建测试库验证排序、字符存储是否符合预期）。

2. **避免使用默认用户**  
   - 禁止直接使用 `postgres` 用户连接业务数据库，防止误操作（如 `DROP DATABASE`）。  
   - 通过 `REVOKE ALL ON DATABASE mydb FROM public;` 回收公共权限，仅授予指定用户访问权。


通过以上注意事项，可确保新建的 PostgreSQL 数据库在安全性、性能和可维护性上符合生产环境要求，同时为后续业务扩展提供良好基础。核心原则：**提前规划、最小权限、参数适配业务、备份优先**。

## 新建数据表

在 PostgreSQL 中，新建表时需根据数据特性和业务场景选择**常规表**、**外部表**或**分区表**，三者的设计目的、适用场景和创建方式有显著区别。以下是具体区分和最佳实践：


### 一、常规表（Regular Table）
**最基础的表类型**，数据直接存储在 PostgreSQL 数据库中，适用于绝大多数普通业务场景。

#### 适用场景：
- 数据量中等（百万级以内），访问频率均匀。
- 无需跨系统访问外部数据，且数据增长可预测。
- 需要事务支持、索引优化、约束校验等 PostgreSQL 核心功能。

#### 创建示例：
```sql
-- 常规表创建（含主键、索引、约束）
CREATE TABLE users (
    id SERIAL PRIMARY KEY,  -- 自增主键
    username VARCHAR(50) NOT NULL UNIQUE,  -- 唯一约束
    email VARCHAR(100) NOT NULL CHECK (email ~* '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$'),  -- 邮箱格式校验
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,  -- 时间戳默认值
    status SMALLINT NOT NULL DEFAULT 1  -- 状态字段（1:正常，0:禁用）
);

-- 为常用查询字段创建索引
CREATE INDEX idx_users_created_at ON users(created_at);
```

#### 注意事项：
- 优先设置主键（`PRIMARY KEY`），确保数据唯一性并提升查询性能。
- 根据查询场景创建合适的索引（如 `WHERE`、`JOIN`、`ORDER BY` 字段），但避免过度索引（影响写入性能）。
- 合理使用约束（`CHECK`、`UNIQUE`、`FOREIGN KEY`）确保数据完整性，但外键约束可能影响高并发写入性能，需权衡。


### 二、外部表（External Table）
**数据存储在数据库外部**（如文件、其他数据库），PostgreSQL 仅维护表结构和访问接口，不存储实际数据。适用于跨系统数据集成或临时数据分析。

#### 适用场景：
- 需访问 CSV/JSON 等文件数据（如日志文件、批量导入数据）。
- 需关联查询其他数据库（如 MySQL、Oracle）的数据（通过 `postgres_fdw` 扩展）。
- 数据无需持久化存储在 PostgreSQL 中，仅需临时查询分析。

#### 创建示例（以访问 CSV 文件为例）：
```sql
-- 1. 启用外部表扩展（file_fdw 用于访问文件，postgres_fdw 用于访问其他PostgreSQL库）
CREATE EXTENSION IF NOT EXISTS file_fdw;

-- 2. 创建外部服务器（指定数据存储位置）
CREATE SERVER csv_server FOREIGN DATA WRAPPER file_fdw;

-- 3. 创建外部表（映射CSV文件结构）
CREATE FOREIGN TABLE external_logs (
    log_time TIMESTAMP,
    level VARCHAR(10),
    message TEXT
)
SERVER csv_server
OPTIONS (
    filename '/data/logs/app.log',  -- 外部文件路径（PostgreSQL进程需有读取权限）
    format 'csv',                   -- 文件格式
    header 'on',                    -- 包含表头
    delimiter ','                   -- 分隔符
);
```

#### 注意事项：
- 外部表**不支持事务、索引、约束**，仅支持查询（`SELECT`），部分场景支持写入（需外部数据源支持）。
- 确保 PostgreSQL 进程对外部文件/数据源有访问权限（如文件权限、网络连通性）。
- 查询性能依赖外部数据源的性能，建议仅用于低频查询或数据分析，不适合核心业务。


### 三、分区表（Partitioned Table）
**将大表按规则拆分为多个子表（分区）**，逻辑上是一张表，物理上存储在多个分区中。适用于超大规模数据（千万级以上），可显著提升查询和维护效率。

#### 适用场景：
- 数据量巨大（如亿级订单、日志数据），单表查询缓慢。
- 数据有明显的分区键（如时间、地区、用户ID），且查询多集中在部分分区（如查询近3个月数据）。
- 需要快速删除历史数据（直接删除分区而非逐行删除）。

#### 分区类型及创建示例：
PostgreSQL 支持**范围分区（RANGE）**、**列表分区（LIST）**、**哈希分区（HASH）**，最常用的是范围分区（如按时间）。

##### 1. 范围分区（按时间分区，适用于日志、订单等）：
```sql
-- 1. 创建主分区表（仅定义结构，不存储数据）
CREATE TABLE order_logs (
    id BIGSERIAL,
    order_id VARCHAR(50) NOT NULL,
    amount NUMERIC(10,2),
    created_at TIMESTAMP WITH TIME ZONE NOT NULL
) PARTITION BY RANGE (created_at);  -- 按created_at字段分区

-- 2. 创建分区（按季度拆分）
-- 2023年Q1分区
CREATE TABLE order_logs_2023_q1 PARTITION OF order_logs
FOR VALUES FROM ('2023-01-01') TO ('2023-04-01');

-- 2023年Q2分区
CREATE TABLE order_logs_2023_q2 PARTITION OF order_logs
FOR VALUES FROM ('2023-04-01') TO ('2023-07-01');

-- 3. 为分区创建索引（建议与主表分区键一致）
CREATE INDEX idx_order_logs_2023_q1_created_at ON order_logs_2023_q1(created_at);
```

##### 2. 列表分区（按枚举值分区，如按地区）：
```sql
-- 按地区分区（华东、华北、华南）
CREATE TABLE user_profiles (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50),
    region VARCHAR(20) NOT NULL  -- 分区键：region
) PARTITION BY LIST (region);

-- 华东分区
CREATE TABLE user_profiles_east PARTITION OF user_profiles
FOR VALUES IN ('shanghai', 'jiangsu', 'zhejiang');

-- 华北分区
CREATE TABLE user_profiles_north PARTITION OF user_profiles
FOR VALUES IN ('beijing', 'tianjin', 'hebei');
```

#### 注意事项：
- **分区键选择**：必须是查询中频繁过滤的字段（如 `WHERE created_at > '2023-01-01'`），否则分区无意义。
- **分区粒度**：不宜过粗（单分区数据量仍过大）或过细（分区数量过多，元数据管理复杂），建议单分区数据量控制在千万级。
- **自动分区**：可通过定时任务（如 `cron`）或触发器自动创建新分区（如每月初创建下月分区），避免数据写入时无对应分区导致错误。
- **约束排除**：确保查询条件包含分区键，PostgreSQL 会自动跳过无关分区（如查询2023-Q1数据时，仅扫描对应分区）。
- **不支持全局索引**：索引需在每个分区单独创建，全局唯一约束需通过触发器或应用层保证。


### 总结：如何选择？
| 表类型   | 核心特点                          | 适用场景                                  | 性能关注点                          |
|----------|-----------------------------------|-------------------------------------------|-----------------------------------|
| 常规表   | 数据存储在库内，支持完整功能      | 中等数据量，普通业务场景                  | 索引优化、约束合理性              |
| 外部表   | 数据在外部，仅维护访问接口        | 跨系统数据集成、临时数据分析              | 外部数据源性能、权限控制          |
| 分区表   | 大表拆分，逻辑统一物理分离        | 超大规模数据，按规则查询/删除频繁         | 分区键选择、分区粒度、自动管理    |

核心原则：**优先使用常规表，数据量超预期时考虑分区表，跨系统访问时使用外部表**。