# 新建项目注意事项
### - 使用flask-script启动项目 
`pip install flask-script`
引入flask_script中的Manager，并用app对象初始化Manager:
```python
from flask import Flask

app = Flask(__name__)

"""使用flask-script启动项目"""
from flask_script import Manager
manager = Manager(app)

if  __name__ == "__main__"
    manager.run()
```
并将这个文件更名为*manage.py* 
然后可以在命令行中使用：
```shell
python manage.py runserver -r -d
```
>拓展：
>`-r`的意思是在项目有改动时，保存后会自动重启;
>`-d`的作用的开启debug mode；
>`-h` 指定主机；
>`-p` 指定端口
>`-threaded` 是否使用多线程 

### - 使用蓝图构建项目
蓝图是为了拆分views，代替了app注册路由的问题
`from flask import Blueprint`
1. 在App/views.py中使用蓝图：
```python
    from flask import Blueprint
    main=Blueprint('main',__name__)

    @main.route('/')
    def hello_world():
        return 'Hello World!'
```
2.在*manage.py*中注册蓝图  

```python
    from flask import Flask
    from flask_script import Manager
    from App.views import *
    app = Flask(__name__)
    #注册蓝图
    app.register_blueprint(main)

    manager = Manager(app)

    if __name__ == '__main__':
        manager.run()

```
> 注：假如不同的views.py中拥有相同的方法名home，这两个views.py分别对应的蓝图名称是first_blue和second_blue，第一的views.py中调用home方法，如下：
> ```python
> # views1.py
> def test():
> return redirect(url_for('home'))
> ```
> 此时会报错，因为flask不知道你要使用那个蓝图中> 的home方法，所以需要在方法名前添加蓝图名,如：
> ```python
> # views1.py
> def test():
> return redirect(url_for('first_blue.home'))
> ``  
> ```

### - flask中render_template
flask中可以使用render_template渲染一个模板文件，如：
```html
return render_template('index.html')
```
但是如果涉及到设置cookie等需要response对象的时候，就不能这么用，需要使用make_response进行包裹：
```html
res= make_response(render_template('index.html'))
res.set_cookie('username','root')
return res
```
> 注意：在有些时候，没有涉及返回对象时，也需要使用make_response进行包裹才能渲染出页面

### - 拓展：配置静态文件和模板文件的位置
如果在flask项目中，我们的模板文件和静态文件不在默认的根目录下的templates和static文件夹中，则需要手动指定.
```python
app = Flask(__name__, static_folder=settings.STATIC_FOLDER, template_folder=settings.TEMPLATE_FOLDER)
```
我这里的settings是我的配置文件，位于project_path/App/settings.py,name如何在settings.py中配置这两个路径呢？
看我的settings.py文件：
```python
# 配置BASE_DIR和模板文件的路径
import os

BASE_DIR = os.path.dirname(os.path.dirname(os.path.abspath(__file__)))

TEMPLATE_FOLDER = os.path.join(BASE_DIR, 'frontend/dist')
STATIC_FOLDER = os.path.join(BASE_DIR, 'frontend/dist/static')
```
> BASE_DIR是通过__file__对象获取到的根目录地址，__file__是当前文件的路径，frontend/dist是我的模板文件所在位置，frontend/dist/static是我的静态文件的位置