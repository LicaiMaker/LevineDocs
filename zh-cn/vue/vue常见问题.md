[TOC]

# scoped和v-html的相互影响
在vue中的style有个scoped属性，指的是style里面的样式只在当前组件有效，但是如果在某个节点中使用了v-html属性，此时传入的HTML字符串里面如果包含一些样式，则不会生效，如：
```vue
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
```js
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