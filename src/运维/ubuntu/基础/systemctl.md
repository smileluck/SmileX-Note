[toc]

---
# 常用指令

```shell
# 重新加载所有被修改过的服务配置，否则配置不会生效
sudo systemctl daemon-reload

# 系统开机时自动启动指定unit，前提是配置文件中有相关配置
sudo systemctl enable test.service

# 启动服务
sudo systemctl start test.service

# 停止服务
sudo systemctl stop test.service

# 重启服务
sudo systemctl restart test.service

# 重新加载配置
sudo systemctl reload test.service

# 如果Unit无法正常停止，使用强杀
sudo systemctl kill test.service

# 显示Unit状态
sudo systemctl status test.service

# 显示Unit是否在运行
sudo systemctl is-active test.service

# 显示Unit是否开机运行
sudo systemctl is-enabled test.service

# 显示unit列表
sudo systemctl list-units
sudo systemctl list-units --all # 列出所有Unit，包括缺失配置文件或启动失败的
sudo systemctl list-units --all --state=inactive # 列出所有没有运行的Unit

```

