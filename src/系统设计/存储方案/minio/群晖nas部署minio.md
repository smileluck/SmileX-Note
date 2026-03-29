# MinIO 群晖 (DSM) 部署完整指南

## 一、整体方案（推荐架构）
### 目标：
- 独立用户：minio
- 独立目录：/volume1/minio
- 数据目录：/volume1/minio/data
- 配置目录：/volume1/minio/.minio

### 原则：
- ✔ 不使用群晖 /var/services/homes
- ✔ 不依赖 DSM 用户体系
- ✔ 避免权限 & fake_home 坑

## 二、完整部署流程
### 1. 创建 minio 用户
```bash
sudo synouser --add minio password "MinIO User" 0 "" 0
```
或（兼容方式）：
```bash
sudo adduser minio
```

### 2. 修改 HOME 目录（关键）
群晖没有 usermod，用手动方式：
```bash
sudo vi /etc/passwd
```
找到：
```
minio:x:xxxx:xxxx::/var/services/homes/minio:/bin/sh
```
改成：
```
minio:x:xxxx:xxxx::/volume1/minio:/bin/sh
```

### 3. 创建目录
```bash
sudo mkdir -p /volume1/minio/data
sudo mkdir -p /volume1/minio/.minio
```

### 4. 设置权限（非常关键）
```bash
sudo chown -R minio:users /volume1/minio
sudo chmod -R 755 /volume1/minio
```
> 群晖默认组是 users

### 5. 安装 MinIO
```bash
wget https://dl.min.io/server/minio/release/linux-amd64/minio
chmod +x minio
sudo mv minio /usr/local/bin/
```
验证：
```bash
minio --version
```

### 6. 启动 MinIO（正确方式）
切换用户：
```bash
su - minio
```
启动：
```bash
export HOME=/volume1/minio

MINIO_ROOT_USER=admin \
MINIO_ROOT_PASSWORD=12345678 \
minio server /volume1/minio/data --console-address ":9001"
```

### 7. 访问
- 控制台：http://NAS_IP:9001
- S3接口：http://NAS_IP:9000

## 三、持久化启动（群晖推荐）
### 用「任务计划」（强烈推荐）
DSM → 控制面板 → 任务计划 → 新建 → 开机启动

脚本内容：
```bash
su - minio -c '
export HOME=/volume1/minio
export MINIO_ROOT_USER=admin
export MINIO_ROOT_PASSWORD=12345678

/usr/local/bin/minio server /volume1/minio/data --console-address ":9001"
'
```

## 四、踩坑总结（重点）
### ❌ 坑1：--config-dir 无效
- 问题：`--config-dir /xxx` 参数失效
- 原因：已被 MinIO 废弃
- 解决：直接用 HOME 环境变量控制

### ❌ 坑2：fake_home_link
- 问题：`/var/services/homes -> @fake_home_link`
- 原因：群晖没启用 homes 服务
- 结果：MinIO 无法创建 .minio
- 解决：`export HOME=/volume1/minio`

### ❌ 坑3：权限错误（最常见）
- 问题：`Unable to write to backend`
- 原因：/volume1/minio 属于 root
- 解决：`sudo chown -R minio:users /volume1/minio`

### ❌ 坑4：没有 usermod
- 问题：`usermod: command not found`
- 原因：群晖精简系统，没有这个命令
- 解决：手动编辑 `vi /etc/passwd`

### ❌ 坑5：HOME 不生效
- 原因：没重新登录 或 shell 没刷新
- 解决：
  - `su - minio`（重新登录）
  - 或 `export HOME=/volume1/minio`

### ❌ 坑6：端口访问失败
- 原因：防火墙 / 绑定地址问题
- 建议：
  - `--address ":9000"`
  - `--console-address ":9001"`

### ❌ 坑7：systemd 不可用
- 问题：`systemctl: command not found`
- 原因：群晖很多机器不支持 systemd
- 解决：用任务计划代替

## 五、最佳实践总结（强烈建议）
### ✅ 推荐结构
```
/volume1/minio/
├── data/       # 数据
└── .minio/     # 配置
```

### ✅ 推荐启动方式
```bash
export HOME=/volume1/minio
minio server /volume1/minio/data
```

### ✅ 推荐权限
- 用户组：`minio:users`

### ❌ 不推荐
- ❌ 用 root 运行 MinIO
- ❌ 用默认 home 目录
- ❌ 依赖 DSM 用户目录

---

### 总结
1. **核心配置**：通过修改 `/etc/passwd` 手动指定 minio 用户的 HOME 目录为 `/volume1/minio`，避免 fake_home 问题；
2. **权限关键**：必须将 `/volume1/minio` 目录的所有者设置为 `minio:users`，否则会出现写入权限错误；
3. **启动方式**：群晖不推荐使用 systemd，优先通过「任务计划」实现开机自启，启动前务必导出 `HOME=/volume1/minio` 环境变量。