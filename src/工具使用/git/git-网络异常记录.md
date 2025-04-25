[TOC]

---

# github ssh push失败

- 问题描述：ssh push时，抛出异常 `github.com port 22: Connection timed out`

- 解决办法：
  
  - 排除网路问题，如果实在不行，可以考虑使用443端口
  
  - 进入 `./ssh` 文件夹
    
    ```shell
    # win
    cd C:\Users\[用户名]\.ssh
    # linux
    cd ~/.ssh
    ```
  
  - 创建并编辑 `config` 文件
    
    ```shell
    # win 
    notepad config
    # vim 
    vim config
    ```
  
  - 输入内容，退出 并保存
    
    ```
    Host github.com
    User git
    Hostname ssh.github.com
    PreferredAuthentications publickey
    IdentityFile C:/Users/Admin/.ssh/id_rsa # 自己的地址
    Port 443
    ```
    
    ```
    Host gitlab.com
    Hostname altssh.gitlab.com
    User git
    Port 443
    PreferredAuthentications publickey
    IdentityFile C:/Users/Admin/.ssh/id_rsa # 自己的地址
    ```

- 检查是否成功
  
  ```shell
  ssh -T -p 22 git@github.com
  
  ssh -T -p 443 git@github.com
  ```

# 访问不稳定

- 问题描述：浏览器上访问 Github 时，无法访问。

- 解决办法：
  
  - 查询ip工具：https://www.ipaddress.com/ （或https://www.ipaddress.com/ip-lookup）
  
  - 需要查询的网址信息
    
    ```host
    199.232.69.194 github.global.ssl.fastly.net
    185.199.111.133 raw.githubusercontent.com
    140.82.112.4 github.com
    ```
  
  - 需更改配置文件：hosts 地址 `C:\Windows\System32\drivers\etc\hosts`
  

# 关于 ssh.github.com 443 connection refused

- 问题描述：ssh 执行 push/pull/fetch 时操作时，出现 `ssh.github.com 443: Connection refused`
- 排查方式:
  - 试试443/22端口访问
  ```shell
  ssh -T -p 22 git@github.com
  
  ssh -T -p 443 git@github.com
  ```
  - 检查ssh请求明细
  ```shell
  ssh -v -v -v git@github.com
  ```
  - 扫描域名
  ```shell
  nslookup github.com
  ```
- 解决办法：
  - 更换dns【8.8.8.8/8.8.4.4】，并清空dns缓存
  - 发现nslookup结果为[::1]，可能使用了ipv6。可以通过禁用ipv6解决