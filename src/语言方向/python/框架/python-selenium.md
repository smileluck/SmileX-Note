[toc]

---

# 简介
Selenium 是一个用于 Web 应用程序测试的工具。最初是为网站自动化测试而开发的，可以直接运行在浏览器上，支持的浏览器包括 IE（7, 8, 9, 10, 11），Mozilla Firefox，Safari，Google Chrome，Opera 和 Edge 等。
  爬虫中使用它是为了解决 requests 无法直接执行 JavaScript 代码的问题。Selenium 本质上是通过驱动浏览器，彻底模拟浏览器的操作，好比跳转、输入、点击、下拉等，来拿到网页渲染之后的结果。Selenium 是 Python 的一个第三方库，对外提供的接口能够操作浏览器，从而让浏览器完成自动化的操作。

# 安装

## 驱动安装
> 版本向下兼容，可以下载最新版本即可
- 国内镜像下载：https://registry.npmmirror.com/binary.html?path=chromedriver/
- 官网：https://googlechromelabs.github.io/chrome-for-testing/#stable


## python 安装和使用

1. 安装依赖库

```shell
pip install selenium
```

2. 简单使用
```python
from selenium import webdriver

# 创建浏览器操作对象
path = 'chromedriver.exe'
browser= webdriver.Chrome(path)

# 访问网站
url = 'https://www.baidu.com'

browser.get(url)
```


# 使用
## 获取cookied