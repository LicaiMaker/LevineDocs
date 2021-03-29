# vuex的使用
vuex是用来保存状态的，可以在组件中传值，常用凡是如下：
```
// bindnumber.js
import {UPDATE_NUMBER} from '../mutaions-type'
const state = {
  phoneNumber: ''
}
const mutations = {
  [UPDATE_NUMBER] (state, phoneNumber) {
    state.phoneNumber = phoneNumber
  }
}
const actions = {
  updateNumber ({commit}, payload) {
      commit(UPDATE_NUMBER, payload)
  }
}

export default {
  namespaced: true,
  state,
  mutations,
  actions
}

```
然后在store/index.js中引入这个文件：
```
//store/index.js
import Vue from 'vue'
import Vuex from 'vuex'
// import components...
import bindNumber from './modules/bindnumber'

Vue.use(Vuex)
// 引入模块
const modules = {bindNumber}
export default new Vuex.Store({
  modules: modules,
  state: {
  },
  getters: {},
  mutations: {},
  actions: {}
})

```
# 封装SessionStorage工具类
但是页面一刷新数据全没有了，因为state中的数据是保留在内存中的。需要使用LocalStorage或者SessionStorage来存贮state变量，所以现在我们来封装一个SessionStorage工具类(ss.js)：
```
// ss.js
// 这是对sessionStorage的封装
const ss = window.sessionStorage
export
default {
  getItem (key) {
    try {
      return JSON.parse(ss.getItem(key))
    } catch (err) {
    // 找不到key对应的值，就返回null
      return null
    }
  },
  setItem (key, val) {
    ss.setItem(key, JSON.stringify(val))
  },
  clear () {
    ss.clear()
  },
  keys (index) {
    return ss.key(index)
  },
  removeItem (key) {
    ss.removeItem(key)
  }
}

```

# state的持久化实现
改造一下bindnumber.js文件

```
// bindnumber.js
import {UPDATE_NUMBER} from '../mutaions-type'
// 引入sessionstorage工具类
import ss from '@/utils/ss'
const state = {
    // 从SessionStorage中读取
  phoneNumber: ss.getItem('phoneNumber')
}
const mutations = {
  [UPDATE_NUMBER] (state, phoneNumber) {
    state.phoneNumber = phoneNumber
    // 将参数值存入SessionStorage
    ss.setItem('phoneNumber', phoneNumber)
  }
}
const actions = {
  updateNumber ({commit}, payload) {
      commit(UPDATE_NUMBER, payload)
  }
}

export default {
  namespaced: true,
  state,
  mutations,
  actions
}

```
> 这样就能保证刷新的时候vuex中的值还在，那么在组件中如何使用state中的值呢？

# 在组件中使用state中的值
```
// phonenumber.vue
<template>
  <div>
        <el-input type="number" v-model="phoneNumber"  placeholder="请输入手机号码"
                  clearable></el-input>
  </div>
</template>

<script>
import {mapState} from 'vuex'
// import api from '@/utils/api'
export default {
  name: 'BindNumber',
  inject: ['reload'],
  data () {
    return {
      ...
    }
  },
  created () {
    ...
  },
  computed: {
    phoneNumber: {
      set (newVal) {
        this.updatePhoneNumber(newVal)
      },
      get () {
        return this.$store.state.bindNumber.phoneNumber
      }
    }
  },
  methods: {
    ...mapMutations('petInfo', {updatePhoneNumber: UPDATE_NUMBER}),
  }
}
</script>

<style scoped>
 ...
</style>

```
> 上面的computed中的phoneNumber中的set和get属性，能够将input和值绑定起来，页面刷新的时候，会调用get方法，从state中读取值（实际上从SessionStorage中读取）；在input中输入值时，会调用set方法，将值存到state中去（实际上存到SessionStorage中去）.

这样就能持久化vuex了