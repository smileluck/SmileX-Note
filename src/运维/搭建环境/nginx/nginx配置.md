[toc]

---

# http访问

## 单机部署

```nginx
 server {
        listen          443 ssl;
        server_name     localhost;

        ssl_certificate /usr/local/nginx/cert/localhost.pem; # 改为自己申请得到的 crt 文件的名称
        ssl_certificate_key /usr/local/nginx/cert/localhost.key; # 改为自己申请得到的 key 文件的名称
        ssl_session_timeout 5m;
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
        ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE;
        ssl_prefer_server_ciphers on;

        location ^~ /smilex/ {
                proxy_redirect off;
                proxy_set_header Host $host;
                proxy_set_header X-Nginx-Proxy true;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_pass http://127.0.0.1:8080;
        }

		# history route
      location ^~ /admin/ {
                proxy_set_header X-Forwarded-For  $proxy_add_x_forwarded_for;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header Host  $host;
                alias /usr/local/serve/frontend/smilex-admin/;
                index index.html index.htm;
                try_files $uri $uri/ /admin/index.html;
        }


        
        # hash route
        #location /admin/ {
        
        #        if ($request_filename ~* ^.*?.(html|htm)$) {
        #              add_header Cache-Control "private, no-store, no-cache, must-revalidate, proxy-revalidate";
        #        }

        #        root /usr/local/serve/frontend/smilex-admin;
        #        index index.html index.htm;
        #}

    }


```

## 腾讯云ssl

```nginx
server {
     #SSL 默认访问端口号为 443
     listen 443 ssl; 
     #请填写绑定证书的域名
     server_name cloud.tencent.com; 
     #请填写证书文件的相对路径或绝对路径
     ssl_certificate cloud.tencent.com_bundle.crt; 
     #请填写私钥文件的相对路径或绝对路径
     ssl_certificate_key cloud.tencent.com.key; 
     ssl_session_timeout 5m;
     #请按照以下协议配置
     ssl_protocols TLSv1.2 TLSv1.3; 
     #请按照以下套件配置，配置加密套件，写法遵循 openssl 标准。
     ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE; 
     ssl_prefer_server_ciphers on;
     location / {
         #网站主页路径。此路径仅供参考，具体请您按照实际目录操作。
         #例如，您的网站主页在 Nginx 服务器的 /etc/www 目录下，则请修改 root 后面的 html 为 /etc/www。
         root html; 
         index  index.html index.htm;
     }
 }
```

## rabbitmq

## Web管理端

```nginx
{
     #默认访问端口号为 80
     listen 15673; 
     #请填写绑定证书的域名
     server_name cloud.tencent.com; 
    location / {
        port_in_redirect on;
        proxy_redirect off;
        proxy_set_header Host $host;
        proxy_set_header X-Nginx-Proxy true;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_pass http://localhost:15672;
    }
 }
```

## Amqp

 这种配置方式比配置Rabbitmq自身的SSL方法要简单。也适用于其他的服务（Redis）等。

1. 修改 `nginx.conf`

```shell
stream {
    include /etc/nginx/conf/stream/*.conf
}
```

2. 在 `/etc/nginx/conf/stream`

```shell
upstream rabbitmq_backend {
    server rabbitmq:5672;
}
 
server {
    listen 5672 ssl;
    proxy_pass rabbitmq_backend;
    ssl_certificate certs/********.pem;
    ssl_certificate_key certs/********.key;
    ssl_session_timeout 5m;
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
    ssl_protocols TLSv1.1 TLSv1.2 TLSv1.3;
    ssl_prefer_server_ciphers on;
}
```

## 前端版本更新

```nginx
server {
   listen 80;
   server_name api.dev.com;
   client_max_body_size 10m;

   # 老网页 v1.1.0 配置
   location ~ ^/v110 {
               alias  /home/ubuntu/api.dev.com/V110;
               index  index.html index.htm;
       }
       
   # 新网页 v1.2.0 配置
   location ~ ^/v120 {
               alias  /home/ubuntu/api.dev.com/V120;
               index  index.html index.htm;
       }

}
```

 在 `nginx` 配置文件语法中，`location` 语句可以使用正则表达式，定义 `set $s $1` 变量，实现了通用配置 

```nginx
server {
   listen 80;
   server_name api.dev.com;
   client_max_body_size 10m;

   # 配置正则 localtion
   location ~ ^/pageV(.*) {
               set $s $1; # 定义后缀变量
               alias  /home/wwwroot/api.dev.com/pageV$s;
               index  index.html index.htm;
       }

}
```

## 反向代理转发数据

- 发送 temp.com/test/1 的会转发到 test.com/8001/1。
- 注意location 的/test/ 和proxy_pass 的斜杠

```nginx
location /test/ {
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    #proxy_set_header X-Nginx-Proxy true;
    
    # 后台接口地址
    proxy_connect_timeout 3600s;
    proxy_send_timeout      3600s;
    proxy_read_timeout      3600s;
    rewrite ^/test/(.*)$ /$1 break;
    proxy_pass http://test.com:8001;

    proxy_redirect default;
    add_header Access-Control-Allow-Origin *;
    add_header Access-Control-Allow-Headers X-Requested-With;
    add_header Access-Control-Allow-Methods GET,POST,OPTIONS;

}
```



### 示例

```shell

        server {
     #SSL 默认访问端口号为 443
     listen 9029 ssl; 
     #请填写绑定证书的域名
     server_name model.example.com; 
     #请填写证书文件的相对路径或绝对路径
     ssl_certificate /etc/nginx/cert/model.example.com_bundle.crt; 
     #请填写私钥文件的相对路径或绝对路径
     ssl_certificate_key /etc/nginx/cert/model.example.com.key; 
     ssl_session_timeout 5m;
     #请按照以下协议配置
     ssl_protocols TLSv1.2 TLSv1.3; 
     #请按照以下套件配置，配置加密套件，写法遵循 openssl 标准。
     ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE; 
     ssl_prefer_server_ciphers on;
        location /video-inject-3d/ {
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    #proxy_set_header X-Nginx-Proxy true;


# 跨域
if ($request_method = 'OPTIONS') {
        add_header Access-Control-Allow-Origin *;
add_header Access-Control-Allow-Methods 'GET, POST, OPTIONS';
add_header Access-Control-Allow-Headers 'DNT,X-Mx-ReqToken,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Authorization,x-request-id';
        return 204;
}


    # 后台接口地址
    proxy_connect_timeout 3600s;
    proxy_send_timeout      3600s;
    proxy_read_timeout      3600s;
    rewrite ^/video-inject-3d/(.*)$ /$1 break;
    proxy_pass http://127.0.0.1:9020;

   
}
```





# 常用的配置说明

## 服务配置

- proxy_pass 。转发转发地址

- rewrite 。转发

  ```nginx
  #只取路径 /test/ 后的
  # 例如 /test/1 ，那么就变成 /1
  rewrite ^/test/(.*)$ /$1 break;
  ```


## 跨域

```nginx
add_header Access-Control-Allow-Origin *;
add_header Access-Control-Allow-Methods 'GET, POST, OPTIONS';
add_header Access-Control-Allow-Headers 'DNT,X-Mx-ReqToken,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Authorization,x-request-id';

    if ($request_method = 'OPTIONS') {
        return 204;
}
```

## http重定向https

```nginx
server {
    listen 80;
    server_name example.com www.example.com;
    return 301 https://$host$request_uri;
}
```

## @跳转www

```shell
server {
	listen 443;
    server_name example.com;
ssl_certificate /www/server/nginx/cert/example.com_nginx/example.com_bundle.crt;
     #请填写私钥文件的相对路径或绝对路径
     ssl_certificate_key /www/server/nginx/cert/example.com_nginx/example.com.key;
     ssl_session_timeout 5m;
     #请按照以下协议配置
     ssl_protocols TLSv1.2 TLSv1.3;
     #请按照以下套件配置，配置加密套件，写法遵循 openssl 标准。
     ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE;
     ssl_prefer_server_ciphers on;

    return 301 https://example.com$request_uri;
}
```

注意 cos配置跨域时，要避免 example.com和www.example.com的配置。



## 日志相关

- error_log。error_log [错误日志地址] [level]。以下警报等级，从高到底，一般生产不会设置warn以上的以免影响性能。
  - debug -调试消息。
  - info -信息性消息。
  - notice -公告。
  - warn -警告。
  - error -处理请求时出错。
  - crit -关键问题。 需要立即采取行动。
  - alert -警报。 必须立即采取行动。
  - emerg - 紧急情况。 系统处于无法使用的状态

# 说明

## proxy_pass 有无斜杠问题

1. 无斜杆：`http://localhost:3000`
2. 有斜杆：`http://localhost:3000/`

3. 请求部分/test/r1，请求为 http://localhost:8080/test/r1

### 无斜杠

```nginx
server {
        listen 8080;
        server_name localhost;

        location /test {
            proxy_pass http://localhost:3000;
        }
        #或者
        location /test/ {
            proxy_pass http://localhost:3000;
        }
        
        #结果都是 将http://localhost:8080/test/r1转发去http://localhost:3000/test/r1
    }

```

-  无斜杆location匹配到的部分也属于请求的部分。 
-  location无论用`/test`还是用`/test/`只要匹配上之后都会将整个请求部分`/test/r1`加到proxy_pass上。 

### 有斜杠

```nginx
server {
        listen 8080;
        server_name localhost;

        location /test {
            proxy_pass http://localhost:3000/;
        }
        #或者
        location /test/ {
            proxy_pass http://localhost:3000/;
        }
        
        #结果都是 将http://localhost:8080/r1转发去http://localhost:3000/r1
    }

```

- proxy_pass：`http://localhost:3000/`。

- 有斜杆location匹配到的部分只用于匹配，不属于请求部分，需要在请求部分将location匹配到的部分剔除。

### 斜杠还有字符

```nginx
server {
        listen 8080;
        server_name localhost;

        location /test {
        	#结果都是 将http://localhost:8080/test/r1转发去http://localhost:3000/abc/r1
            proxy_pass http://localhost:3000/abc;
        }
        #或者
        location /test/ {
        	#结果都是 将http://localhost:8080/test/r1转发去http://localhost:3000/abcr1
            proxy_pass http://localhost:3000/abc;
        }
        
    }
```

