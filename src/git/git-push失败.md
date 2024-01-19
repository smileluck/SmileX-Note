[toc]

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

    