[TOC]

# scoped和v-html的相互影响
在vue中的style有个scoped属性，指的是style里面的样式只在当前组件有效，但是如果在某个节点中使用了v-html属性，此时传入的HTML字符串里面如果包含一些样式，则不会生效，如：
```
<template>
    <div class="container">
        <div v-html="v-html-str"/>
    </div>
</template>
<script>
export default {
  name: 'home',
  data () {
    return {
      v-html-str: '<span class="black-text">绑定成功!</span>',
    }
  }
}
</script>

<style scoped>
  container{
    display: flex;
    flex-direction: column;
    width: 100%;
    height: 100vh;
    justify-content: center;
    align-items: center;
  }
  .black-text{
    color: #333333;
    font-size:1.5rem;
    font-family: "PingFang SC";
    letter-spacing: 0.01rem;
    font-style: normal;
    font-weight: bold;
  }
</style>
```
> 在单文件组件里，scoped 的样式不会应用在 v-html 内部，因为那部分 HTML 没有被 Vue 的模板编译器处理。如果你希望针对 v-html 的内容设置带作用域的 CSS，你可以替换为 CSS Modules 或用一个额外的全局 <style> 元素手动设置类似 BEM 的作用域策略。

因为设置了scoped属性，所以上述的`v-html-str`中`<span class="black-text">`中的`black-text`不能生效。

解决方案：使用深度选择器
> - 1.使用`>>>` 修饰class，如：
`>>> .black-text{}`
> - 2.使用`/deep/`修饰class,如：`/deep/ .black-text{}`  
这样就能作用于v-html了,参考链接：[Vue.js中v-html渲染的dom添加scoped的样式](https://segmentfault.com/a/1190000018820271),

# vue中如何修改element-ui的el-input样式
在使用element-ui时，需要其中的一些样式，同时又需要是自己的样式生效，此时该怎么办呢？
同样的，需要使用穿透，即使用`/deep`或`>>>`两种方式.例子如下：
我使用elementui的input组件的效果的同时，不要边框，则可以这么写css样式：
```css
/deep/ .no-border-input .el-input__inner{
   border: 0 none;
  }
```
或
```css
>>> .no-border-input .el-input__inner{
   border: 0 none;
  }
```

> `.el-input__inner`必须添加上，才能生效，.no-border-input表示自定义的名字
# element-ui中的select中的图标在点击时有阴影，如何去掉？
> 将cursor设为none即可
```
/deep/ .el-select .el-input .el-select__caret{
  cursor: none;
}
```
> 同理，在使用switch是也有这个问题：
```
/deep/ .el-switch__core, .el-switch__label{
  cursor: none;
}
```
# vue中如何修改element-ui的textarea的样式
textarea和上面的input是不一样的，需要单独定义其样式:
```
/deep/  .el-textarea__inner {
    border: 0 none; // 去掉边框
    color: #48534A;
    font-family: PingFangSC-Regular;
    font-size: 2rem; // 修改字体大小
    color: #B2B2B2;
    letter-spacing: 0;
  }
```
> 当然了，要想全局修改`el-input`和`el-switch`，在`app.vue`中添加上述样式，只需要去掉`/deep/`即可

# vue中使用::placeholder伪元选择器更改input中的placeholder样式
```
/*input*/
  /deep/  .el-input__inner{
   border: 0 none;
    color: #48534A;
    max-lines: 1;
    /*padding: 0;*/
    font-size: 1.8rem;
    box-sizing: border-box;
     &::placeholder{
      font-size: 1.8rem;
       color: #C5C5C5;
     }
  }

/*textarea*/
  /deep/  .el-textarea__inner {
    color: #48534A;
    font-family: PingFangSC-Regular;
    font-size: 1.8rem;
    color: #48534A;
    letter-spacing: 0;
    border: 0.5px solid #979797;
    border-radius: 10px;
    box-sizing: border-box;
    /*这里使用的是less语法，&表示本对象*/
    &::placeholder{
      font-size: 1.8rem;
      color: #C5C5C5;
     }
  }
```
> 注：上面使用的是less语法，需要在style中添加`lang='less'`,并安装相关less依赖

> 更多伪元素选择器:https://www.html.cn/book/css/selectors/pseudo-element/index.htm

# el-input不能输入？
在使用el-input时不能输入任何东西，只需要添加v-model属性，绑定一个变量即可。
```
<el-input v-model="n" class="name" placeholder="预设内容" clearable></el-input>
<script>
export default {
  name: 'settings',
  data () {
    return {
      n: ''
    }
  }
}
</script>
```

# vue中使用axios向flask后端传递post参数，后端一直获取不到？
有时候需要在vue中发送post请求时需要设置‘Content-Type:application/json’
> 参考地址：[axios 发 post 请求，后端接收不到参数的解决方案](https://blog.csdn.net/csdn_yudong/article/details/79668655)

# 在ios中，vue中返回上一个页面，页面不刷新的问题
首先，保存页面数据的方法有以下两种：
- 1.使用keep-alive和activated钩子函数 

在App.vue中：
```
<template>
  <div id="app">
    <keep-alive>
      <router-view></router-view>
    </keep-alive>
  </div>
</template>
```
> 如果想控制单个页面使用keep-alive模式，
```
<template>
  <div id="app">
    <keep-alive>
      <router-view v-if="$route.meta.keepAlive"></router-view>
    </keep-alive>
    <router-view v-if="!$route.meta.keepAlive"></router-view>
  </div>
</template>
```
> 并且在router/index.js中添加meta信息：keepAlive：true,如下：
```
// router/index.js
routes: [
    {
      path: '/info',
      name: 'InfoDisplay',
      component: InfoDisplay,
      meta: {
        title: '提示',
        keepAlive: true
      }
    },
    ...
]    
```
然后在页面的activated方法中也保留一份添加获取数据的方法，返回这个页面时，页面就会执行activated这个钩子函数，从而执行里面的获取数据的方法，这个可以使页面刷新(在keep-alive模式下，声明周期为created->mounted->activated,返回这个页面时直接执行：activated)。  

---

- 2.不使用keep-alive，使用sessionStorage或localStorage老缓存信息  

使用sessionStorage或localStorage老缓存信息 ，并在mounted方法中读取这些缓存数据并填充页面。

> 但是，在ios中执行时，却不起作用，重新返回页面时，没有执行activated和mounted方法（页面没有刷新）。所以在ios中vue页面的数据并没有保存，解决办法是，让页面强制刷新，执行生命周期方法：
```js
//在页面的mounted方法中添加下面的代码
// 这部分代码是为了解决在ios上页面返回时不刷新的问题--start
    var u = navigator.userAgent
    var isiOS = !!u.match(/\(i[^;]+;( U;)? CPU.+Mac OS X/) // ios终端
    if (isiOS) {
      window.onpageshow = function (e) {
        if (
          e.persisted ||
          (window.performance && window.performance.navigation.type === 2)
        ) {
          window.location.reload()
        }
      }
    }
    // 这部分代码是为了解决在ios上页面返回时不刷新的问题--end
```
> 注意：使用window.location.reload()刷新时是会带着参数一起刷新的，如果带着参数会影响功能，需要去掉参数
# 为什么vue组件的属性，有的需要加冒号“:”，有的不用?
> 加冒号的，说明后面的是一个变量或者表达式；没加冒号的后面就是对应的字符串字变量！
```
<el-dialog
        :lock-scroll="false"// 加冒号，认为后面的‘false’就是bool类型的值，否则就是字符串‘false’
        :visible.sync="centerDialogVisible"
        title="呼叫失败"
        width="80%"
        :show-close="false"
        center>
        <span class="center-dialog-content">{{error_msg}}</span>
        <span slot="footer" class="dialog-footer dialog-btn-group">
          <span type="primary" @click="dialogConfirm" class="confirm-btn">确定</span>
        </span>
      </el-dialog>
```
# 页面刷新会导致后续操作或请求中断
当有以下代码：
```js
window.location.reload()
// 获取地理位置发送通知
this.getLocation()// 接口请求

```
上面的刷新页面会导致后面的接口请求终止。
所以会有这样一个常见的问题：
```js
api.weChatCalling()
          .then((res) => {
            if (res.data.Code === 0) {
              // 网络请求
              this.getLocation()
            }
          })
          .catch(err => {
            console.log('异常:', err)
          })
          .finally(() => {
            window.location.reload
          })

getLocation: function(){
    ...
    // 网络请求
}          
```
在上面的finally函数中刷新了页面，但是在then中又去发起了另一个网络请求，此时很有可能，then中的请求会被中断，因为then中的请求发起之后，就会最终到finally中去了，于是在刷新之后，请求就被中断了。
> 解决办法：使用async和await，让then中的代码执行完成之后，才到finally中去
```js 
api.weChatCalling()
          .then(async (res) => {
            if (res.data.Code === 0) {
              // 网络请求
              await this.getLocation() // 阻塞，等待请求完成
            }
          })
          .catch(err => {
            console.log('异常:', err)
          })
          .finally(() => {
            window.location.reload
          })
```
> 注意，await后面可以放任何表达式，不过我们更多的是放一个返回promise 对象的表达式
```js
getLocation: function(){
    return new Promise((resolve,reject) => {
        ....
        // 网络请求
        var res = xxx
        resolve(res)
    })
} 
```
> 当然，当getLocation中有连续的网络请求时(后面的请求需要拿到前面的请求的返回值)，也可以使用async和await，最后将多个请求的最终结果返回
```
getLocation: function(){
    return new Promise((resolve,reject)=>{
        api.get1().then(async res=>{
            var res1=await api.get2(res)
            var res2=await api.get3(res1)
            resolve(res2)
        }).catch(err=>{
            reject(err)
        })
    })
}
```
这样就能保证getLocation中所有接口都执行完毕之后再返回结果，而这个过程中getLocation中调用者一直是阻塞的，知道返回结果，‘用异步函数实现了同步的效果’

# 在vue(js)中，如何让一段文字单行/多行显示，多余的文字不显示，并以省略号结尾？
在css中添加以下代码即可：
- 单行显示
```
text-overflow:ellipsis;
white-space:nowrap;
overflow:hidden;
```
- 多行显示
```
.line-clamp-n {
  overflow : hidden;
  text-overflow: ellipsis;
  display: -webkit-box;
  -webkit-line-clamp: n;
  -webkit-box-orient: vertical;
}
```
> `-webkit-line-clamp: n`中的n代表显示的行数

# 经常显示this.func() is not function？
明明方法已经定义了，为什么一直提示没有这个方法呢？
```
methods: {
 textTranslate: function (text, to) {
 
  $.ajax({
   url: 'url',
   type: 'post',
   dataType: 'jsonp',
   data: {
    a:'a'
   },
   success: function (data) {
    this.xxx()
   }
  })
 }
}
```
> 注意了：success回调函数里面的this并不是指向的VueModel,当然不能调用定义在vue文件中的方法啦
解决办法：
- 1.使用that代替this
```
methods: {
 textTranslate: function (text, to) {
 let that=this
  $.ajax({
   url: 'url',
   type: 'post',
   dataType: 'jsonp',
   data: {
    a:'a'
   },
   success: function (data) {
    that.xxx()
   }
  })
 }
}
```
- 2.将方法的改为箭头函数的方法
```
success:(data) => {
    that.xxx()
 }
```
> 这样子箭头函数里的this其实是指向VueModel的

# vue中class和style的绑定
- class绑定
```
// 第一种 对象语法
<div class="static" v-bind:class="{ 'class-a': isA, 'class-b': isB }"></div>
<div class="static" v-bind:class="classObj"></div>
// 第二种 数组
<div v-bind:class="[classA, classB]">
// 三元表达式
<div v-bind:class="[classA, isB ? classB : '']"> // 此例始终添加 classA，但是只有在 isB 是 true 时添加 classB 。注意：和上面的对象语法是不一样的
```
- style绑定
```
// 对象语法
<div v-bind:style="{ color: activeColor, fontSize: fontSize + 'px' }"></div>
<div v-bind:style="styleObject"></div>
// 数组语法
<div v-bind:style="[styleObjectA, styleObjectB]">
// 自动添加前缀
当 v-bind:style 使用需要厂商前缀的 CSS 属性时，如 transform(-webkit-transform)，Vue.js 会自动侦测并添加相应的前缀。
```
> 参考地址：https://www.kancloud.cn/dingyiming/vue/114193

# 微信开发工具中如何查看vue页面的详细信息
- 全局安装vue-devtools
```
npm install -i vue-devtools
```
- 命令行键入vue-devtools
```
vue-devtools
```
然后按照界面上的提示`<script src="http://localhost:8098"></script>`添加到想要调试的页面中去就行了


# 为什么vue的transition中使用了translateX或translateY不能按照直线运动？
可能是该元素中的已经有translate的样式，如：
```
.message-box{
  width: 10rem;
  height: fit-content;
  text-align:center;
  display:flex;
  flex-direction:row;
  justify-content:center;
  align-items:center;
  position:fixed;
  left:50%;
  top:5rem;
  transform:translateX(-50%);
}
```
现在添加transition属性后，要将translateX(-50%)加上，比如，我要让他沿y轴运动，那么一定要让其x轴一直保持translateX(-50%)状态：
```
.slide-leave-to {
    ...
    transform: translate(-50%,-100%);
  }
```
# vue2中如何动态为已创建的实例添加响应式的属性？
> 对于已创建的实例，vue是不允许添加根级别的响应式property，但是是可以向嵌套对象添加响应式的property的：
```js
Vue.set(obj,propertyName,value)
```
> 还可以使用`vm.$set`方法，它是Vue.set的别名
```js
this.$set(obj,'b',2)
```
> 但是，如果你需要添加多个响应式的属性呢？如果使用Object.assign添加到原有的对象上的话，这些新添加的属性**不是**响应式的，需要将原有对象和需要混合进去的property一起创建一个新的对象即可：
```js
// 用来代替：Object.assign(this.obj,{a:1,b:2})
this.obj = Object.assign({},this.obj, {a:1,b:2})
```
> Object.assign()第一个是参数为`{}`的话，就表示新建一个对象了，因为，如果第一个参数一个已创建的对象的话，返回的仍然是这个对象(并且所有参数都已经混入到这个对象中)
```
let c ={a:1}
let temp = Object.assign(c,{b:2})
temp===c // true
c // {a:1,b:2}
temp // {a:1,b:2}
let temp2 =Object.assign({},c,{d:3})
c //{a:1,b:2},c的值并未改变
temp2 // {a:1,b:2,d:3},新对象
```
- vue2和vue3的响应式原理：
```js
// vue2
Object.defineProperty(data, 'count', {
    get () {},
    set () {}
})
// vue3
new Proxy(data, {
    set (key,value) {},
    get (key) {}
})
```