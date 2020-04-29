> 为了更好的使用flask，实现低耦合，将项目进行拆分

## 准备材料
- flask-script
- flask-SQLAlchemy
- flask-migrate
## 项目拆分后的目录结构
 新建一个python-package，取名叫App,我们所要做的就是在App/__init__.py文件中添加方法`create_app`,并在该方法中完成以下内容：
- 1.Flask项目的初始化
- 2.加载配置项
- 3.SQLAlchemy等第三方插件的初始化
- 4.注册蓝图

App包下的目录结构：
```
- App
 -- __init__.py
 -- ext.py
 -- models.py
 -- settings.py
 -- views.py
```
其中在__init__.py中的内容为：
```python
    
from flask import Flask
from App import views
from App.ext import init_ext
from App.settings import envs


def create_app():
    # 创建flask对象
    app = Flask(__name__)
    # 从settings中加载配置
    app.config.from_object(settings.envs.get('develop'))
    # 初始化第三方库，插件
    init_ext(app)
    # 注册蓝图
    views.init_blue(app)
    return app
```
在create_app中可以看到与settings(项目配置文件)，ext(第三方库的初始化文件)产生了关联，并且这个create_app方法返回了一个Flask对象，这个方法将会在flask项目的启动文件`app.py`中被调用，详情况，见下面。

## 编写App下各个文件
 #### 1.settings.py  
 这个文件主要是用于配置Flask和第三方库所需要的的配置项，如secret_key，数据uri等信息，在配置文件里，一般会写四个环境即，开发环境(DevelopConfig),测试环境(TestingConfig),演示环境(StagingConfig),正式环境(ProductConfig).
 ```python
 # settings.py
 # 将json格式的数据库参数转化为标准的数据库URI
def parse_db_uri(db_info):
    engine = db_info.get('ENGINE') or 'mysql'
    driver = db_info.get('DRIVER') or 'pymysql'
    user = db_info.get('USER') or 'root'
    password = db_info.get('PASSWORD') or 'root'
    host = db_info.get('HOST') or 'localhost'
    port = db_info.get('PORT') or '3306'
    name = db_info.get('NAME') or 'test'
    return '{}+{}://{}:{}@{}:{}/{}'.format(engine, driver, user, password, host, port, name)


# 基础参数配置(公共参数)
class Config:
    DEBUG = False

    TESTING = False

    SECRET_KEY = '12345lkjhaqwea'

    SQLALCHEMY_TRACK_MODIFICATIONS = False


# 开发环境参数配置
class DevelopConfig(Config):
    DEBUG = True
    # 注意：不同环境下的数据库名称NAME应该不一样
    DATABASE = {
        'ENGINE': 'mysql',
        'DRIVER': 'pymysql',
        'USER': 'root',
        'PASSWORD': '123456',
        'HOST': 'localhost',
        'PORT': '3306',
        'NAME': 'PythonFlaskModel'
    }
    SQLALCHEMY_DATABASE_URI = parse_db_uri(DATABASE)


# 测试环境参数配置
class TestingConfig(Config):
    TESTING = True
    DATABASE = {
        'ENGINE': 'mysql',
        'DRIVER': 'pymysql',
        'USER': 'root',
        'PASSWORD': '123456',
        'HOST': 'localhost',
        'PORT': '3306',
        'NAME': 'PythonFlaskModelTesting'
    }
    SQLALCHEMY_DATABASE_URI = parse_db_uri(DATABASE)


# 演示环境参数配置
class StagingConfig(Config):
    DATABASE = {
        'ENGINE': 'mysql',
        'DRIVER': 'pymysql',
        'USER': 'root',
        'PASSWORD': '123456',
        'HOST': 'localhost',
        'PORT': '3306',
        'NAME': 'PythonFlaskModelStaging'
    }
    SQLALCHEMY_DATABASE_URI = parse_db_uri(DATABASE)


# 演示环境参数配置
class ProductConfig(Config):
    DATABASE = {
        'ENGINE': 'mysql',
        'DRIVER': 'pymysql',
        'USER': 'root',
        'PASSWORD': '123456',
        'HOST': 'localhost',
        'PORT': '3306',
        'NAME': 'PythonFlaskModelProduct'
    }
    SQLALCHEMY_DATABASE_URI = parse_db_uri(DATABASE)


envs = {
    'develop': DevelopConfig,
    'testing': TestingConfig,
    'staging': StagingConfig,
    'product': ProductConfig,
}

 ```
 这四种环境放在`envs`变量中，并在__init__.py中的create_app()中将需要的环境配置引入即可：
```python 
# __init__.py
# 从settings中加载配置
app.config.from_object(settings.envs.get('develop'))
```

 #### 2.ext.py 
 这个文件用于初始化第三方库
 ```python
from flask_sqlalchemy import SQLAlchemy
from flask_migrate import Migrate

db = SQLAlchemy()
migrate = Migrate()


def init_ext(app):
    # 初始化第三方插件
    db.init_app(app)
    migrate.init_app(app=app, db=db)
 ```
在该文件中，各个第三方插件的实例化均采用懒加载模式，延迟初始化，这种方式是为了避免循环应用的问题，在__init__.py中的crate_app方法中调用init_ext方法即可初始化这些插件

#### 3.views.py
views.py是用于编写视图函数的，在这里面需要用到蓝图这个这个东西

```python
# views.py
from flask import Blueprint,

main = Blueprint('main', __name__)


def init_blue(app):
    app.register_blueprint(blueprint=main)


@main.route('/')
def hello_world():
    return 'hello,world!'
```
在views.py中需要初始化一个蓝图对象，并在其中添加init_blue方法,这个方法是用于注册蓝图，同样采用懒加载的模式，并在__init__.py中调用这个方法：
```python
from flask import Flask
from App import views

def create_app():
    app = Flask(__name__)
    ...
    # 注册蓝图
    views.init_blue(app)
    return app
```

#### models.py
models.py用于模型的创建，这里需要引入ext.py中的SQLAlchemy对象db来进行字段的声明
```python
from .ext import db

class Person(db.Model):
    p_id = db.Column(db.Integer, primary_key=True, autoincrement=True)
    p_name = db.Column(db.String(16))

    def __init__(self, p_name):
        self.p_name = p_name

    def __repr__(self):
        return '<Person %s>' % self.p_name
```
---
> APP已拆分完毕，对比在pycahrm中生成的Flask模板项目，可以发现，现在拆分后的项目的Flask初始化已经不在根目录下的app.py中了，而是放在APP/__init__.py中，为什么要这么做呢？第一，是因为我们想将这个app.py变成django中的manage.py那样的只用来接受命令行参数来启动的文件，那么就不应该将内容繁多的参数的初始化操作放在这个启动文件中；第二，拆分文件是为了更好的维护，项目越来越大，拆分文件就很有必要了

下面就来改造一下原来的app.py文件了
## 改造app.py
#### 1. 重命名app.py为manage.py
为了区分一下App目录，所以改了个和django中的启动文件一样的名字manage.py

#### 2.使用flask-script中Manager.py启动项目
原来的app.py内容为：
```python
app = Flask(__name__)

@app.route('/')
def index():
    return 'hello,world!'

if __name__ == '__main__' :
    app.run()
```
更改后的app.py(manage.py)
```python
from flask_script import Manager

import App

app = App.create_app()
manager = Manager(app)

if __name__ == '__main__':
    manager.run()
```
这里使用了flask-script这个库，使用里面的Manager来启动项目，主要为了在命令行接受各种参数而采用了Manager，这样就成功改造了flask项目。  

项目启动方式：
`python manage.py runserver -r -d -port 8000`
（后面的参数可选）