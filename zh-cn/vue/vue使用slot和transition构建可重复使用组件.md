vue中拥有transition组件，可以为组件添加transition动画
> 下面先构建一个带有渐变效果的通用组件
```
<!--使用transition和slot来实现可重用的，带不透明度渐变效果的组件-->
<template>
  <transition name="fade" v-bind="$attrs" v-on="$listeners">
    <slot></slot>
  </transition>
</template>

<script>
export default {
  name: 'FadeTransition'
}
</script>

<style scoped>
  .fade-enter-active,
  .fade-leave-active {
    transition: opacity 0.3s;
  }
  .fade-enter,
  .fade-leave-to {
    opacity: 0;
  }
</style>

```
> 使用这个渐变的组件
```
<template>
  <div>
    <button v-on:click="fade">toggle</button>
    <FadeTransition >
        <div key="red" v-if="show" class="box"></div>
    </FadeTransition>
  </div>
</template>

<script>
import FadeTransition from './FadeTransition'
export default {
  name: 'TestFadeTransition',
  components: {FadeTransition},
  data () {
    return {
      show: false
    }
  },
  methods: {
    fade: function () {
      this.show = !this.show
    }
  }
}
</script>

<style lang="scss" scoped>
.box {
  width: 200px;
  height: 200px;
  border-radius: 5px;
  background-color: red;
  box-shadow: rgba(251, 17, 116, 0.5) 0px 6px 20px;
}
</style>

```
可以将上面的列子修改一下，使之每次点击按钮变换盒子的颜色
```
<template>
  <div>
    <button v-on:click="fade">toggle</button>
    <FadeTransition mode="out-in">
        <div key="red" v-if="show" class="box"></div>
        <div key="blue" v-else class="blue-box"></div>
    </FadeTransition>
  </div>
</template>

<script>
import FadeTransition from './FadeTransition'
export default {
  name: 'TestFadeTransition',
  components: {FadeTransition},
  data () {
    return {
      show: false
    }
  },
  methods: {
    fade: function () {
      this.show = !this.show
    }
  }
}
</script>

<style lang="scss" scoped>
.box {
  width: 200px;
  height: 200px;
  border-radius: 5px;
  background-color: red;
  box-shadow: rgba(251, 17, 116, 0.5) 0px 6px 20px;
}
.blue-box{
  @extend .box;
  background-color: blue;
  box-shadow: rgba(12, 141, 213, 0.5) 0px 6px 20px;
}
</style>

```
> 上面的mode='out-in'非常重要，它会通过`v-bind="$attrs"`传递给transition组件，表示在组件切换的出现的先后顺序

> 现在使用transition-group和slot实现动态添加组件和删除组件的效果
```
<template>
    <transition-group
      enter-active-class="fadeIn"
      leave-active-class="fadeOut"
      move-class="fadeMove"
      v-bind="$attrs"
      v-on="hooks"
    >
      <slot></slot>
    </transition-group>
</template>

<script>
export default {
  name: 'FadeTransitionGrop',
  props: {
    duration: {
      type: Number,
      default: 300
    }
  },
  computed: {
    hooks () {
      return {
        beforeEnter: this.setDuration,
        afterEnter: this.cleanUpDuration,
        beforeLeave: this.setDuration,
        afterLeave: this.cleanUpDuration,
        leave: this.setAbsolutePosition,
        ...this.$listeners
      }
    }
  },
  methods: {
    setDuration (el) {
      el.style.animationDuration = `${this.duration}ms`
    },
    cleanUpDuration (el) {
      el.style.animationDuration = ''
    },
    setAbsolutePosition (el) {
      // if (this.group) {
      el.style.position = 'absolute'
      // }
    }
  }
}
</script>

<!--注意着个地方的style不能添加scoped，不然没有动画效果-->
<style>
@keyframes fadeIn {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}
.fadeIn {
  animation-name: fadeIn;
}
@keyframes fadeOut {
  from {
    opacity: 1;
  }
  to {
    opacity: 0;
  }
}
.fadeOut {
  animation-name: fadeOut;
}
.fadeMove {
  transition: transform 0.3s ease-out;
}
</style>

```
> hooks中的函数是组件transition的钩子函数，beforeEnter和beforeLeave需要设置动画持续时间；setAbsolutePosition函数设置了元素的position:absolute;这样会引起其他元素的移动，从而触发移动的动画（.fadeMove）

如何使用呢？
```
<template>
    <div>
      <button @click="addItem">add</button>
      <div class="box-wrapper">
        <FadeTransitionGroup :duration="100">
          <div class="box"
                v-for="(item, index) in list"
                @click="remove(index)"
                :key="item">
          </div>
        </FadeTransitionGroup>
      </div>
    </div>
</template>

<script>
import FadeTransitionGroup from './FadeTransitionGroup'
export default {
  name: 'TestFadeTransitionGroup',
  components: {FadeTransitionGroup},
  data () {
    return {
      list: [1, 2, 3, 4, 5]
    }
  },
  methods: {
    remove (index) {
      this.list.splice(index, 1)
    },
    addItem () {
      let randomIndex = Math.floor(Math.random() * this.list.length)
      this.list.splice(randomIndex, 0, Math.random())
    }
  }
}
</script>

<style lang="scss" scoped>
.box-wrapper {
  display: flex;
  flex-direction: row;
  width: 100%;
  flex-wrap: wrap;
}
.box {
  display: inline-block;
  width: 50px;
  height: 50px;
  margin-top: 20px;
  margin-right: 10px;
  background-color: rgb(108, 141, 213);
  box-shadow: rgba(108, 141, 213, 0.5) 0px 6px 20px;
  border-radius: 10px;
}
</style>

```
> splice函数是来操作ArrayObject，用来向其中添加元素或删除元素

> 参考地址:https://vuejsdevelopers.com/2018/02/26/vue-js-reusable-transitions/

> ***拓展：transition-group不使用钩子函数实现动态添加和动态删除数字***：
```
<!-- Use preprocessors via the lang attribute! e.g. <template lang="pug"> -->
<template>
 <div id="app">
  <div id="list-demo">
    <button @click="add">Add</button>
    <button @click="remove">Remove</button>
    <transition-group name="list" tag="p">
      <span v-for="item in items" :key="item" class="list-item">
        {{ item }}
      </span>
    </transition-group>
  </div>
 </div>   
</template>

<script>
export default {
  name:'ss',
  data() {
    return {
      items: [1, 2, 3, 4, 5, 6, 7, 8, 9],
      nextNum: 10
    }
  },
  methods: {
    randomIndex() {
      return Math.floor(Math.random() * this.items.length)
    },
    add() {
      this.items.splice(this.randomIndex(), 0, this.nextNum++)
    },
    remove() {
      this.items.splice(this.randomIndex(), 1)
    }
  }
}
</script>

<!-- Use preprocessors via the lang attribute! e.g. <style lang="scss"> -->
<style>
.list-item {
  display: inline-block;
  margin-right: 10px;
}
 .list-enter-active {
   transition: all 1s ease;
 }
.list-leave-active {
  position:absolute; 
  transition: all 1s ease;
}
 .list-enter,
.list-leave-to {
  opacity: 0;
  transform: translateY(30px);
}
  .list-move {
    transition: transform 0.8s ease;
  }
</style>
``
> 参考地址：https://v3.vuejs.org/guide/transitions-list.html#reusable-transitions
