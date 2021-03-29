> 这篇文章是为了自定义一个message提示tip框，编写插件，自定义一些操作，来快速的调用这个插件
message有四种状态：success，error， info，warn

先看看我的目录结构：
```
frontend //项目根目录
---src
    ---plugins //插件根目录
        ---message
            ---iconfont
                ---iconfont.css //icon字体图标
            ---message.js  //message插件
            ---Message.vue  //message的样式文件
```
### 一.编写message.vue
先将提示框的的样式通过vue编写出来，其中包含了控件的操作，样式等
```html
// message.vue
<template>
  <div class="message-box" :style="styles" v-if="isShow">
    <i class="iconfont" :class="icon"></i>
    <span>{{message}}</span>
  </div>
</template>

<script>
export default {
  name: 'Message',
  props: ['type', 'message'],
  data () {
    return {
      isShow: false,
      types: {
        // 成功
        success: {
          backgroundColor: '#f0f9eb',
          borderColor: '#e1f3d8',
          color: '#67c23a',
          icon: 'icon-success'
        },
        // 错误
        error: {
          backgroundColor: '#fde2e2',
          borderColor: '#fef0f0f0',
          color: '#f56c6c',
          icon: 'icon-error'
        },
        // 警告
        warn: {
          backgroundColor: '#fdf6ec',
          borderColor: '#faecd8',
          color: '#e6a23c',
          icon: 'icon-warning'
        },
        // 提示
        info: {
          backgroundColor: '#edf2fc',
          borderColor: '#e4e7ed',
          color: '#909399',
          icon: 'icon-info'
        }
      }
    }
  },
  computed: {
    icon () {
      return this.types[this.type].icon
    },
    styles () {
      const styles = {...this.types[this.type]}
      // 删除用不到的icon属性
      delete styles.icon
      return styles
    }
  },
  methods: {
    show () {
      this.isShow = true
    },
    hide () {
      this.isShow = false
    }
  }
}
</script>

<style lang="scss" scoped>
// 引入字体图标库
@import "iconfont/iconfont.css";
.message-box{
  width: 10rem;
  min-height: 1.5rem;
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
  padding: 8px 10px;
  border-radius: 5px;
  box-shadow: 0 0 10px gray;
  font-size: 0.8rem;
  color:white;
  font-family: 'Consolas', 'Deja Vu Sans Mono', 'Bitstream Vera Sans Mono', monospace;
  .iconfont {
    margin-right: 10px;
  }
}
</style>

```
> 对上面的代码有以下几点解释：
- 1.在data中定义了types，里面包含四种状态下的样式，其中
```
 backgroundColor: '#fde2e2',// message框的背景颜色
 borderColor: '#fef0f0f0',// 边框颜色
 color: '#f56c6c'// 字体颜色
 icon: 'icon-warning'//字体图标
```
> 上面的字体图标icon-xx是阿里巴巴字体图标库中的图标，我是通过font-class引用的方式(将生成的class文件下载到本地，然后引入到项目中去)将我需要的图标引入到我的插件中，具体使用方法，详见https://www.iconfont.cn/help/detail?spm=a313x.7781069.1998910419.d8cf4382a&helptype=code

- 2.div使用了:styles绑定样式，i标签使用了:class绑定了字体图标样式

div中的:styles就是绑定的内容就是通过computed中的styles和icon两个方法绑定的，值得注意的是：  

a.如果这两个方法在methods中的话，在标签中引用的时候是需要加上括号的，在computed中则不需要(因为在computed中表示的是属性计算值，直接按名称引用即可)；  

b.i标签中之所以使用:class绑定，是因为iconfont的格式要求如此，`<i class="iconfont icon-xxx"></i>`前面的iconfont不能少，icon-xxx是具体的字体图标
- 3.在css中通过`@import "iconfont/iconfont.css";`引入了阿里巴巴图标库的iconfont(我已提前将其下载到本地)
- 4.message中通过isShow属性和show,hide方法控制这显示与否


### 二.编写message.js插件
```
// 引入message.vue
import Message from './Message.vue'
export default {
  install (Vue, options) {
    // 继承自Message
    const VueMsg = Vue.extend(Message)
    let vm = null // 代表Message的实例对象
    // 定义message的显示方法
    const showMessage = ({text, type, timeout = 2000}) => {
      // 使用promise是为了一些需要在隐藏之后需要做一些操作的业务
      return new Promise(resolve => {
        if (!vm) {
          vm = new VueMsg()
          // 将实例对象挂载到Dom节点上
          vm.$mount()
          // 将实例对象的根节点挂载到body中,vm.$el就是实例对象的根节点
          document.querySelector('body').appendChild(vm.$el)
        } else {
          vm.hide()
          // 清除定时器
          clearTimeout(showMessage.timer)
        }

        vm.type = type
        vm.message = text

        vm.show()
        showMessage.timer = setTimeout(() => {
          vm.hide()
          // 在hide之后使用resolve，表示message隐藏了
          resolve()
        }, timeout)
      })
    }

    Vue.Message = Vue.prototype.$message = showMessage
  }
}

```
> 上面值得注意的几点：
- 1.通过Vue.extend方法得到Message的对象
- 2.在showMessage方法中使用了Promise，并在setTimeout方法结束后使用了resolve，这样就能在调用处，获知何时message会隐藏，以及隐藏后可以干点其他操作:
```
// another.vue
this.$message({
   type: 'success',
   text: '成功'
 }).then(() => {// 连续调用
   this.$message({
     type: 'error',
     text: '失败'
 })
```
- 3.在showMessage方法一开始做了vm判空操作，是为了不重复创建vm对象，并且在创建vm后将其挂载到DOM中并插入到body标签中；在vm不为空的情况下，调用hide方法并且清除了定时器，这样保证只有一个定时器存在。值得注意的是，我们将定时器赋值给了showMessage.timer，这样是为了清除它。

> 思考：如果在其他vue中，调用this.$message()时，需要传递type和以及timeout，是不是会导致代码冗长，我希望的方式是:
```
this.$message.success('success')
this.$message.error('error')
...
```
这种方式要好看很多，那么修改如下：
```
import Message from './Message.vue'
export default {
  install (Vue, options) {
    const VueMsg = Vue.extend(Message)
    let vm = null // 代表message的实例对象

    const showMessage = ({text, type, timeout = 2000}) => {
      // 使用promise是为了一些需要在隐藏之后需要做一些操作的业务
      return new Promise(resolve => {
        if (!vm) {
          vm = new VueMsg()
          // 将实例对象挂载到Dom节点上
          vm.$mount()
          // 将实例对象的根节点挂载到body中,vm.$el就是实例对象的根节点
          document.querySelector('body').appendChild(vm.$el)
        } else {
          vm.hide()
          // 清除定时器
          clearTimeout(showMessage.timer)
        }

        vm.type = type
        vm.message = text

        vm.show()
        showMessage.timer = setTimeout(() => {
          vm.hide()
          // 在hide之后使用resolve，表示message隐藏了
          resolve()
        }, timeout)
      })
    }

    showMessage.success = (message, timeout) => {
        showMessage({
         type: 'success',
         text: message,
         timeout
       })
    }
    
    showMessage.error = (message, timeout) => {
        showMessage({
         type: 'error',
         text: message,
         timeout
       })
    }
    ...

    Vue.Message = Vue.prototype.$message = showMessage
  }
}

```
> 上面虽然为四种状态下都封装了方法，确实方面调用了，但是代码更加冗长重复了，那么再改进一下：
```
import Message from './Message.vue'
export default {
  install (Vue, options) {
    const VueMsg = Vue.extend(Message)
    let vm = null // 代表message的实例对象

    const showMessage = ({text, type, timeout = 2000}) => {
      // 使用promise是为了一些需要在隐藏之后需要做一些操作的业务
      return new Promise(resolve => {
        if (!vm) {
          vm = new VueMsg()
          // 将实例对象挂载到Dom节点上
          vm.$mount()
          // 将实例对象的根节点挂载到body中,vm.$el就是实例对象的根节点
          document.querySelector('body').appendChild(vm.$el)
        } else {
          vm.hide()
          // 清除定时器
          clearTimeout(showMessage.timer)
        }

        vm.type = type
        vm.message = text

        vm.show()
        showMessage.timer = setTimeout(() => {
          vm.hide()
          // 在hide之后使用resolve，表示message隐藏了
          resolve()
        }, timeout)
      })
    }

    // 定义一些快捷函数
    const fn = (type, message, timeout) => {
      return Promise.resolve(
        showMessage({
          type,
          text: message,
          timeout
        }))
    }
    showMessage.success = fn.bind(null, 'success')
    showMessage.error = fn.bind(null, 'error')
    showMessage.info = fn.bind(null, 'info')
    showMessage.warn = fn.bind(null, 'warn')

    Vue.Message = Vue.prototype.$message = showMessage
  }
}

```
> 定义的fn函数是公共的函数，其内部仍然通过Promise返回。下面的四个函数采用bind方法，将type值传入其中，这样就能得到这四个不同的函数。bind函数会生成一个新的函数，第一个参数表示其上下文对象，后面的参数表示调用者的参数。让我们来看看其调用格式：
```
this.$message.success('哈哈哈').then(()=>{
    this.$message.error('xx')
    ...
})
或者
this.$message.success('ss', 5000)// 第二个参数表示timeout
```

> 注：**这种循序渐进的方式非常值得借鉴.(基本功能实现->通过调用方式改进(对象判空与清除定时器，添加promise返回,bind函数缩减代码量))**

### 3.在main.js中注册插件
```
// 这里引入的是js文件
import message from './plugins/message/message'

Vue.config.productionTip = false
Vue.use(message)

/* eslint-disable no-new */
new Vue({
  el: '#app',
  store,
  router,
  components: { App },
  template: '<App/>'
})

```

### 拓展：可以为Message.vue添加动画，显示更为顺滑（使用transition组件）：
```
<template>
  <transition name="slide">
    <div class="message-box" :style="styles" v-if="isShow">
      <i class="iconfont" :class="icon"></i>
      <span>{{message}}</span>
    </div>
  </transition>
</template>

<script>
export default {
    ...
}
</script>

<style lang="scss" scoped>
@import "iconfont/iconfont.css";
.message-box{
 ...
  transform:translateX(-50%);
  ...
}
  .slide-enter-active,
  .slide-leave-active {
    transition: all 0.4s;
  }
  .slide-enter,
  .slide-leave-to {
    opacity: 0;
    <!--这里很重要，因为其本身已经有了translateX(-50%),必须保持x轴的状态-->
    transform: translate(-50%,-100%);
  }
</style>

```