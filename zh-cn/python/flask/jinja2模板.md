# block
挖坑和填坑
比如base.html:
```html
<head>
	<title>{%block title%} {%endblock%}</title>

	<script type="text/javascript" src="{% static 'bootstrap-3.3.7/js/bootstrap.min.js' %}"></script>
	{%block header_extends%}{%endblock%}
</head>
```
那么在home..html中继承base.html:
```html
{% block title %}
    我的文章|首页
{% endblock %}

{% block header_extends %}
    <script src="https://cdn.hcharts.cn/highcharts/highcharts.js"></script>
    <link rel="stylesheet" type="text/css" href="{% static 'home.css' %}">
{% endblock %}
```
# 继承
在上面的继承中如果使用父模板中block里面的内容，则可以使用`{{super()}}`即可保留

# marco
一般使用宏定义的方式,具体方法是新建一个HTML文件，在里面使用marco定义一些方法：
```html
    # mac.html
    {% marco sayHello(name)%}
        <p>你好啊，{{name}}</p>
    {% endmarco%}
```
然后在其他的HTML文件中引入这个方法：
```html
    # main.html
    {% from 'mac.html' import sayHello%}
    {{ sayHello('jinja2') }}
```