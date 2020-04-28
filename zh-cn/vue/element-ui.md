> #### 安装element-ui
>
> `npm i element-ui -S`

> #### 使用

```javascript
import Vue from 'vue';
import ElementUI from 'element-ui';
import 'element-ui/lib/theme-chalk/index.css';
import App from './App.vue';

Vue.use(ElementUI);

new Vue({
  el: '#app',
  render: h => h(App)
});
```
> 参考链接：https://element.eleme.cn/#/zh-CN/

> 注意事项：和vue结合使用时，要保证vue的包和element-ui的包在同一个目录下(frontend/node_modules)