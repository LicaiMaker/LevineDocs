## restful概念

## restful设计6原则
- uniform interface 统一接口
- stateless 无状态
- Cacheable 可缓存 减少服务器的请求压力
- client-server c-s模式 服务端客户端分离，客户端不包括数据，服务端不包括用户状态，两端都可以任意的升级，并增加服务的稳定性
- layered system 分层系统
- code on demand 按需编码


## flask-restful的使用
#### 1.安装flask-restful
```python 
    pip install flask-restful
```
#### 2.定义Resource类
需要自定义类并继承Resource类：
```python
from flask import render_template
from flask_restful import Resource
class Index(Resource):
    def get(self):
        return render_template('index.html')
```
注：如果需要可以定义get,put,post,deleted等方法来处理不同类型的请求
```python
class index(Resource):
    def get(self):
        pass
    def post(self):
        pass
```

#### 3.reqparse的使用
reqparse是用来对请参数的验证，如对参数的类型验证，是否为空验证，和设置提示信息等
- a.基础使用
```python
from flask_restful import reqparse

parser = reqparse.RequestParser()
parser.add_argument('rate', type=int, help='Rate cannot be converted')
parser.add_argument('name')
args = parser.parse_args()
```
> 使用type来指定类型，help指定数据不合法的提示信息
- b.required参数
```python
parser.add_argument('name', required=True,
help="Name cannot be blank!")
```
- c.action参数
当传过来的某个键有多个值时，如果需要将这个键对应的多个值整合成一个列表，则可以将action参数设为append即可。
```python
parser.add_argument('name', action='append')
```
> 你的请求：`curl http://api.example.com -d "name=bob" -d "name=sue" -d "name=joe"`
- d.dest参数
dest参数用于为参数指定别名，在解析之后，获取参数的值时使用这个别名获取
```python
parser.add_argument('name', dest='public_name')
args = parser.parse_args()
args['public_name']
```
- e.location参数
location参数用于指定参数解析器从request中的哪个参数获取。
```python
parser.add_argument('name', type=int, location='form')

# Look only in the querystring
parser.add_argument('PageSize', type=int, location='args')

# From the request headers
parser.add_argument('User-Agent', location='headers')

# From http cookies
parser.add_argument('session_id', location='cookies')

# From file uploads
parser.add_argument('picture', type=werkzeug.datastructures.FileStorage, location='files')
```
> 注意：当`location`的值为==json==时，需要使用`type=list`   

另外，location可以使用多个值来获取多个参数：
`parser.add_argument('text', location=['headers', 'values'])`
- f.bundle_errors参数
这个参数是将多个参数报错信息一并检查并返回，而不是遇到第一个参数不合法就返回报错信息
```python
from flask_restful import reqparse

parser = reqparse.RequestParser(bundle_errors=True)
parser.add_argument('foo', type=int, required=True)
parser.add_argument('bar', type=int, required=True)

# If a request comes in not containing both 'foo' and 'bar', the error that
# will come back will look something like this.

{
    "message":  {
        "foo": "foo error message",
        "bar": "bar error message"
    }
}

# The default behavior would only return the first error

parser = RequestParser()
parser.add_argument('foo', type=int, required=True)
parser.add_argument('bar', type=int, required=True)

{
    "message":  {
        "foo": "foo error message"
    }
}
```
> 当然，如果我们在多个请求中都使用了bundle_errors这个参数，则可以全局设置:
> ```
> from flask import Flask
> app = Flask(__name__)
> app.config['BUNDLE_ERRORS'] = True
> ```
> 但是，需要注意的是，这个全局参数会 覆盖本地RequestParser中的bundle_errors参数

另外，可以在通过插值的方法，进自己的信息和类型错误拼接，在保留原有错误信息的基础上自定义信息：
```python
from flask_restful import reqparse

parser = reqparse.RequestParser()
parser.add_argument(
    'foo',
    choices=('one', 'two'),
    help='Bad choice: {error_msg}'
)

# If a request comes in with a value of "three" for `foo`:

{
    "message":  {
        "foo": "Bad choice: three is not a valid choice",
    }
}
```
> 更多信息，查看：https://flask-restful.readthedocs.io/en/latest/reqparse.html
#### 4.@marshal_with的使用
在restful中的api中，需要向客户端返回规范的json数据，但有时候需要返我们所需要格式的数据，此时，需要使用flask-restful中的fields来为想输出的数据指定一个规范，并最终将这些规范通过@marshal_with来将原始数据转换成我们想要的数据并返回。  

基础使用：
```python
from flask_restful import Resource, fields, marshal_with

resource_fields = {
    'name': fields.String,
    'address': fields.String,
    'date_updated': fields.DateTime(dt_format='rfc822'),
}

class Todo(Resource):
    @marshal_with(resource_fields, envelope='resource')
    def get(self, **kwargs):
        return db_get_todo()  # Some function that queries the db
```
等价于：
```python
class Todo(Resource):
    def get(self, **kwargs):
        return marshal(db_get_todo(), resource_fields), 200

```
里面可以使用attribute(更改名字，也可以传入lambda函数，可以用来截取字符串等)，default来指定默认值，或者自定义格式类(只需继承fields类即可)，支持列表，和复杂的数据格式等
> 更多细节，见：https://flask-restful.readthedocs.io/en/latest/fields.html
#### 5.添加路由
```python
api.add_resource(Index, '/', endpoint='index')
api.add_resource(Code, '/code', endpoint='code')
api.add_resource(OpenId, '/openid', endpoint='openid')
```

## flask-restful中的坑
 > 1.无论怎样操作都检测不到模型的创建和修改？
 > 这是因为模型所在的文件不能被启动的文件(manage.py)所引用到，需要整个项目中模型所在文件能直接或间接的被引用到，如：
```
graph TD
    A[manage.py]--call---B[*APP/init.py* 中的方法*create_app*]
    B--call---C[*App/ext.py*中的*init_ext*方法,初始化一些第三方插件如sql-alchemy中的db,restful中Api等]
    B--call---D[*APP/urls.py*中的*init_urls*方法,其中该方法用于添加restful的路由,取代蓝图]
    C==from .ext import api==>D
    E[*flask-restful*中的Resoource]==import==>D
    F[*APP/models.py*]==from .models import *==>E
    C==from .ext import db==>F
```