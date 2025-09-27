
## 多层ssh配置
即使经过多次 SSH 跳转（即通过多层 SSH 连接到目标服务器），仍然可以进行远程调试，不过需要进行一些额外配置才能让 VS Code 正确识别调试环境。

### 实现方式
1. **配置 SSH 跳转链**  
   在本地的 `~/.ssh/config`（Windows 通常是 `C:\Users\<用户名>\.ssh\config`）中配置多层跳转，例如：
   ```ssh-config
   # 第一层跳板机
   Host jump-host
     HostName 跳板机IP或域名
     User 用户名
     Port 22

   # 目标服务器（通过跳板机连接）
   Host target-host
     HostName 目标服务器内网IP
     User 目标服务器用户名
     ProxyJump jump-host  # 通过跳板机跳转
   ```
   这样配置后，VS Code 的 Remote-SSH 扩展可以直接通过 `target-host` 连接到最终服务器，无需手动执行多次 SSH 命令。

2. **在目标服务器上配置调试环境**  
   确保目标服务器上安装了所需的调试工具（如 Python 解释器、Node.js、GDB 等），以及对应语言的调试扩展（在 VS Code 远程窗口中安装）。

3. **正常配置调试文件**  
   在 VS Code 中打开远程项目后，创建常规的调试配置文件（如 `.vscode/launch.json`），配置方式与直接 SSH 连接单台服务器相同，VS Code 会自动通过已建立的 SSH 通道进行调试通信。

### 注意事项
- 跳转链的网络稳定性可能影响调试体验，建议确保各层 SSH 连接稳定。
- 若中间跳板机有特殊网络限制（如端口封锁），可能需要在 `config` 中额外配置端口转发。

通过上述配置，多层 SSH 跳转环境下的远程调试可以和直接连接一样顺畅。