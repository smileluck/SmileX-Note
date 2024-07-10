[toc]

---

# 前言

Beautifulsoup4是一个用于解析HTML和XML文档的Pyhon库。它提供了简单而直观的方式来遍历、搜索和修改文档树，BeauifiulSoup4可以很好地与Pyhon的请求库(如requests)一起使用，用于从网页中提取信息。

# 相关操作

## 安装

```shell
pip install beautifulsoup4
```

## 获取网页内容(requests)

```python
import requests

response = requests.get(url % page)
htmlText = response.text
```

## 搜索Script并提取变量

```python
from bs4 import BeautifulSoup
import re 

soup = BeautifulSoup(htmlText, "html.parser")

# Get the script tags
scripts = soup.find_all("script", attrs={'type': "application/ld+json"})
## notice：{’type‘:"xxx"}和{type:"xxx"}，the search result isn't equal

# reg match variables
variables = []
for script in scripts:
    matches = re.findall(r"vars+(\w+)\s*=\s*(. ?);", str(script))
    if matches:
        for match in matches:
            variables.append((match[0], match[1]))

# print result
for name, value in variables:
    print(f"{name}:{value}")
```

- 正则匹配json 

  ```python
  pattern = re.compile(r'\{.*\}‘)
  ```

  