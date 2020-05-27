## 创建http.js
将put,get,拦截器放在http.js文件中
```js
import axios from 'axios'
// import router from './router/index'

axios.defaults.timeout = 10000 // 请求超时5秒

// 创建axios实例
const service = axios.create({
  timeout: 30000 // 请求超时时间
})
// 添加request拦截器
service.interceptors.request.use(config => {
  return config
}, error => {
  Promise.reject(error)
})
// 添加response拦截器
service.interceptors.response.use(
  response => {
    let res = {}
    res.status = response.status
    res.data = response.data
    return res
  },
  error => {
    if (error.response && error.response.status === 404) {
      // router.push('/blank.vue')
      console.log('error http 404:', error.response)
    }
    return Promise.reject(error.response)
  }
)

export function get (url, params = {}, headers = {}) {
  params.t = new Date().getTime() // get方法加一个时间参数,解决ie下可能缓存问题.
  return service({
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
  return service(sendObject)
}

// 封装put方法 (resfulAPI常用)
export function put (url, data = {}, headers = {}) {
  return service({
    url: url,
    method: 'put',
    headers: headers,
    data: JSON.stringify(data)
  })
}

// 删除方法(resfulAPI常用)
export function deletes (url) {
  return service({
    url: url,
    method: 'delete',
    headers: {}
  })
}

// 不要忘记export
export {
  service
}

```

## 创建congfig.js中
将接口地址统一放在congfig.js中
```js
var apiHost = 'https:xxx.com'
var webHost = 'https://yyy.com'
var webAPIHost = webHost + '/api'
var service = {
  apiHost,
  webHost,
  webAPIHost,
  getOpenId: `${webAPIHost}/openid`,
  verifyQrCodeActive: `${apiHost}/api/v2/active`,
  weChatState: `${apiHost}/api/v1/wechatstate`, // 请求用户状态
  verifyCode: `${apiHost}/api/v1/verifycode`, // 根据电话号码获取验证码
  verify: `${apiHost}/api/v4/VerifyCodeOnly`, // 根据手机号和验证码验证
  qrCodeInfo: `${webAPIHost}/qr_code_info`, // qrcode绑定的信息
  weChatCalling: `${apiHost}/api/v4/wechatcalling`,
  locationDesc: `${webAPIHost}/locationDesc`,
  msgNotification: `${webAPIHost}/msgNotification`
  // getURLFrom: `${webHost}/getUrlFrom`,
  // getCode: `${webHost}/code`
}
export {
  service
}

```

## 创建api.js
将接口封装在api.js中
```js
// 解构写法
import {get, post} from './http'
import {service} from './config'

const wxConfig = () => {
  return get(service.wxConfig)
}
const wxConfigWithUrl = (url) => {
  return get(service.wxConfig, {
    url: url
  })
}
const getHeader = function (openId) {
  return {
    'content-type': 'application/json',
    'Authorization': 'Basic ' + openId
  }
}

const getOpenId = (code) => {
  return get(service.getOpenId, {
    code: code
  })
}
const weChatState = (openId, nickName = '', serviceno = '') => {
  return post(service.weChatState,
    {
      Nickname: nickName,
      serviceno: serviceno
    }
    , getHeader(openId))
}
const verifyQrCodeActive = (qrCodeId) => {
  return get(service.verifyQrCodeActive, {
    qrcodeid: qrCodeId
  })
}
// 激活二维码
const activateQrCode = (openId, qrcodeid, ucallfreeid) => {
  return post(service.verifyQrCodeActive, {
    qrcodeid: qrcodeid,
    ucallfreeid: ucallfreeid
  }, getHeader(openId))
}
// 根据手机号获取验证码
const getVerifyCode = (openId, number) => {
  return get(service.verifyCode, {
    tel: number
  }, getHeader(openId))
}
// 验证手机号是否合法
const verifyNumber = (openId, number, verifyNumber) => {
  return post(service.verify, {
    AuthCode: verifyNumber,
    Tel: number
  }, getHeader(openId))
}

const getInfo = (qrCodeId) => {
  return get(service.qrCodeInfo, {qrcodeid: qrCodeId})
}
/**
 * 保存老人信息
 * @param qrCodeId 二维码id
 * @param oldManInfo 老人信息，类型：{}
 * @param phone_number 联系人，类型：[]
 */
const saveInfo = (qrCodeId, oldManInfo, phoneNnumber) => {
  return post(service.qrCodeInfo, {
    qr_code_id: qrCodeId,
    old_man_info: oldManInfo,
    phone_number: phoneNnumber
  }, {
    'Content-Type': 'application/json'
  })
}
const weChatCalling = (openId, phoneNumbers, qrcodeid) => {
  var serviceNo = ''
  for (var i = 0; i < phoneNumbers.length; i++) {
    serviceNo += phoneNumbers[i] + '`'
  }
  serviceNo = serviceNo.substr(0, serviceNo.length - 1)
  console.log('serviceno:', serviceNo)
  return post(service.weChatCalling, {
    Enterpriseid: service.enterpriseid,
    YhServiceno: service.yhServiceno,
    Callmode: 1,
    Servicetype: 1,
    Serviceno: serviceNo,
    Display: 1,
    Eqrcode: qrcodeid
  }, getHeader(openId))
}
/**
 * 根据经纬度获取地理位置的描述信息
 * @param lat 维度
 * @param lon 精度
 * @param key 腾讯地图的key
 */
const getLocationDesc = (lat, lon) => {
  // var d = [lat, lon].join(',')
  return get(service.locationDesc, {
    'lat': lat,
    'lon': lon
  })
}
/**
 * 发送短信通知
 * @param orgid 企业id
 * @param password 密码
 * @param mobile 手机号
 * @param content 短信内容
 */
const sendMsgNotification = (mobile, content) => {
  return post(service.msgNotification, {
    'mobile': mobile,
    'content': content
  }, {
    'Content-Type': 'application/json'
  })
}

export default {
  wxConfig,
  wxConfigWithUrl,
  getOpenId,
  weChatState,
  verifyQrCodeActive,
  activateQrCode,
  getVerifyCode,
  verifyNumber,
  getInfo,
  saveInfo,
  weChatCalling,
  getLocationDesc,
  sendMsgNotification
}

```