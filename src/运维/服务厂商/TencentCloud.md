[TOC]

---

# 卸载 barad_agent

> 腾讯云监控

```shell
wget -qO- https://raw.githubusercontent.com/littleplus/TencentAgentRemove/master/remove.sh | bash
```

# 防火墙开放后不生效

更改防火墙后，对应端口仍然无法访问。需要重启一下服务。

```shell
#查看防火墙状态 
systemctl status firewalld
#开启防火墙 
systemctl start firewalld  
#关闭防火墙 
systemctl stop firewalld

#若遇到无法开启
#先用：
systemctl unmask firewalld.service 
#然后：
systemctl start firewalld.service
```

如果不生效，就需要手动添加端口放行了（系统自带的防火墙是默认不开的，如果只想使用控制台的，最好关闭系统防火墙）

```shell
# 查询防火墙开放的端口列表
firewall-cmd --zone=public --list-ports

#查询已开放的端口 
netstat  -ntulp | grep 3306

#查询指定端口是否已开.开启:yes,未开启:no。
firewall-cmd --query-port=666/tcp

# 手动开放端口
firewall-cmd --zone=public --add-port=8080/tcp --permanent

#移除3306端口：
firewall-cmd --zone=public --permanent --remove-port=3306/tcp

# 重新加载一下防火墙配置规则
firewall-cmd --reload 
```
