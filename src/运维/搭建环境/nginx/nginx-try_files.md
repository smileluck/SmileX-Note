[TOC]

---

# try_files详解

Checks the existence of files in the specified order and uses the first found file for request processing; the processing is performed in the current context. The path to a file is constructed from the `*file*`parameter according to the [root](http://nginx.org/en/docs/http/ngx_http_core_module.html#root) and [alias](http://nginx.org/en/docs/http/ngx_http_core_module.html#alias) directives. It is possible to check directory’s existence by specifying a slash at the end of a name, e.g. “`$uri/`”. If none of the files were found, an internal redirect to the `*uri*` specified in the last parameter is made. 

　　关键点1：按指定的file顺序查找存在的文件，并使用第一个找到的文件进行请求处理

　　关键点2：查找路径是按照给定的root或alias为根路径来查找的

　　关键点3：如果给出的file都没有匹配到，则重新请求最后一个参数给定的uri，就是新的location匹配

　　关键点4：如果是格式2，如果最后一个参数是 = 404 ，若给出的file都没有匹配到，则最后返回404的响应码

# `try_files` 使用方法

`try_files` 指令的基本语法如下：

```nginx
try_files file1 [file2 ...] uri;
try_files file1 [file2 ...] =code;
```

- **`file1`, `file2`, ...**：要检查的文件路径。
- **`uri`**：如果所有文件都不存在，则重定向到这个 URI。
- **`=code`**：如果所有文件都不存在，则返回指定的 HTTP 状态码。

# 示例

```nginx
location / {
try_files $uri $uri/ /index.html;
}
```

  在这个示例中：

1. Nginx 首先尝试查找 `$uri` 对应的文件。
2. 如果文件不存在，尝试查找 `$uri/` 对应的目录。
3. 如果目录也不存在，返回 `/index.html`。

# 注意事项

4. **路径匹配**：
   - `$uri` 和 `$uri/` 是相对路径，基于 `root` 或 `alias` 指令指定的目录。
   - `$uri` 匹配文件，`$uri/` 匹配目录。
5. **性能**：
   - `try_files` 指令会按顺序检查文件是否存在，因此应将最可能存在的文件放在前面。
6. **重定向**：
   - 如果 `try_files` 指令中最后一个参数是 URI，则会进行内部重定向。
   - 如果最后一个参数是 `=code`，则直接返回指定的 HTTP 状态码。
7. **安全性**：
   - 确保 `try_files` 指令中的路径不会暴露敏感文件。

# `try_files` 与 `root` 和 `alias` 的区别

## `root`

- **定义**：指定请求的根目录。

- **使用**：Nginx 会在 `root` 目录下查找文件。

- **示例**：
  
  ```nginx
  location /user {
  root /path/to/your/dist;
  try_files $uri $uri/ /user/index.html;
  }
  ```
  
  在这个示例中，Nginx 会在 `/path/to/your/dist/user` 目录下查找文件。

## `alias`

- **定义**：将请求的 URI 替换为指定的目录。

- **使用**：Nginx 会将 `location` 匹配的部分替换为 `alias` 指定的目录。

- **示例**：
  
  ```nginx
  location /user {
  alias /path/to/your/dist;
  try_files $uri $uri/ /index.html;
  }
  ```
  
  在这个示例中，Nginx 会在 `/path/to/your/dist` 目录下查找文件，而不会在 `/path/to/your/dist/user` 目录下查找。

# 总结

- **`try_files`**：用于检查文件是否存在，并根据结果采取不同的操作。
- **`root`**：指定请求的根目录，Nginx 会在该目录下查找文件。
- **`alias`**：将请求的 URI 替换为指定的目录，Nginx 会直接在该目录下查找文件。

# 示例对比

## 使用 `root`

```nginx
location /user {
root /path/to/your/dist;
try_files $uri $uri/ /user/index.html;
}
```

- 请求 `/user/about` 时，Nginx 会在 `/path/to/your/dist/user/about` 查找文件或目录。

## 使用 `alias`

```nginx
location /user {
alias /path/to/your/dist;
try_files $uri $uri/ /index.html;
}
```

- 请求 `/user/about` 时，Nginx 会在 `/path/to/your/dist/about` 查找文件或目录。
  通过理解 `try_files` 与 `root` 和 `alias` 的区别，你可以更灵活地配置 Nginx 以满足不同的需求。
