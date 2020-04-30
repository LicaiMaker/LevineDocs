## nginx读取文件
nginx读取配置的顺序，/etc/nginx/nginx.conf文件包含了sites-enable,所以在sites-available中新建了虚拟主机后，还需要再在sites-enable中新建一个软链接才行，当然了，你也可以将自己的多个配置文件放在conf.d中，不过所有配置文件都要以.conf结尾(可以在nginx.conf中取消这个限制)，比如：123.com.conf,456.com.conf等等，同时，要在nginx.conf中引入这个文件夹：
```shell
 #导入外部服务器配置文件存放地址
include /etc/nginx/conf.d/*.conf;
```
## nginx 指定配置文件启动
`service nginx -c 123.com.conf start `

## nginx server配置文件：/etc/nginx/conf.d/*.conf
默认虚拟主机default.conf 如下：
```
server {
    listen       80;            #该虚拟主机监听的端口
    server_name  localhost;     #虚拟主机名称，对于不同主机，可以通过server_name来区别，可以通过监听端口来区别

    #charset koi8-r;
    #access_log  /var/log/nginx/host.access.log  main;

    #location 匹配根路径
    location / {
        root   /usr/share/nginx/html;   #返回的页面的根路径
        index  index.html index.htm;    #默认返回页面
    }

    #404错误页面
    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

    # proxy the PHP scripts to Apache listening on 127.0.0.1:80
    #
    #location ~ \.php$ {
    #    proxy_pass   http://127.0.0.1;
    #}

    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    #
    #location ~ \.php$ {
    #    root           html;
    #    fastcgi_pass   127.0.0.1:9000;
    #    fastcgi_index  index.php;
    #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
    #    include        fastcgi_params;
    #}

    # deny access to .htaccess files, if Apache's document root
    # concurs with nginx's one
    #
    #location ~ /\.ht {
    #    deny  all;
    #}
}
```

## nginx路径匹配和接口转发
### 1.location是否以“／”结尾
在ngnix中location进行的是模糊匹配

- 没有“/”结尾时，location/abc/def可以匹配/abc/defghi请求，也可以匹配/abc/def/ghi等
- 而有“/”结尾时，location/abc/def/不能匹配/abc/defghi请求，只能匹配/abc/def/anything这样的请求

### 2.proxy_pass是否以“／”结尾
 在nginx中配置proxy_pass时，当在后面的url加上了/，相当于是绝对路径，则nginx不会把location中匹配的路径部分加入代理uri；如果没有/，则会把匹配的路径部分加入代理uri。

 - 如果proxy_pass的URL定向里包括URI，那么请求中匹配到location中URI的部分会被proxy_pass后面URL中的URI替换：
```
location /name/ {
    proxy_pass http://127.0.0.1/remote/;
}
```
> 请求http://127.0.0.1/name/test.html 会被代理到http://example.com/remote/test.html

- 如果proxy_pass的URL定向里不包括URI，那么请求中的URI会保持原样传送给后端server：
```
location /name/ {
    proxy_pass http://127.0.0.1;
}
```
> 请求http://127.0.0.1/name/test.html 会被代理到http://127.0.0.1/name/test.html

另外几种情况：
```
location /api/ {
    proxy_pass http://127.0.0.1:3000;
 }
```
> 访问 http://127.0.0.1:80/api/cc, 后端结果为 您的 请求 地址是/api/cc
```
location /api/ {
    proxy_pass http://127.0.0.1:3000/;
 }
```
> 访问 http://127.0.0.1:81/api/cc, 后端结果为 您的 请求 地址是/cc

参考地址：
https://segmentfault.com/a/1190000018933857