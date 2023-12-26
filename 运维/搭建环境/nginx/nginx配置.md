[toc]

---

# http访问

## 单机部署

```conf
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