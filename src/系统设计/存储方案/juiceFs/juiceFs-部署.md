
## 安装

### JuiceFs
```bash
# 安装 JuiceFS
# 下载 (以 v1.3.0 为例，请根据实际情况替换版本号)
wget https://github.com/juicedata/juicefs/releases/download/v1.3.0/juicefs-1.3.0-linux-amd64.tar.gz

# 解压
tar -zxvf juicefs-1.3.0-linux-amd64.tar.gz

# 移动到系统路径
sudo install juicefs /usr/local/bin

# FUSE 安装ji # Ubuntu 22.04+ 建议安装 libfuse2
```

### HAProxy（可选）
1. 安装
    ```bash
    # 安装 HAProxy
    sudo apt install haproxy
    ```
2. 配置文件
   ```bash
    # 配置 HAProxy
    sudo nano /etc/haproxy/haproxy.cfg
    ```
3. 文件内容
   ```
    listen tikv_pd
        bind *:6241          # HAProxy 对外暴露的端口，避免与本地 PD 冲突    
        mode tcp              # PD 通讯必须使用 TCP 模式 
        balance roundrobin    # 轮询负载均衡
        option tcplog 
        # 检查 PD 节点是否存活     
        server pd1 192.168.112.103:6200 check
        server pd2 192.168.112.103:6202 check
        server pd3 192.168.112.103:6204 check        

    # 可选：配置监控页面，方便查看 PD 挂载状态     

    listen stats  
        bind *:6242   
        mode http 
        stats enable
        stats uri /stats
        stats refresh 10s
   ```
4. 加载配置文件
   ```bash
    # 加载配置文件
    sudo haproxy -c -f /etc/haproxy/haproxy.cfg
   ```
### KeepAlived(可选搭配HAProxy使用)  

> TODO 配置 KeepAlived


## 常用指令

### 基本命令

- 查询状态：```juicefs status tikv://<PD_ADDR1>:2379/jfs_metadata```
- 查看挂载点：```mount | grep /mnt/jfs```
- 

### s3 gateway



### 格式化文件系统
```bash
juicefs format \
    --storage s3 \
    --bucket http://<minio-endpoint>:<port>/<bucket-name> \
    --access-key <your-minio-ak> \
    --secret-key <your-minio-sk> \
    tikv://root:password@<tikv-pd-endpoint>:2379/jfs \
    my-jfs-name

# 示例

juicefs format \
    --storage s3 \
    --bucket http://192.168.112.200:9000/platform \
    --access-key admin \
    --secret-key 123456 \
    tikv://root:password@192.168.112.103:6241/jfs \
    jfs
```

### 挂载文件系统
> https://juicefs.com/docs/zh/community/command_reference#mount

- cache-size = MemAvailable × 50% ~ 70%

```bash
juicefs mount \
    --background \  # 后台挂载
    --cache-dir /data/jfs-cache \  # 必须指向 SSD 路径！
    --cache-size 10240 \          # 单位是 MiB，这里是 10240MiB = 10GB
    --max-uploads 50 \           # 最大并发上传数，默认 20
    --buffer-size 2048 \        # 读写缓冲区的总大小；单位为 MiB (默认：300)。
    --writeback \              # 开启写回缓存，默认关闭
    --free-space-ratio 0.1 \     # 保持磁盘至少 10% 的剩余空间，防止撑爆系统盘
    --prefetch 2 \              # 预读文件数，默认 1
    tikv://root:password@192.168.112.103:6241/jfs \  # 元数据地址
    --log-level info \          # 日志级别，默认 info
    --trash-days 1 \           # 回收站天数，默认 7
    --block-size 8192 \        # 块大小，默认 4096
    --attr-cache 10 \            # 属性缓存过期时间；单位为秒，默认为 1。 
    --entry-cache 10 \            # 目录缓存过期时间；单位为秒，默认为 2409600。
    --dir-entry-cache 10 \            # 目录缓存过期时间；单位为秒，默认为 2409600。
    /mnt/jfs
```

  

## 常见问题

### 回收站

> https://juicefs.com/docs/zh/community/security/trash

juice删除文件不是直接删除，而是移动到回收站，默认1天后自动删除，通过--trash-days参数可以修改回收站的天数

```bash
# 初始化新的文件系统
juicefs format META-URL myjfs --trash-days=7

# 修改已有文件系统
juicefs config META-URL --trash-days=7

# 设置为 0 以禁用回收站
juicefs config META-URL --trash-days=0
```

## 性能压测

### 1. 基础功能测试（手动验证）
首先确保多机挂载后，文件能正常同步。

* **Node A 执行：**
    ```bash
    echo "Hello JuiceFS" > /mnt/jfs/test_node_a.txt
    ```
* **Node B 执行：**
    ```bash
    cat /mnt/jfs/test_node_a.txt
    ```
* **MinIO 验证：** 登录 MinIO 控制台，查看对应的 Bucket。你应该能看到很多经过混淆的 `chunks` 和 `blocks` 目录。
    > **注意**：JuiceFS 会将文件切块存储，你无法在 MinIO 里直接看到 `test_node_a.txt` 的明文，必须通过 JuiceFS 挂载点查看。

---

### 2. 写入性能测试 (使用 `dd`)
这是测试**顺序写入**带宽最快的方法。建议测试大于缓存（`--cache-size`）的文件，以穿透缓存直接测试 MinIO 的真实写入速度。

```bash
# 写入一个 2GB 的大文件
dd if=/dev/zero of=/mnt/jfs/big_file_test bs=1M count=2048 conv=fsync

# 模拟视频流写入
dd if=/dev/zero of=/mnt/jfs/test_heavy bs=4M count=512 conv=fsync

# 模拟频繁的小数据更新
dd if=/dev/zero of=/mnt/jfs/test_small bs=4k count=100000 conv=fsync

```
* **观察点**：查看 `MB/s` 数值。如果数值远低于你的网络带宽，可能需要调整 JuiceFS 的 `--max-uploads` 参数。

---

### 3. 内置压力测试 (推荐：`juicefs bench`)
JuiceFS 自带了一个非常强大的性能评估工具，它会模拟大文件和小文件的创建、读写及删除，并给出统计报告。

```bash
juicefs bench /mnt/jfs -p [threadnum]
```
**输出示例解读：**
* **Write**: 顺序写入速度。
* **Read**: 顺序读取速度。
* **Stat**: 元数据查询性能（反映了 TiKV 的响应速度）。
* **Create/Delete**: 小文件处理能力。

---

### 4. 实时监控 (使用 `juicefs stats`)
在进行上传测试的同时，打开另一个终端运行此命令。它可以让你实时看到：
* **Local**: 正在写入本地缓存的速度。
* **Object Store (S3)**：正在上传到 MinIO 的真实速度。
* **Metadata**: 访问 TiKV 的延迟。

```bash
juicefs stats /mnt/jfs
```



---

### 5. 专业级压测 (使用 `fio`)
如果你需要模拟高并发写入（例如多台相机同时录制视频数据），建议使用 `fio`：

```bash
sudo apt install fio

# 模拟 16 个并发线程写入小文件（模拟传感器数据碎片）
fio --name=juicefs-test --directory=/mnt/jfs --rw=write --bs=4k --size=1G --numjobs=16 --time_based --runtime=60 --group_reporting
```

---

### 性能调优建议
如果你发现上传速度不理想，可以尝试在 **每台机器** 的 `mount` 参数中添加以下优化：

* **`--max-uploads=50`**: 增加并发上传到 MinIO 的连接数（默认 20）。
* **`--writeback`**: 开启写缓存（文件先写到本地 SSD，后台异步上传到 MinIO）。**注意：对于多机共享，开启此项可能存在极短的数据一致性窗口。**
* **`--buffer-size=300`**: 增大读写缓冲区（单位 MB）。

**既然你正在处理 Embodied AI 的视频数据，是否需要我针对“大量短视频文件”或“海量图片”的场景帮你优化挂载参数？**

## 重启 JuiceFS 挂载

1. 强制刷缓存（非常重要）
   ```bash
   sync
   ```
2. 卸载（优先优雅）
   ```bash
   fusermount -u /mnt/jfs
   ```
1. 如果失败再用
   ```bash
   fusermount -uz /mnt/jfs
   ```
1. 重新挂载（修正后的命令）
   ```bash
   juicefs mount \
    --cache-dir /data/jfs-cache \
  --cache-size 10240 \
  --max-uploads 30 \
  --buffer-size 400 \
  --writeback \
  --free-space-ratio 0.1 \
  --prefetch 2 \
  tikv://root:password@192.168.112.103:6241/jfs \
  --log-level info \
  /mnt/jfs
  ```