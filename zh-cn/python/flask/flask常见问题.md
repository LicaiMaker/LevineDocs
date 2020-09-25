[TOC]
### 1.Python3出现UnicodeEncodeError: 'ascii' codec can't encode characters问题
这是由于Python环境字符集不是utf-8导致的，所以遇到中文时就不能编码了，所以现在需要将
字符集编程设为UTF-8
- 1.第一种：改变系统的环境变量
`echo 'export LANG=en_US.UTF-8' >> ~/.bashrc`
然后重新登录即可
- 2.第二种：改变Python环境变量
在python的Lib\site-packages文件夹下新建一个sitecustomize.py，内容为：
```
# encoding=utf8  
import sys  
from importlib import reload # 这句在Python2中不需要
reload(sys)  
sys.setdefaultencoding('utf8')   
```
然后重启Python解释器

> 查看当前的Python环境的字符集：`python -c "import sys; print(sys.stdout.encoding)"`,注意，以下代码返回UTF-8并不能说明没有问题:`python -c "import sys; print(sys.getdefaultencoding())"`

> 参考链接：1.https://blog.csdn.net/chenlide683424/article/details/100914599?utm_medium=distribute.pc_relevant.none-task-blog-baidujs-5   2.https://blog.csdn.net/icyfox_bupt/article/details/103300826?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.nonecase&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.nonecase  3.https://blog.csdn.net/lezeqe/article/details/85677822

### 2.flask-sqlalchemy中如何查询某个表中的某个记录是否存在？
> 假设我定义了一个模型PhoneNumber:
```
class PhoneNumber(db.Model, Base):
    __tablename__ = 'phonenumber'
    id = db.Column(db.Integer, primary_key=True, autoincrement=True)
    phoneNumber = db.Column(db.String(50))


    def __init__(self, phoneNumber, qId):
        self.phoneNumber = phoneNumber

```
现在查询PhoneNumber中某条记录是否存在：
```
# 在SQL中的语句应该是：
select exists(select * from phonenumber where phonenumber.phoneNumber = 'xxx') as a1;
```
```
# 在flask中：
# 单条件
exists = db.session.query(db.exists().where(PhoneNumber.phoneNumber =='xx')).scalar()
# 多条件
exists = db.session.query(db.exists().where(PhoneNumber.phoneNumber =='13550217062', id == 7)).scalar()
```
> 我把这个方法封装起来，任何模型都能使用:
```
# 这里使用text函数来将字符串转为真正的条件
from sqlalchemy import text 
from flask_sqlalchemy import SQLAlchemy

db = SQLAlchemy()
db.init_app(app)# app为Flask对象
# 根据条件,查询单表中是否存在某个记录
def isRecordExist(model, condition):
    q = db.session.query(model).filter(text(condition))
    exists = db.session.query(
        q.exists()
    ).scalar()
    return exists
```
> 调用
```
# 注意：这里调用的条件格式不一样，因为这里的条件传入的是字符串格式
# 单条件
exists = isRecordExist(PhoneNumber, "phoneNumber = '13550217062' )# 这里是单等号
# 多条件
exists = dbutils.isRecordExist(PhoneNumber, "phoneNumber = '13550217062' and  id = '7'") # 多条件用and连接
# 所以这里调用的格式和上面直接使用是条件的格式是有区别的
```
> 注意里面的varchar类型的参数需要用引号括起来

### 3.在使用flask_script的情况下，如何更改默认的flask启动端口？
我们使用flask_script启动项目，使用的命令是：
```
python manage.py runserver
```
> 注意：这里的runserver，其实就是flask_script实现的Server,默认端口是5000，如果想更改默认端口，则只需要重写Server即可：
```
from flask_script import Manager, Server
import App

app = App.create_app() # flask实例
manager = Manager(app)
# 重新定义端口,runserver其实就是用的这个命令
server = Server(port=5001)
# 替换runserver命令
manager.add_command('runserver', server)


if __name__ == '__main__':
    manager.run()

```