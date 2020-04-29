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
因为设置了scoped属性，所以上述的`v-html-str`中`<span class="black-text">`中的`black-text`不能生效。

解决方案：使用深度选择器
> - 1.使用`>>>` 修饰class，如：
> `>>> .black-text{}`
> - 2.使用`/deep/`修饰class,如：`/deep/ .black-text{}`  
> 这样就能作用于v-html了,参考链接：[Vue.js中v-html渲染的dom添加scoped的样式](https://segmentfault.com/a/1190000018820271),

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

> `.el-input__inner`必须添加上，才能生效

# el-input不能输入？
在使用el-input时不能输入任何东西，只需要添加v-model属性，绑定一个变量即可。
```vue
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
```vue
<template>
  <div id="app">
    <keep-alive>
      <router-view></router-view>
    </keep-alive>
  </div>
</template>
```
> 如果想控制单个页面使用keep-alive模式，
```vue
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
```js
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