[toc]

---

# stash

- `git stash save "备注信息"`。 常用来取代`git stash`命令，为本次暂存添加备注信息 ，只处理已被版本控制了的。
- `git stash push --include-untracked -m "备注信息" `。 为本次暂存添加备注信息 ，处理已被版本控制和未被版本控制了的问题。
- `git stash list`：查看缓存栈素有的缓存内容
- `git stash pop` 。 将暂存的代码恢复到工作区，注意该命令会把缓存栈中栈顶的内容删除 
- `git stash pop [stash@{数字}]`：将指定的缓存内容恢复到工作区，`stash@{数字}`从`git stash list`命令的显示内容中可得。
- `git stash apply`：将暂存的代码恢复到工作区，不删除栈顶内容。
- `git stash apply [stash@{数字}]`：将指定暂存的代码恢复到工作区，不删除内容。
- `git stash drop stash@{数字}`：删除对应的缓存内容，`stash@{数字}`从`git stash list`命令的显示内容中可得
- `git stash clear`：删除所有的缓存内容





# restore

- git restore --staged . 。删除 add 的所有文件