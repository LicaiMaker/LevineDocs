在*django*项目中，我们会使用*settings.py*中定义一些参数，比如：  

```python
SECRET_KEY = '!pv4o62*84k9s(u$y%k5$e8vura2_w)y5aa^-(e@4hk&'
DEBUG = True
ALLOWED_HOSTS =['*']
GITHUB_WEBHOOK_KEY='9tXDogwdEUBLJwwgczgfaeNAAbQDXIDo4Yth'
WX_APP_SERCRET='b07f584b45cafc8ba28909211'
```
并且这些参数会随着*settings.py*上传到*github*上，这样很不安全，故我们不能将这些参数上传，所以只能想办法将这些机密参数从*settings.py*中分离出来。
在*python*中提供了*python-decouple*:

## 1.*在Django*项目的虚拟环境中安装*python-decouple*
```python  
pip install python-decouple
```


## 2.添加*.env*或者*.ini*文件，并把配置信息写入这个文件
参考目录：*https://pypi.org/project/python-decouple/*
因为我使用的是*.env*文件，创建*.ini*文件的可以参考上面的链接
### a.在*Django*项目的根目录下，创建*.env*文件，添加配置信息：
```python  
SECRET_KEY = !pv4o62*84k9s(u$y%k5$e8vura2_w)y5aa^-(e@4hk&
DEBUG = True
ALLOWED_HOSTS =*
GITHUB_WEBHOOK_KEY=9tXDogwdEUBLJwwgczgfaeNAAbQDXIDo4Yth
WX_APP_SERCRET=b07f584b45cafc8ba28909211
```
>注：不能在每个字符串上加''
### b.将*.env*添加到*.gitignore*
因为不能将这个文件传到服务器上，所以必须在*.gitignore*中添加*/.env*
*/.env*

>注：如果*.gitignore*文件不能生效(*commit*时仍然将*.env*文件提交了)，此时可以执行：

```python  
#cmd 进入.gitignore所在的目录下，删除缓存
git rm -r --cached .
git add .
git commit -m 'update .gitignore'
```
> *.gitignore* 只能忽略那些原来没有被追踪的文件，如果某些文件已经被纳入了版本管理中，则修改*.gitignore*是无效的。那么解决方法就是先把本地缓存删除（改变成未被追踪状态），然后再提交.


## 3.在settings.py中调用.env中的变量
```python  
from decouple import config,Csv
#SECURITY WARNING: keep the secret key used in production secret!
SECRET_KEY = config('SECRET_KEY')

#SECURITY WARNING: don't run with debug turned on in production!
DEBUG = config('DEBUG',cast=bool)

ALLOWED_HOSTS = config('ALLOWED_HOSTS', cast=Csv())
#微信小程序相关参数
WX_APP_SERCRET=config('WX_APP_SERCRET')
#github webhook key
GITHUB_WEBHOOK_KEY=config('GITHUB_WEBHOOK_KEY')
```
*config*中有个参数 cast,在官方文档上是这么写的：
```python  
By default, all values returned by decouple are strings, after all they are read from text files or the envvars.
However, your Python code may expect some other value type, for example:
    Django’s DEBUG expects a boolean True or False.
    Django’s EMAIL_PORT expects an integer.
    Django’s ALLOWED_HOSTS expects a list of hostnames.
    Django’s SECURE_PROXY_SSL_HEADER expects a tuple with two elements, the name of the header to look for and the required value.
```
*cast*表示相应的返回值类型，但只能返回常用的基本类型，比如要想返回一个列表，那只能自定义一个*lambda*函数:
```python  
config('ALLOWED_HOSTS', cast=lambda v: [s.strip() for s in v.split(',')])
```

但是这个写法太复杂了，在decouple中已经定义好了相应的函数*Csv()*：
```python  
ALLOWED_HOSTS = config('ALLOWED_HOSTS', cast=Csv())
```
> 参考链接：*https://pypi.org/project/python-decouple/#id12*

## 4.提交本地修改到*github*,然后服务器从github上*pull*代码
在提交本地代码之前，先将本地的虚拟环境的配置导出来，早*pycahrm*中打开*Terminal*，此时显示已经是在项目的根目录下，并且在虚拟环境中，执行
```python  
pip freeze >requirements.txt
```
这样就生成了配置清单，并随后将会被上传值*github*
```python  
git add .
git commit -m 'update'
git push
```
登录服务器，将*github*刚刚更新的代码*pull*下来，进入虚拟环境，然后进入*Django*项目的根目录下，看到*requirements.txt*文件，执行：
```python  
pip install -r requirements.txt
```
导入安装更新的包。

## 5.将*.env*直接上传到服务器

因为已经将相应的变量通过*decouple*从*settings.py*中分离出来了，不用上传*github*，但是服务器上需要使用这些变量，故需将*.env*文件上传至服务器的相应目录，保证了这些数据的安全性。
使用*scp*命令将本地文件上传至阿里云服务器：
```javascript  
scp 本地文件 登录名@服务器ip地址:Django项目根目录下
```
比如我的上传*.env*文件到服务器的*Django*项目根目录下的命令：
```javascript  
scp /e/mysite/.env root@182.92.119.50:/home/mysite/
```


重启服务器就可以了
```javascript  
service apache2 restart
```

