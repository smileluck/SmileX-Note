## 文档

- [tikv 部署](https://docs.pingcap.com/zh/tidb/stable/production-deployment-using-tiup/#%E7%AC%AC-3-%E6%AD%A5%E5%88%9D%E5%A7%8B%E5%8C%96%E9%9B%86%E7%BE%A4%E6%8B%93%E6%89%91%E6%96%87%E4%BB%B6)
- [tikv 官网](https://tikv.org/docs/7.1/deploy/install/production/)

## 单机部署三节点集群

### 配置

> host 尽量不使用127.0.0.1，而是使用具体的IP地址，局域网或者公网的IP地址。

```yaml
global:
  user: "tikv"
  ssh_port: 22
  deploy_dir: "/data/sdb/tikv/tidb-deploy"
  data_dir: "/data/sdb/tikv/tidb-data"

server_configs:
  tikv:
    readpool.unified.max-thread-count: 8
    readpool.storage.use-unified-pool: false
    readpool.coprocessor.use-unified-pool: true
    storage.block-cache.capacity: "4GB"
    raftstore.capacity: "0"
  pd:
    replication.location-labels: ["host"]

pd_servers:
  - host: 127.0.0.1
    name: "pd-1"           # 必须唯一
    client_port: 6200      # 必须唯一
    peer_port: 6201        # 必须唯一
    data_dir: "/data/sdb/tikv/tidb-data/pd-6200"
  - host: 127.0.0.1
    name: "pd-2"           # 必须唯一
    client_port: 6202      # 必须唯一
    peer_port: 6203        # 必须唯一
    data_dir: "/data/sdb/tikv/tidb-data/pd-6202"
  - host: 127.0.0.1
    name: "pd-3"           # 必须唯一
    client_port: 6204      # 必须唯一
    peer_port: 6205        # 必须唯一
    data_dir: "/data/sdb/tikv/tidb-data/pd-6204"


# tidb_servers:
#   - host: 127.0.0.1
#     port: 6210
#     status_port: 6211

#   - host: 127.0.0.1
#     port: 6212
#     status_port: 6213


tikv_servers:
  - host: 127.0.0.1
    port: 6220
    status_port: 6221
    data_dir: /data/sdb/tikv/tikv1-data
    config:
      server.labels: { host: "tikv1" }

  - host: 127.0.0.1
    port: 6222
    status_port: 6223
    data_dir: /data/sdb/tikv/tikv2-data
    config:
      server.labels: { host: "tikv2" }

  - host: 127.0.0.1
    port: 6224
    status_port: 6225
    data_dir: /data/sdb/tikv/tikv3-data
    config:
      server.labels: { host: "tikv3" }


monitoring_servers:
  - host: 127.0.0.1
    port: 6230

grafana_servers:
  - host: 127.0.0.1
    port: 6231

alertmanager_servers:
  - host: 127.0.0.1
    web_port: 6232
    cluster_port: 6233
```

### 配置/etc/sysctl.conf

```shell
vim /etc/sysctl.conf

# /etc/sysctl.conf
fs.file-max=1000000 # 整个操作系统内核能够分配的最大文件句柄
net.ipv4.tcp_syncookies = 0 # 减少 CPU 开销，提升内网 TCP 吞吐
vm.swappiness = 0 # 防止内存置换导致的磁盘 IO 延迟抖动

# 生效
sysctl -p

```

### 修改 Systemd 全局配置

- 编辑文件：sudo vi /etc/systemd/system.conf 和 sudo vi /etc/systemd/user.conf
- 找到或添加以下行：
    ```
    DefaultLimitNOFILE=1000000
    ```
- 重载配置 
    ```
    sudo systemctl daemon-reexec
    ```

### 修改用户层面的限制(limits.conf)
- 假设启动用户是tikv
- 编辑文件：sudo vi /etc/security/limits.conf
- 添加以下行：
    ```
    tikv soft nofile 1000000
    tikv hard nofile 1000000
    tikv soft stack 32768
    tikv hard stack 32768
    ```

### final: 加载配置和验证
- 加载配置
  - sudo systemctl daemon-reload
- 集群
  - tiup cluster deploy tidb-test v8.5.5 ./topology.yaml --user root
  - tiup cluster start <cluster-name>
- 验证
  - ulimit -Hn