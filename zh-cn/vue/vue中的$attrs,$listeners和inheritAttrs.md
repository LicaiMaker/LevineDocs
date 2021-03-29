-先说说$attrs和$listeners一起inheritAttrs的作用**
现在有以下组件层次关系：
```
<ComponentA>
    <ComponentB>
        <ComponentC>
        </ComponentC>
    </ComponentB>
</ComponentA>    

```
现在想将A中的变量传给C,该怎么办呢？
当然可以将变量通过B中的props属性传递给B,然后在B中通过C的props属性传递给C，如下：
```
<ComponentA>
    <ComponentB :b1='1'>// B中有props: ['b1']
        <ComponentC :c1="{{b1}}"> // C中有props:['c1']
        </ComponentC>
    </ComponentB>
</ComponentA>

```
但是现在有了$attrs，它包含了父作用域中不作为 prop 被识别 (且获取) 的特性绑定 (class 和 style 除外)：
```
<template>
 <div>
  <p>我是父组件</p>
  <test name="tom" :age="12" :id="12345" class="child" style="color: red" />
 </div>
</template>
 
<script>
export default {
 components: {
  test: {
   template: `
   <div>
    <p>我是子组件</p>
    <test2 v-bind="$attrs" s1="sss" s2="sss" />
   </div>`,
   props: ["name"],
   created() {
    console.log(this.$attrs); // {age: 12, id: 12345}
   },
   components: {
    test2: {
     template: `<p>我是孙子组件</p>`,
     props: ["age", "s1"],
     created() {
      console.log(this.$attrs); // {s2: "sss", id: 12345}
     }
    }
   }
  }
 }
};
</script>
```
可以使用$attrs来向孙子组件传递参数：
```
// 父组件father.vue
<div>
    <son :a='1'>// 儿子组件（加冒号的，说明后面的是一个变量或者表达式；没加冒号的后面就是对应的字符串字变量）
    </son>
</div>
// 儿子组件son.vue
<div>
    <grandson v-bind='$attrs'>// 孙子组件
    </grandson>
</div>
// 孙子组件grandson.vue
<div>
   this is grandson component!
   attrs:{{$attrs}}
   attrs中的值是：{{$attrs['a']}}
</div>
```
**inheritAttrs**的作用：
默认情况下父作用域的不被认作 props 的特性绑定 (attribute bindings) 将会“回退”且作为普通的 HTML 特性应用在子组件的根元素上.
> 但是如果我们并不想在根元素上添加这些不需要的属性,通过设置 inheritAttrs 到 false，这些默认行为将会被去掉。而通过 (同样是 2.4 新增的) 实例属性 $attrs 可以让这些特性生效，且可以通过 v-bind 显性的绑定到非根元素上

总结：$attrs存储非prop特性，inheritAttrs控制vue对非prop特性默认行为

另外$listeners的也可以保证爷孙组件间通信
```
// father.vue
<template>
<div>
      <son :a="10" :b="2" v-on:onClick="onGrandClicked">
</template>
<script>
import Son from './son'
export default {
  name: 'father',
  components: {Son},
  methods: {
    onGrandClicked () {
      console.log('我是爷爷！我被点击了！')
    }
  }
}
</script>
// son.vue
<template>
    <div>
      <grandson v-bind="$attrs" v-on="$listeners"></grandson>
    </div>
</template>

<script>
import grandson from './grandson'
export default {
  name: 'son',
  components: { grandson}
}
</script>
// grandson.vue
<template>
    <div @click="onGrandClick">
      <span>孙子</span>
    </div>
</template>

<script>
export default {
  name: 'grandson',
   methods: {
    onGrandClick () {
      this.$emit('onClick')
      console.log('我是孙子！我被点击了!')
    }
  }
}
</script>
```
> 在son组件中农为grandson组件添加了`v-on='$listeners'`,就能把father中添加的onClick事件传递给了son然后传递给了grandson

> 更多关于$attrs和$listeners详情如下：
https://blog.csdn.net/songxiugongwang/article/details/84001967