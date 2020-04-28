 

## 一.通过自定义指令去修改（单个修改比较好）
- 1.在main.js 页面里添加自定义指令
```js
Vue.directive('title', {//单个修改标题
  inserted: function (el, binding) {
    document.title = el.dataset.title
  }
})
```
- 2.在需要修改的页面里添加v-title 指令
```html
<div v-title data-title="我是新的标题"></div>.
```
## 二.使用插件vue-wechat-title
- 1.安装插件
```shell
npm install vue-wechat-title --save
```
- 2.在main.js里面引入插件
```js
import VueWechatTitle from 'vue-wechat-title'//动态修改title
Vue.use(VueWechatTitle)
```
- 3.在路由里面 添加title(router/index.js)
```js
         routes: [{
			path: '/login',
			component: Login,
			meta: {
				title: '登录'
			}
		}, {
			path: '/home',
			component: Home,
			meta: {
				title: '首页'
			}
		},]
```
- 4.在app.vue 中添加 指令 v-wechat-title
```
<router-view v-wechat-title='$route.meta.title' />
```
## 三.通过router.beforeEach导航守卫来修改

- 1.在router的index里写自己的路径和title
```js
const router = new Router({
	routes: [{
			path: '/login',
			component: Login,
			meta: {
				title: '登录'
			}
		}, {
			path: '/home',
			component: Home,
			meta: {
				title: '首页'
			}
		},	
	]

})
```
- 2.使用router.beforeEach对路由进行遍历，设置title
```js
router.beforeEach((to, from, next) => {//beforeEach是router的钩子函数，在进入路由前执行
  if (to.meta.title) {//判断是否有标题
	console.log(to.meta.title)
    document.title = to.meta.title
  }
  next()
})
```

> 拓展：当我为了解决授权问题的时候，在router.beforeEach添加title后，发现title一闪而过，又回到了默认的title(`可在index.html中的title标签中修改`),我的代码如下：
```js
router.beforeEach((to, from, next) => {
  // 因为下面使用了location.assign，这个改变title的方式无效
  if (to.meta.title) {
    document.title = to.meta.title
  }
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
})
```
> 这里使用了lication.assign后，设置title无效，所以我暂时采取的方式是，在每个组件中的created方法中添加`document.title = this.$route.meta.title`,只是，在页面切换时有一个title切换的过程（A页面title ->默认title ->B页面title，而不是：A页面title -> B页面title）