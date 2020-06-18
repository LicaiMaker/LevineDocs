> 引入，在django中拥有优秀的orm映射工具，通过 
> `python manage.py makemigrations`  
> `python manage.py migrate`  
> 两条命令就能迁移数据库，对数据库进行相应的更改，那么在flask中怎么进行数据库迁移呢？


# 安装flask-migrate
`pip install flask-mgrate`
# 实例化flask-migrate插件
`migrate=Migrate(app=app,db=db)`
这里的db是SQLAlchemy实例
# 使用flask-migrate
### 1.初始化
- 基本使用：`flask db init`，
- 配合flask-script使用，在manage.py中添加：
```python 
from flask_migrate import MigrateCommand
from flask_script import Manager

import App

app = App.create_app()
manager = Manager(app)
# 添加这句，使用flask-script插件
manager.add_command('db', MigrateCommand)

if __name__ == '__main__':
    manager.run()

```
此时使用`python manage.py db init`(这是初始化指令，只调用一次)，
然后就会发现项目中多了一个migrations文件夹
### 2.生成迁移文件并执行数据库更改
- 基本使用 `flask db migrate`
- 配合flask-script `python manage.py db migrate`

> 可以在后面加参数 --message '创建Student模型'  

此时在migrations/verions中会增加一个python文件(每次迁移后生成的文件通过链表的形式关联reverion,downversion)，点击查看里面的内容，会发现有一个upgrade方法和downgrade方法，里面是对应的数据更新语句，执行upgrade方法downgrade方法数据库则会进行相应的更改：
- ```python manage.py db upgrae```
- ```python manage.py db downgrade```
> 注：执行upgrade方法和downgrade方法有点类似django中`python manage.py migrate`  

>命令解释：
>python manage.py db init 只调用一次，初始化指令

> python manage.py db migrate --message 生成迁移文件，内部迁移文件通过链表关联(版本号),message表示迁移日志，下同

> python manage.py db upgrade --message 执行迁移文件，数据库内容升级

> python manage.py db downgrade --messsage 执行迁移文件，数据库内容降级，这世界还是有后悔药的，

> python manage.py db --help

---
> 另外，怎么在pycharm中查看数据的更改呢？以mysql为例：
> 点击右侧的Database，然后点击绿色的加号，然后选择DataSource -> MySql,DataBase,里面的DataBase填写为你是用的数据库名称(比如我在mysql中新建了一个叫flaskTest的数据库，则此处填写flaskTest),user,password填写好就点击TestConnection,可能会让你装驱动，测试通过之后就点击apply然后确定就好了，在Database中就能看到@localhost实例，双击schemas就能看到表了，如果看不到就邮件点击一下数据库名，点击DataBaseTool就，点击第一个Manage Shown Schemas,选择你要展示的数据库，再刷新一下就可以了


> 非常注意：在修改模型的字段的长度后，这时是检测不到数据库模型有更改的，需要在生成migration/env.py中添加一句`compare_type=True`即可：
```
  with connectable.connect() as connection:
        context.configure(
            connection=connection,
            target_metadata=target_metadata,
            compare_type=True,
            process_revision_directives=process_revision_directives,
            **current_app.extensions['migrate'].configure_args
        )

```
> 参考链接：[链接1](http://blog.sina.com.cn/s/blog_1417f234e0102wp6i.html)
> [链接2](https://segmentfault.com/q/1010000020709691)