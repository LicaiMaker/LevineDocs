#### 开发准备：
- node.js(安装地址：https://nodejs.org/zh-cn/download/)  

为避免npm下载很慢，可以使用淘宝镜像：
```
 npm install -g cnpm --registry=https://registry.npm.taobao.org
```
- vue-cli  
使用cnpm/npm 下载vue-cli       `cnmp install -g vue-cli`或`nmp install -g vue-cli`

- django2.x(2.2以上)  
注意：暂时不要使用django3.0及以上版本，目前django3.0中的django.utils中的six已经被单独提取出来了，有些依赖库中就使用了django.utils中的six，所以不能使用django3.x

> 官网是这么描述的：==django.utils.six - Remove usage of this vendored library or switch to six.==

#### 流程
- 创建django项目(最好是在python虚拟环境中创建这个项目，如何创建并使用虚拟环境，自行百度)
```python
 django-admin startproject mysite   # 创建mysite项目
```
当然使用pycharm可以不用手动输入这些命令

- 创建vue项目  
在django项目的根目录下执行以下命令：
```
vue-init webpack frontend # frontend为前端项目的名称
```
然后可以一路回车选择默认，也可以根据需要自定义

- 调试vue项目  
在vue项目的目录下执行：  
```shell
cd frontend # frontend为vue项目目录
npm install # 安装需要的依赖模块 
npm run dev # 运行调式的服务，会启动一个web服务，访问localhost:8080 即可调式 
```
> 当然上面的npm也可以换成cnpm    

这样就可以可以看到vue的精美界面了

- vue项目打包  
在vue项目开发完成之后，需要进行打包，打包后会在vue项目中生成一个dist文件夹，里面包了一些的HTML，js,css等文件
```
npm run build        ## 打包vue项目，会将所有东西打包成一个dist文件夹 
```
- 更改django中的设置  
为了django的能使用vue项目，需要更改setting中的一些设置，如模板文件的位置，静态文件的位置
```python
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': ['frontend/dist'],#更改为vue项目的dist目录
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]

STATICFILES_DIRS = [
    os.path.join(BASE_DIR, 'frontend/dist/static'),  # 公共静态资源文件夹
]
```
- 修改django中url文件
```python
urlpatterns = [
    url(r'^admin/', admin.site.urls),
    url(r'^$', TemplateView.as_view(template_name='index.html')),# 指向frontend/dist/index.html
]
```

- 启动django项目
`python manage.py runserver `
或者
`python manage.py runserver 0.0.0.0:8000#局域网类的设备都可访问`