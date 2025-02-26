# nuxt-image
## 使用webp时，在ssr模式下无法生成/_ipx/

> IPX_FILE_NOT_FOUND when using npm run build 

临时解决方案：
1. 通过 `pnpm generate` 生成出 _ipx 文件夹， 然后通过nginx反向代理
2. nginx 配置
   ```nginx
    location ^~ /_ipx/ {
      # proxy_pass http://127.0.0.1:<nuxt_website_port>/_ipx/;
      # proxy_set_header Host $host;
      #  proxy_set_header X-Real-IP $remote_addr;
      #  proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      #  proxy_set_header X-Forwarded-Proto $scheme;

        # Cache optimized images (example)
        alias /www/wwwroot/test.kuosanyinzi.com/_ipx/;
        autoindex off;
        expires 30d;
        add_header Cache-Control "public, max-age=2592000";
        # Explicitly allow HEAD requests (example)
        if ($request_method !~ ^(GET|HEAD)$) {
            return 405;
        }
    }
   ```