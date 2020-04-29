> webhook用于，托管在github上的仓库被触发时(如push,事件可自己选择)，github将会向配置的webhook payload url发送一个post请求，头部包含一些secret信息(如果在github配置了secret的话)

## 在github仓库中配置webhook的payload url
在github中的仓库的settings下的webhooks中添加一个webhook，
- payload url填写你自己的地址：http://example.com/webhook
- Content-type看自己需要；一般都设置成application/json
- secret用于回调地址中的校验，一般可以在命令行中用ssh-keygen生成一个id_rsa.pub文件，可将这个文件中的内容作为secret，也可以自己生成一个随机字符串(不过要保证在校验时，服务器上的secret和github上的secret要一致)

## 编写一个视图来处理github的回调
我们可以在自己的项目中新建这个路由或者在其他的项目中创建这个路由，这个路由必须和github上payload url一样，我在我的项目中来处理github的这个post请求。
- 新建一个Resource，取名WebHook.py(因为我的项目使用了flask-restful)(当然也可以通过蓝图来创建)
```python
import hmac
import os
from flask import request
from flask_restful import Resource
from App import settings


class WebHook(Resource):
    def post(self):# github中会使用post方法来请求
        '''用于github的webhook的post请求，拿到请求后
            验证其secret，IP地址是否是在github调用白名单内，
            以及最终的部署命令
            '''
        signature = request.headers.get('X-Hub-Signature')
        sha, signature = signature.split('=')
        secret = str.encode(settings.Config.GITHUB_WEBHOOK_KEY)
        hashhex = hmac.new(secret, request.data, digestmod='sha1').hexdigest()
        if hmac.compare_digest(hashhex, signature):
            # 使用shell来执行提交命令
            if depoly():
                return "params accepted successfully!"
            else:
                return "出现错误"
def depoly():
    try:

        '''首先下本地需要执行pip freeze > requirements.txt然后在远端将这里面的包安装'''
        # 1.启动python虚拟环境并切换到仓库目录
        os.system('/bin/bash --rcfile /home/OldManInfo_env/bin/activate')
        # 2.从远程仓库pull最新代码
        # 由于切换到另一个工作目录是在子进程中完成的，不会影响主进程的目录，故需要在切换后“接着”使用其他命令
        cmd0 ='cd /home/OldManInfo_Flask'
        cmd1='git pull origin master'
        # 3.从requirements.txt中安装最新的库
        cmd2='pip install -r requirements.txt'
        # 4.数据库迁移（一般只是在第一次中使用，故可省略）
        # 需要在远端服务器上创建这个数据库
        cmd3='python manage.py db init'
        cmd4='python manage.py db migrate'
        cmd5='python manage.py db upgrade'
        # 5.重启gunicorn服务器（如果在gunicorn.conf.py中配置了reload参数，则会自动重启）
        cmd="&&".join([cmd0,cmd1,cmd2])+("&& %s"%cmd3 if settings.Config.INIT_DB else "")\
        +("&& %s && %s"%(cmd4,cmd5) if settings.Config.MIGRATE_DB else "")
        # 写入文件
        # with open('flask.log','w') as f:
        #     f.write('\ncommand is :%s'%cmd)
        #     f.close()
        os.system(cmd)
        return True
    except Exception as e:
        # print('exception:',e)
        # 写入文件
        # with open('flask.log', 'w') as f:
        #     f.write('\nexception is :',  e)
        #     f.close()
        return False
```
> 在deploy方法中执行了一些shell命令。用于代码更新和数据库操作，其中在settings.py中配置了两个布尔参数INIT_DB和MIGRATE_DB来控制是否初始化数据库和是否迁移数据连个操作

但是这个操作可能会显示超时，这是因为再执行os.system命令时，可能会超时，但是命令基本上都会被执行，同样，可以用subprocess.Popen单体os.system，更多详见python中使用shell命令
- 添加路由
需要将这个Resource添加到‘http://example.com/webhook’上
`api.add_reource(WebHook,'/webhook',endpoint='webhook')`