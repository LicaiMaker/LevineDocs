部署以阿里云Ubuntu为例
云服务器的环境搭建详情见“云服务器环境搭建”一文
本文从开始部署准备开始：(记住现在已经是在虚拟环境下操作了)
## 安装Gunicorn和gevent
 `pip install Gunicorn gevent`
Gunicorn (独角兽)是一个高效的Python WSGI Server,通常用它来运行 wsgi application(由我们自己编写遵循WSGI application的编写规范) 或者 wsgi framework(如Django,Paster),地位相当于Java中的Tomcat。

WSGI就是这样的一个协议：它是一个Python程序和用户请求之间的接口。WSGI服务器的作用就是接受并分析用户的请求，调用相应的python对象完成对请求的处理，然后返回相应的结果。

简单来说gunicorn封装了HTTP的底层实现，我们通过gunicorn启动服务，用户请求与服务相应都经过gunicorn传输。
gevent是支持协程的第三方库，具体详情见：https://www.liaoxuefeng.com/wiki/897692888725344/966405998508320

## 编写gunicorn.conf.py文件
在项目的更目录下新建文件gunicorn.conf.py文件，用于保存gunicorn的配置，内容如下：
```python
# gunicorn.conf.py

# 并行工作进程数
workers = 4
# 指定每个工作者的线程数
threads = 2
# 监听内网端口5000
bind = '127.0.0.1:5000'
# 设置守护进程,将进程交给supervisor管理
daemon = 'false'
# 工作模式协程
worker_class = 'gevent'
# 设置最大并发量
worker_connections = 2000
# 设置进程文件目录
pidfile = '/var/run/gunicorn.pid'
# 设置访问日志和错误信息日志路径
accesslog = '/var/log/gunicorn_acess.log'
errorlog = '/var/log/gunicorn_error.log'
# 设置日志记录水平
loglevel = 'warning'
```
## 配置nginx
/etc/nginx/sites-available目录下新建一个配置文件,名字随便取，我的叫oldmaninfo.conf,在文件中输入：
```nginx
server {
    listen      80;
    server_name mustberich.cn;
    charset     utf-8;

    client_max_body_size 75M;

    location /static {
        alias /home/OldManInfo_Flask/frontend/dist/static;
    }

    location / {
        proxy_pass http://127.0.0.1:5000;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;    }

    location /MP_verify_uViqlJwR546DA88L.txt {
        alias /home/OldManInfo_Flask/frontend/dist/MP_verify_uViqlJwR546DA88L.txt;
    }
}

```
> location /中proxy_pass的http://127.0.0.1:5000要与gunicorn.conf.py中的端口号一致

因为sites-available中的文件是虚拟主机，需要在sites-enable中创建一个他的软连接，才能运行这个配置文件；
`ln /etc/nginx/sites-available/oldmaninfo.conf /etc/nginx/sites-enabled/oldmaninfo.conf`
> nginx读取配置的顺序，/etc/nginx/nginx.conf文件包含了sites-enable,所以在sites-available中新建了虚拟主机后，还需要再在sites-enable中新建一个软链接才行
## 启动nginx
先检查nginx语法有没有错误：
`nginx -t`
然后，启动nginx服务：`service nginx start`  

拓展：重启nginx服务是:`service nginx reload`,
停止nginx服务是:`service nginx stop`

## 启动Gunicorn

切换到项目的启动文件目录下，`cd /home/OldManInfo_Flask`,我的启动文件manage.py和gunicorn.conf.py就位于这个目录下
```shell
nohup gunicorn  -c gunicorn.conf.py manage:app &
```
> nohup  命令  &是用于后台挂机执行该命令

>有时候需要重启gunicorn才能生效，重启gunicorn:  1.pstree -ap|grep gunicorn    
>找到第一条中的进程号 2. kill -HUP 进程号
>但是这种命令已经过时，一般在gunicorn.cong.py中配置参数`reload = True`即可在代码改动时自动重启