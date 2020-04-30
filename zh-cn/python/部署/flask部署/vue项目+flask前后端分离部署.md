## nginx部署

- 前端部署时，可以将前端的index.html所在目录映射到‘/’路径下，表示通过：http://www.example.com就能访问到首页；
- 后端部署时，为了解决跨域的问题，可以使用‘/api’的方式，将后端的启动目录映射到：http://www.example.com/api目录下，这样就不会跨域了,前端就可以通过'http://www.example.com/api/'来请求后端的接口：http://127.0.0.1:5000/request1。（其中proxy_pass http://127.0.0.1:5000/中末尾的‘/’起了关键作用，末尾加了‘/’,则最终请求的url会不带/api,如果末尾不带‘/’,则最终请求地址会带上‘/api’,变成‘http://127.0.0.1:5000/api/request1’,这样就不符合要求了）
```nginx
    ...
    server_name http://www.example.com
    ....
    location / {
        root /home/OldManInfo_Flask/frontend/dist;
        index index.html;
        autoindex on;
        autoindex_exact_size on;
        autoindex_localtime on;
    }

    location /api/ {
        proxy_pass http://127.0.0.1:5000/;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }

```
> 其中location /api/如果api末尾不加斜杠，则匹配的最终路径是’//code‘,所以得加上斜杠