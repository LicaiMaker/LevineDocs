> inject和provide需要一起使用，以允许一个祖先组件向其所有子孙后代注入一个依赖，不论组件层次有多深，并在起上下游关系成立的时间里始终生效。 

以上是官方文档的定义，其实inject和provide是一种组件传参方式，它有自己独特的使用场景，当子组件调用父组件的data或者methods时，通过$parent这个API获取到，父子组件两级的情况下可以这么做，如果组件层级非常深，组件层级有N级的话，那就$parent.$parent.....，代码会非常累赘，所以才有了inject和provide
需要注意的是：inject和provide 绑定并不是可响应的。这是刻意为之的。然而，如果你传入了一个可监听的对象，那么其对象的属性还是可响应的。
- provide   

provide组件传递给子、孙组件的属性，主要有下面的几种形式：
```
//选项一
provide: {
  content: 'hello world'
}

//选项二
provide(){
  return {
    content: 'hello world'
  }
}
```
- inject
inject是子孙组件用来接收父级组件属性/方法。选项有以下几种：
```
//选项一
inject: ['content']

//选项二
inject: {
  content: 'content'
}

//选项三
inject: {
  content: {
    from:'content',
    default:'hello world'
  }
}
```
完整案例：
```
<!--父组件-->
<script>
import Son from "@/components/Son.vue";
export default {
  name: "Parent",
  components: { Son },
  provide() {
    return {
      foo: this.foo,
      content: this.content
    }
  },
  data() {
    return {
      content: 'hello world'
    }
  },
  methods: {
    foo() {
      console.log("我是父组件")
    }
  }
};
</script>
<!--子组件-->
<script>
export default {
  name: "Son",
  inject: ['content','foo'],
  created() {
    this.foo();
  }
};
</script>
```
> 参考地址：https://www.jianshu.com/p/970ac33c597c

另外使用provide/inject实现页面刷新.
首先在App.vue中添加下面的代码：
```
<template>
  <div id="app">
  <!--使用isRouterAlive变量-->
    <router-view v-if="isRouterAlive"/>
  </div>
</template>

<script>
export default {
  name: 'App',
  provide () {
    return {
      reload: this.reload
    }
  },
  data () {
    return {
      isRouterAlive: true
    }
  },
  created () {
  },
  methods: {
    reload () {
    this.isRouterAlive = false
    this.$nextTick(function () {
      this.isRouterAlive = true
    })
  }
  }
}
</script>
```
然后在其他子孙组件中使用这个reload
```
export default {
  inject: ['reload'],
  created () {
    this.reload()
  }
  ...
}
```