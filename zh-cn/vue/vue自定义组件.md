> 定义一个topbar,拥有返回键和保存键，可选择显示或不显示
## 1.定义组件样式
新建一个component，命名为topbar.vue

```html
<template>
  <el-row class="bartop">
    <i class="el-icon-menu color-purple margin-horizontal big-font-size" @click="goHome"/>
    <i class="linear-line"/>
    <!--<el-divider direction="vertical"></el-divider>-->
    <i class="el-icon-back color-purple margin-horizontal big-font-size" v-show="!hideBackBtn" @click="onBackTap"/>
    <div v-show="this.showSaveBtn" class="save-area" @click="save">
      <el-image :src="save_icon_url" class="color-purple img-icon-style"/>
      <span class="color-purple small-font-size margin-left-style">保存</span>
    </div>
  </el-row>
</template>

```
样式表如下
```css
<style scoped>
  .bartop {
    display: flex;
    flex-direction: row;
    justify-content: left;
    align-items: center;
    height: 5rem;
    width: 100%;
    background-color: white;
  }

  .margin-horizontal {
    margin-left: 3vw;
    margin-right: 3vw;
  }

  .linear-line{
    width: 0.2rem;
    height: 50%;
    background-color: lightgray;
    border-radius:0.1rem;
  }
  .color-purple {
    color: #432B99;
  }
  .big-font-size{
    font-size: 3rem;
    font-family: "PingFang SC";
  }
  .small-font-size{
    font-size: 1.7rem;
    font-family: "PingFang SC";
  }
  .img-icon-style{
    width: 2.5rem;
    height: 2.5rem;
  }
  .margin-left-style{
    margin-left: 0.5rem;
  }
  /*.bartop:last-child{*/
    /*margin-left: auto;*/
    /*margin-right: 2rem;*/
  /*}*/
  .save-area{
    margin-left: auto;
    margin-right: 2rem;
    display: flex;
    flex-direction: row;
    justify-content: center;
    align-items: center;
  }
</style>
```
> 我是用了element-ui,使用npm下载这个elementui包（可取官网查看），然后在vue项目中的main.js引入elementui
```js
...
import ElementUI from 'element-ui'
...
```
## 2.添加属性
在上面的组件中，我添加了两个属性来控制两个按钮的显示(showSaveBtn和hideBackBtn)
```html
 <i class="el-icon-back color-purple margin-horizontal big-font-size" v-show="!hideBackBtn" @click="onBackTap"/>
 <div v-show="this.showSaveBtn" class="save-area" @click="save">
```
故在组件中药添加这两个属性
```
<template>...</template>
<script>
export default {
  name: 'topbar',
  ...
  props: {
    showSaveBtn: false,
    hideBackBtn: false
  }
}
</script>
<style>...</style>
```

## 3.添加事件
为按钮添加事件
```
<template>...</template>
<script>
export default {
  name: 'topbar',
  ...
  methods: {
    save: function () {
      this.$emit('saveInfo')
    },
    onBackTap: function () {
      this.$router.back()
    },
    goHome: function () {
      this.$router.push('/')
    }
  },
  props: {
    showSaveBtn: false,
    hideBackBtn: false
  }
}
</script>
<style>...</style>
```
> 注：上面的emit方法，表示调用外部的saveInfoz这个自定义方法，所以需要在使用这个组件的时候需要添加@saveInfo='方法'

## 4.在其他vue文件中使用这个组件
- 引入这个组件
```
...
<script>
import Topbar from './topbar'
export default {
  name: 'settings',
  components: {Topbar},
</script>
...
```
- 使用这个组件
```
<template>
    <Topbar class="topbar" :show-save-btn="true" @saveInfo="save"/>
</template>
...
```
> class=‘topbar’表示css样式和Tobar这个组件没有关系，不会改变其内部的样式排版
- 为按钮定义事件
```
export default {
  name: 'settings',
  components: {Topbar},
  methods:{
      save:function(){
          ...
      }
  }
```
> topbar内部使用了`this.$emit('saveInfo')`,当点击按钮时，就会触发@saveInfo后面的方法

## 5.组件完整代码
```
<template>
  <el-row class="bartop">
    <i class="el-icon-menu color-purple margin-horizontal big-font-size" @click="goHome"/>
    <i class="linear-line"/>
    <!--<el-divider direction="vertical"></el-divider>-->
    <i class="el-icon-back color-purple margin-horizontal big-font-size" v-show="!hideBackBtn" @click="onBackTap"/>
    <div v-show="this.showSaveBtn" class="save-area" @click="save">
      <el-image :src="save_icon_url" class="color-purple img-icon-style"/>
      <span class="color-purple small-font-size margin-left-style">保存</span>
    </div>
  </el-row>
</template>

<script>
export default {
  name: 'topbar',
  data: function () {
    return {
      save_icon_url: require('../assets/save.png')
    }
  },
  methods: {
    save: function () {
      this.$emit('saveInfo')
    },
    onBackTap: function () {
      this.$router.back()
    },
    goHome: function () {
      this.$router.push('/')
    }
  },
  props: {
    showSaveBtn: false,
    hideBackBtn: false
  }
}
</script>

<style scoped>
  .bartop {
    display: flex;
    flex-direction: row;
    justify-content: left;
    align-items: center;
    height: 5rem;
    width: 100%;
    background-color: white;
  }

  .margin-horizontal {
    margin-left: 3vw;
    margin-right: 3vw;
  }

  .linear-line{
    width: 0.2rem;
    height: 50%;
    background-color: lightgray;
    border-radius:0.1rem;
  }
  .color-purple {
    color: #432B99;
  }
  .big-font-size{
    font-size: 3rem;
    font-family: "PingFang SC";
  }
  .small-font-size{
    font-size: 1.7rem;
    font-family: "PingFang SC";
  }
  .img-icon-style{
    width: 2.5rem;
    height: 2.5rem;
  }
  .margin-left-style{
    margin-left: 0.5rem;
  }
  /*.bartop:last-child{*/
    /*margin-left: auto;*/
    /*margin-right: 2rem;*/
  /*}*/
  .save-area{
    margin-left: auto;
    margin-right: 2rem;
    display: flex;
    flex-direction: row;
    justify-content: center;
    align-items: center;
  }
</style>

```