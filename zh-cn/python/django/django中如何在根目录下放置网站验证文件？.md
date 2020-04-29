> 现在很多网站的服务需要验证域名权限，需要在网站根目录下放置一个txt文本文件，但是需要能够在域名根目录下直接访问，共有以下几种方法

# 在django中配置
在django中配置的方法有两种
### 第一种方法  
在views.py中新添加一个方法，在该方法中将校验文件中的内容读取出来，通过HttpResponse返回去，然后在urls.py中添加路由`url(r'^baidu-verify-7F2A3DC19B.txt',"刚刚写的方法")`,并传入这个方法名即可在域名根目录下访问这个验证文件了
> 参考链接：http://xieboke.net/article/87/
### 第二种方法
直接在urls.py中设置路由：
`url('MP_verify_eCuMRMZfC1i8hSsw.txt', TemplateView.as_view(template_name='MP_verify_eCuMRMZfC1i8hSsw.txt',content_type='text/plain'))`即可访问，值得注意的是，这里的文件如果直接写名字的话需要在setting中的Templates中的DIRS中填写模板文件的位置，而验证文件在该DIRS配置的路径下；否则，可以填写绝对路径即可
> 参考链接：https://www.cnblogs.com/wuyongcong/articles/10122225.html
# Nginx配置

在Nginx中的配置文件中添加以下配置：
###  第一种配置方法

`
location = /abcdefg.txt {
root /var/xyz/;
}
`

> 使用等号，优先级最高
###  第二种配置方法

`
location /aaa.txt {
    alias /path/to/aaa.txt;
}
`

> root和alias的区别：root代表一个目录，即一个文件的所在目录，不具体到这个文件,如：/home/project/；alias代表绝对路径，具体到这个文件如：/home/project/aaa.txt;