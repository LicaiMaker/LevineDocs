## 基本流程

- 用户登录->验证密码是否正确->正确则生成一个token，并将token保存到用户信息中，并将token返回给客户端;否则，则报错
- 使用token获取数据->验证token
- 验证token步骤：获取token中的用户名->根据用户名获取token（生成token的步骤中已经将token存入用户信息中）-> 验证token是否一致并验证是否有效->若有效，则返回数据；若过期，则提示token过期

## flask实现
### views.py
```python
# views.py
# 登录程序
# 模拟数据存放用户名和密码
# 模拟数据库
users = {
    'levine': ['123456']
}


def gen_token(uid):
    # uid,随机数，时间戳+7200s，用冒号链接,注意，需要添加encode()转成bytes
    token = base64.b64encode(':'.join([str(uid), str(random.Random()), str(time.time() + 7200)]).encode())
    users[uid].append(token.decode())
    return token


def verify_token(token):
    # 判断token是否相等
    _token = base64.b64decode(token)

    _token = _token.decode() # 转成str
    if not users.get(_token.split(':')[0])[-1] == token:
        return -1  # 用户名不存在，或者token不存在或者token不相等
    # 判断是否过期
    if float(_token.split(':')[-1]) >= time.time():
        return 1
    else:
        return 0


@main.route('/login', methods=['GET', 'POST'])
def login():
    # 获取请求头中的Authorization的值，并使用base64进行解密，然后用“:”分割得到用户名和密码
    # 注意，需要添加decode()将bytes转成str
    uid, pw = base64.b64decode(request.headers['Authorization'].split(' ')[-1]).decode().split(':')
    if users.get(uid)[0] == pw:  # 用户名和密码正确，则生成一个token并返回
        return gen_token(uid)
    else:
        return 'error'


@main.route('/test', methods=['GET', 'POST'])
def test():
    token = request.args.get('token')
    print("token:" + token, 'type:', type(token))
    print(verify_token(token))
    print(users)
    if verify_token(token) == 1:
        return 'data'
    else:
        return 'error'
```

### 调用流程 
- 登录
```python
 import requests
 # 请求登录接口
 r = requests.get('http://127.0.0.1:5000/login', auth=('levine', '123456'))
 print(r.text)# 返回一个token
```
- 请求数据
```python
# 请求数据
# 使用登录接口返回来的token请求数据
token='bGV2aW5lOjxyYW5kb20uUmFuZG9tIG9iamVjdCBhdCAweDAwMDAwMUU2ODU0RUI4Mzg+OjE1ODM3NTIwOTIuMzMzNDMwOA=='
r = requests.get('http://127.0.0.1:5000/test',params={'token':token} )
print(r.text)

```