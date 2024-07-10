[toc]

---

# 使用第三方库pipreqs

使用第三方库pipreqs生成项目的 requirements.txt 文件，pipreqs会分析项目中的 Python 源代码文件，找出所有依赖的包，并将它们及其版本写入 requirements.txt 文件。pipreqs可以只将用到的库生成到requirements.txt文件。

1. 先安装pipreqs库

	```shell
    pip install pipreqs
	```

2. 生成 requirement
	
	```shell
	pipreqs ./ --encoding=utf8  --force
	```
	
	- –encoding=utf8 ：为使用utf8编码
	- –force ：强制执行，当 生成目录下的requirements.txt存在时覆盖
	- . /: 在哪个文件生成requirements.txt 文件

# 安装需要的库

```shell
pip install -r requirements.txt
```

