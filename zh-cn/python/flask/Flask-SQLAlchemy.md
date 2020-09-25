- 数据库的链接参数配置
- 链接数据库
- 增删改查

> Flask-SQLAlchemy这个第三方库是用来数据模型的创建和数据库的增删改查操作的。

## 安装flask-SQLAlchemy
`pip install flask-SQLAlchemy`
## 配置数据参数
```
app.config['SQLALCHEMY_DATABASE_URI']='mysql+pymysql://root:123456@localhost:3306/flaskTest'
```
标准格式是：`'engin+driver://username:password@host:port/databasename'`,即'引擎名+驱动://用户名:密码@主机:端口/数据库的名字'

## 初始化SQLAlchemy对象
```python 
   db = SQLAlchemy()
```

## 创建模型
在models.py中创建模型Person
```python
# 引入SQLAlchemy对象db
import db
# db是前面初始化的SQLAlchemy对象
class User(db.Model):
    id = db.Column(db.Integer, primary_key=True, autoincrement=True)
    name = db.Column(db.String(16))
    password=db.Column(db.String(16))

    def __init__(self, name):
        self.name = name

    def __repr__(self):
        return '<User %s>' % self.name
```
此时可以在python命令行中使用db.create_all()来检查数据库是否可以联通和创建Person表,但是建议使用flask-SQLAlchemy来进行进行数据迁移

## 增删改查

- 1.增加数据（就相当于增加一个实例对象）
```python
user1 = User(name='long',password='123456')

db.session.add(user1)

db.session.commit()
```

- 2.修改数据

修改用户表里面的name为long的姓名为：fang

首先查询到名为long的这个用户

`user1 = User.query.filter_by(name='long').first()`

赋值／修改

`user1.name = 'fang'`

提交

`db.session.commit()`

- 3.先查询删除

```
user1 = User.query.filter_by(name='fang').first()

db.session.delete(user1)

db.session.commit()
```

- 4.查询

1.查询所有用户数据

`User.query.all()`

2.查询有多少个用户

`User.query.count()`

3.查询第1个用户

`User.query.first()`

4.查询id为4的用户[3种方式]
```
(1)User.query.get(4) # 根据主键值来查询

(2)User.query.filter_by(id=4).first()

(3)User.query.filter(User.id==4).first()
```
> 注意:使用model.query.filter_by时需要使用first来获取记录，才能进行后续操作，否则在对查询结果操作时，不能起作用

- 5.查询名字结尾字符为g的所有数据[开始/包含]
```
User.query.filter(User.name.endswith('g')).all()  --[User:wang, User:zhang, User:tang]

包含：

User.query.filter(User.name.contains('g')).all()　　－－[&lt;User 1&gt;, &lt;User 2&gt;, &lt;User 5&gt;]

获取第二个对象的名字：

list = User.query.filter(User.name.contains('g')).all()

list[1].name
```
- 6.查询名字不等于wang的所有数据[2种方式]
```
(1)!=: User.query.filter(User.name!='wang').all()

(2)not:User.query.filter(not(User.name=='wang')).all()
```
- 7.查询名字和邮箱都以 li 开头的所有数据[2种方式]
```
(1)and: User.query.filter(and(User.name.startswith('li'),User.email.startswith('li'))).all()

(2)不需要and_:User.query.filter(User.name.startswith('li'),User.email.startswith('li')).all()

```

- 8.查询password是 123456 或者 email 以 itheima.com 结尾的所有数据
```
User.query.filter(or_(User.password=='123456',User.email.endswith('itheima.com'))).all()
```
- 9.查询id为 [1, 3, 5, 7, 9] 的用户列表
```
 User.query.filter(User.id.in_([1,3,5,7,9])).all()
```
- 10.查询name为liu的角色数据（重要）
```
User.query.filter(User.name=='liu').first().role.name
```
- 11.查询所有用户数据，并以邮箱排序
```
User.query.order_by('email').all()
```
- 12.每页3个，查询第2页的数据
```
User.query.paginate(2,3,False).items  查询数据

User.query.paginate(2,3,False).page  ---当前页

User.query.paginate(2,3,False).pages ---总页数
```

> 常用筛选关键字  

> filter和filter_by:
> filter用类名.属性名，比较用==，filter_by直接用属性名，比较用=
> 不过这个是语法小细节
```
session.query(MyClass).filter(MyClass.name == 'some name')
session.query(MyClass).filter_by(name = 'some name')
```

> 补充：模型一对多的关系与relationship的用法以及与django中GenericRelations的用法
### flask中模型中的而一对多的关系建立
```python
class Person(db.Model):
    p_id = db.Column(db.Integer, primary_key=True, autoincrement=True)
    p_name = db.Column(db.String(16))
    addresses = db.relationship('Address', backref='person', lazy=True)


# 地址
class Address(db.Model):
    a_id = db.Column(db.Integer, primary_key=True, autoincrement=True)
    a_detail = db.Column(db.String(16))
    person_id = db.Column(db.Integer, db.ForeignKey('person.p_id',ondelete='CASCADE'))# 这里使用ondelete属性设置级联删除
```
一个Person对应多个地址，先往数据库中插入一条Person数据Person('张三')，再向数据库中插入两条Address数据：Address('北京市'，1)，Address('深圳市'，1)，这两条数据都是和Person('张三')关联的。
> 那么如何查找张三所对应的多条地址信息呢？
#### 传统方法
```python
    p=Person.query.get(1)# 张三的p_id=1
    address=Address.query.filter_by(person_id=p.p_id)
```
> 注意：使用Person.query.get()的方式只能适用于用主查询，而且是直接传主键值即可，不能填写pid=12这种而是直接传Person.query.get(12)就行
#### flask中使用relationship
首先在Person中需要和Address建立关系
```python
class Person(db.Model):
    p_id = db.Column(db.Integer, primary_key=True, autoincrement=True)
    p_name = db.Column(db.String(16))
    addresses = db.relationship('Address', backref='person', lazy=True)
```
> db.relationship() 做了什么？这个函数返回一个可以做许多事情的新属性。在本案例中，我们让它指向 Address 类并加载多个地址.  

>那么 backref 和 lazy 意味着什么了？backref 是一个在 Address 类上声明新属性的简单方法。您也可以使用 my_address.person 来获取使用该地址(address)的人(person)。

> lazy 决定了 SQLAlchemy 什么时候从数据库中加载数据:
> 'select' (默认值) 就是说 SQLAlchemy 会使用一个标准的 select 语句必要时一次加载数据。
> 'joined' 告诉 SQLAlchemy 使用 JOIN 语句作为父级在同一查询中来加载关系。
> 'subquery' 类似 'joined' ，但是 SQLAlchemy 会使用子查询。
> 'dynamic' 在有多条数据的时候是特别有用的。不是直接加载这些数据，SQLAlchemy 会返回一个查询对象，在加载数据前您可以过滤（提取）它们。详细信息，查看官网：[http://www.pythondoc.com/flask-sqlalchemy/models.html#one-to-many](http://www.pythondoc.com/flask-sqlalchemy/models.html#one-to-many)

那么获取addresses的方式就是：
```python
    p=Person.query.get(1)# 张三的p_id=1
    address=p.addresses
```
并且，还可以同伙address获取person：
```python
    a=Address.query.first()
    a.person #这里的person是上面relationship中backref的值
```
> 注意；在flask中获取一对多关系的数据时，获取到的是多个对象的集合，而非数据(与django不同)，所以需要使用对象里面的属性值时，可以采用列表生成式将其属性值读取出来：`[a.a_detail for a in p.address ]`a.address获取多个Address对象，需要Address里面的a_detail属性值

> 另外，需要注意的一个大问题是，lazy的值，lazy的值设为True的话，上面的列子中p.address可能会为空，默认情况下lazy=select(针对数据量小时建议使用，否则使用dynamic)，即在查询时即可检出，lazy=joined在两张表关联时，可获取数据，lazy=dynamic时，返回的是一个query对象,需要执行相应方法才可以获取对象，比如.all()获取所有数据，更多详情见：https://shomy.top/2016/08/11/flask-sqlalchemy-relation-lazy/

#### django的做法
下面是一个阅读量模型
```python 
class ReadNum(models.Model):
   readed_num=models.IntegerField(default=0)
   content_type = models.ForeignKey(ContentType, on_delete=models.DO_NOTHING)#在这里不使用CASCADE
   object_id = models.PositiveIntegerField()
   content_object = GenericForeignKey('content_type', 'object_id')
```
使用GenericForeignKey来关联，并让与之关联的模型继承下面的类即可获取对应的阅读量：
```python 
 为了read_statistics的完整使用，我们将该方法提取到一个类ReadNumExpandMethod中，并将该类放到read_statistics的models.py文件中，并且让blog\models.py文件中的Blog模型继承该类,则可以使用里面的get_readed_num方法
	    
	    #read_statistics\models.py
	    class ReadNumExpandMethod:
		    def get_readed_num(self):
		        try:
		            ct=ContentType.objects.get_for_model(self)
		            readnum=ReadNum.objects.get(content_type=ct, object_id=self.pk)
		            # print("number:",readnum.readed_num ,self.pk,ct)
		            return readnum.readed_num   
		        except exceptions.ObjectDoesNotExist as e:
		            return 0  

		#在blog\models.py中的Blog类继承 ReadNumExpandMethod                            	     
		class Blog(models.Model,ReadNumExpandMethod):
```

### flask-sqlalchemy中增删改查须知
因为在flask-sqlalchemy中的model对象没有save，delete方法，这些方法都内置在了SQLAlchemy对象中，需要使用SQLAlchemy对象来进行增删改查，可以写一个基础的Base类，提供save和delete等方法，让所有的model继承这个Base类，则这些model对象页拥有了save,delete方法：
```python
# 创建一个基类，用于对象的CRUD操作
class Base:
    def save(self):
        try:
            db.session.add(self)  # self实例化对象代表就是u对象
            db.session.commit()
        except Exception as e:
            print("name:%s,exception:$s"%self.__repr__(),e)
            db.session.rollback()

    # 定义静态类方法接收List参数
    @staticmethod
    def save_all(List):
        try:
            db.session.add_all(List)
            db.session.commit()
        except:
            db.session.rollback()

    # 定义删除方法
    def delete(self):
        try:
            db.session.delete(self)
            db.session.commit()
        except:
            db.session.rollback()
            
            
class Test(db.Model,Base):
    __tablename__='test'
    name = db.Column(db.String(20),primary_key=True)    
    
    def __init(self,name):
        self.name=name
    
# 示例    
    
t =Test(name='haha')
t.save()
```
> 但是，上面的使用db.session.commit()方法，表示将当前的事务结束（先调用db.session.flush()然后结束当前事务）,如果出现问题，rollback只会回滚最后一次commit，前面的已提交的就不能一起回滚了，比如：
```
try:
    model1=Model1(id=1,....)
    db.add(model1)
    db.session.commit()
    # 此时产生了某种异常...
    model2=Model2(....)
    db.add(model2)
    db.session.commit()
except Exception as e:
    db.session.rollback()
```
> 此时上面产生了异常后，虽然会执行db.session.rollback()，但是model1已经写入数据库，不可回滚了，所以，这里要求所有回滚是无效的，修改如下：
```
try:
    model1=Model1(id=1,....)
    db.add(model1)
    db.session.flush()
    # 此时产生了某种异常...
    model2=Model2(....)
    db.add(model2)
    db.session.flush()
    # 最后执行commit
    db.session.commit()
except Exception as e:
    db.session.rollback()
```
> 上面使用了flush函数，并不会结束事务，所以可以回滚，所以将上面的base类改写：
```
# 创建一个基类，用于对象的CRUD操作
class Base:
    def save(self):
        try:
            db.session.add(self)  # self实例化对象代表就是u对象
            db.session.flush()
        except Exception as e:
            print("name:%s,exception:$s"%self.__repr__(),e)
            db.session.rollback()

    # 定义静态类方法接收List参数
    @staticmethod
    def save_all(List):
        try:
            db.session.add_all(List)
            db.session.flush()
        except:
            db.session.rollback()

    # 定义删除方法
    def delete(self):
        try:
            db.session.delete(self)
            db.session.flush()
        except:
            db.session.rollback()
```
> flush和commit之后都能够拿到对象的id，因为已经写入数据库了，只是执行flush时是可回滚的，而commit，提交当前事务，只能回滚最后一次commit，而且commit本质上也是执行了flush方法的