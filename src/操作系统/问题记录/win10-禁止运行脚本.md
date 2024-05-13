[toc]

---

# 关于在前端npm 安装全局命令后，执行命令会遭到拒绝

> https://learn.microsoft.com/zh-cn/powershell/module/microsoft.powershell.core/about/about_execution_policies?view=powershell-7.4

- 异常信息：

```shell
dts-gen : 无法加载文件 D:\dev\repositories\node_global\dts-gen.ps1，因为在此系统上禁止运行脚本。有关详细信息，请参阅 ht
tps:/go.microsoft.com/fwlink/?LinkID=135170 中的 about_Execution_Policies。
```

- 解决办法：
  1. 设置当前用户的执行策略，允许脚本执行
    ```shell
  Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
    ```
  2. 执行完后记得将权限设置会默认的 `Restricted`

    ```shell
	Set-ExecutionPolicy -ExecutionPolicy Restricted -Scope CurrentUser
    ```

