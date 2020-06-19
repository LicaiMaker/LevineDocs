# 安装flask-bootstrap
`pip install flask-bootstrap`

# 初始化

`BootStrap(app)`
# 使用
首先在一个HTML文件中继承bootstrap/base.html文件
```html
    {% extends 'bootstrap/base.html' %}
```
在bootstrap/base.html中有许多block，我们只需要重写这些block即可  
- {% block content %}
- {% block title %}
- {% block metas %}
- {% block navbar %}
...
> 更多block请查看flask-bootstrap官网
> 另外flask-bootstrap中还内置了许多utils,这些都是使用Marco宏定义的文件，位于bootstrap/utils.html,比如使用这文件里的flash_messages方法：
```html
{% from 'bootstrap/utils.html' import flashed_messages %}
```
> 更多使用方式，请查看flask-bootstrap官网