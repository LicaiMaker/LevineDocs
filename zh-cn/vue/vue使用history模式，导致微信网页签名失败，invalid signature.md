> vue项目中为了去掉路径中的#号，导致微信签名失败，原因分析：history模式副带的页面刷新问题和iOS、Android获取url方式不同的兼容问题，在vue-router模式为history的情况下, 由于IOS微信浏览器在验证微信jssdk签名时,需要的URL是第一次进入该应用时的URL, 并不是当前页面的URL, 所以这里需要针对IOS微信浏览器作特殊处理；

> 从 A页面，跳转到B页面，由于没有刷新，B调用 JSSDK的 内容，由于vue-router切换的时候 都是操作的浏览器历史记录，真实url为第一次刚进入时的url。每次路由变化时都重新请求下签名，签名的url 需要用第一次进入时的url。

>IOS：微信IOS版，每次切换路由，SPA的url是不会变的，发起签名请求的url参数必须是当前页面的url就是最初进入页面时的url

>Android：微信安卓版，每次切换路由，SPA的url是会变的，发起签名请求的url参数必须是当前页面的url(不是最初进入页面时的)

- 解决方法：
在beforeRouterEnter或者beforeEach方法
```js
    beforeRouteEnter(to, from, next) {
          var u = navigator.userAgent;
          var isiOS = !!u.match(/\(i[^;]+;( U;)? CPU.+Mac OS X/); //ios终端
          if (isiOS && to.path !==  location.pathname) {
                location.assign(to.fullPath) // 此处不可使用location.replace
          } else {
                next()
          }
    } 
```

> 参考地址：https://www.jianshu.com/p/54cb36db6479

> 另外，在小程序中打开vue项目中的某个网页时，这个网页中需要使用微信的获取地理位置的接口，在微信开发工具和ios上使用都没有问题，唯独在android的微信小程序中报了config:failed ,invalid signature。
> 这是因为在android的小程序中也需要请求第一次进入该项目时的url，故新的代码如下：
```js
// ios微信和android微信适配
  var u = navigator.userAgent
  var isiOS = !!u.match(/\(i[^;]+;( U;)? CPU.+Mac OS X/) // ios终端
  if (isiOS && to.path !== location.pathname) {
    location.assign(to.fullPath) // 此处不可使用location.replace
  } else if (u.indexOf('miniProgram')) {
    location.assign(to.fullPath) // 此处不可使用location.replace
  } else {
    next()
  }
```