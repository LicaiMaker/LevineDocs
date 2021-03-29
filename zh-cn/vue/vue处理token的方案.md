> 我的项目是和微信相关的使用的是微信登录，我的后端是这样设计的：根据前端登录微信获取的openID来生成token：
```
from flask import current_app
from flask_restful import Resource, reqparse
from itsdangerous import TimedJSONWebSignatureSerializer as Serializer


def parse_get():
    parser_get = reqparse.RequestParser()
    parser_get.add_argument('openId', required=True, help="openId不能为空！")
    return parser_get.parse_args()


class TokenApi(Resource):
    # 拿openID生成token
    def get(self):
        try:
            params = parse_get()
            openId = params['openId']
            s = Serializer(current_app.config['SECRET_KEY'], expires_in=3600)
            return {
                'code': 0,
                'token': bytes.decode(s.dumps({'openId': openId})),
                'expiresIn': 3600
            }
        except Exception as e:
            return {
                'code': -1,
                'token': None,
                'expiresIn': 3600,
                'message': '%s' % e
            }
```
> 接口返回了token和expiresIn(过期时间)参数，expiresIn方便前端判断过期后，重新请求token

前端添加request拦截器，判断token是否过期，若过期，则重新请求token，并发起请求，让用户无感知的情况下刷新token;另外，在后端返回中，如果报了401，则重新请求token。
```
import axios from 'axios'
import api from './api'
import ss from './ss'
import {service} from './config'

axios.defaults.timeout = 5000 // 请求超时5秒
// 创建axios实例
const _service = axios.create({
  timeout: 30000 // 请求超时时间
})
// 添加request拦截器
_service.interceptors.request.use(async (config) => {
  // 加时间戳防止请求缓存
  console.log('请求地址：', config.url)
  // 拦截自己服务器编写的接口地址
  if (config.url && config.url.startsWith(service.webAPIHost)) {
    // 添加authorization头信息
    if (ss.getItem('petToken')) {
      // 判断token过时与否
      if (new Date().getTime() > ss.getItem('petTokenExpiresIn')) {
        // todo: token过期,重新请求token,这里是个死循环，接口请求又被拦截了
        // 如果请求的是token接口，则不用拦截，否则会陷入死循环
        if (!config.url.startsWith(service.token)) {
          let token = await getToken()
          if (token) {
            config.headers.Authorization = 'Bearer ' + token
            console.log('使用新的token')
          } else {
            // 没有获取到token，用老的token
            config.headers.Authorization = 'Bearer ' + ss.getItem('petToken')
          }
        }
      } else {
        // token未过期
        config.headers.Authorization = 'Bearer ' + ss.getItem('petToken')
      }
    }
  }
  // config.headers.Authorization = 'Bearer ' + ss.getItem('petToken')
  config.params = {
    _t: Date.parse(new Date()) / 1000,
    ...config.params
  }
  return config
}, error => {
  return Promise.reject(error)
})
// 添加response拦截器
_service.interceptors.response.use(
  response => {
    let res = {}
    res.status = response.status
    res.data = response.data
    return res
  },
  async error => {
    console.log('嘿嘿：', error)
    // 401 则重新过去token
    if (error && error.response && error.response.status === 401) {
      let token = await getToken()
      console.log('tokenss:', token)
    }
    if (error && error.response && error.response.status === 404) {
      // router.push('/blank.vue')
      console.log('error http 404:', error.response)
    }
    return Promise.reject(error.response)
  }
)

export function get (url, params = {}, headers = {}) {
  console.log('hhh:', url, params)
  params.t = new Date().getTime() // get方法加一个时间参数,解决ie下可能缓存问题.
  return _service({
    url: url,
    method: 'get',
    headers: headers,
    params: params
  })
}

// 封装post请求
export function post (url, data = {}, headers = {}) {
  // 默认配置
  let sendObject = {
    url: url,
    method: 'post',
    headers: headers,
    data: data
  }
  sendObject.data = JSON.stringify(data)
  return _service(sendObject)
}

// 封装put方法 (resfulAPI常用)
export function put (url, data = {}, headers = {}) {
  return _service({
    url: url,
    method: 'put',
    headers: headers,
    data: JSON.stringify(data)
  })
}

// 删除方法(resfulAPI常用)
export function deletes (url, data = {}, headers = {}) {
  return _service({
    url: url,
    method: 'delete',
    headers: headers,
    data: JSON.stringify(data)
  })
}

// 不要忘记export
export {
  _service
}

// 获取token方法
function getToken () {
  let openId = ss.getItem('openId')
  console.log('参数：', openId)
  return api.getToken(openId)
    .then(({data: {code, token, expiresIn}}) => {
      console.log('获取token成功：', token)
      if (code === 0) {
        ss.setItem('petToken', token)
        // 保存token过期时间
        ss.setItem('petTokenExpiresIn', new Date().getTime() + expiresIn * 1000)
        return Promise.resolve(token)
      } else {
        return Promise.resolve(null)
      }
    }).catch(err => {
      console.log('获取token出错：', err)
      return Promise.resolve(null)
    })
}

```
> 上面有问题就是：虽然前端根据expiresIn参数判断token还未过期，但是传到后端时，token已经过期了，则会返回401错误，虽然我在401的情况下，重新请求了token，但是不能重新请求之前发起的请求（之前的请求就是token过期的那个请求），所以我们可以在尝试下面两种方式：

- 1.前端在判断token过期时间时，可以将条件设置为，离过期时间还有5分钟，这样就可以在token失效5分钟前重新请求token，不至于到后端的时候token已经过期了
- 2.如果返回401，重新请求token之后，此时什么都不做(页面跳转，请求等等之类的，都不执行)，让用户重新主动发起一次请求；或者在请求token后，重定向到主页。

> https://blog.csdn.net/qq_41609488/article/details/107602360?utm_medium=distribute.pc_aggpage_search_result.none-task-blog-2~all~sobaiduend~default-1-107602360.nonecase&utm_term=jwt%20%E8%AE%A9%E6%9F%90%E4%B8%AAtoken%E5%A4%B1%E6%95%88&spm=1000.2123.3001.4430