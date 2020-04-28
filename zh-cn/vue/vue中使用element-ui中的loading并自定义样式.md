## 使用element-ui中的loading
```
const loading = this.$loading({
        lock: true,
        text: 'loading',
        spinner: 'el-icon-loading',
        background: 'rgba(0, 0, 0, 0.7)'
      })

setTime(function(){
    loading.close()
},3000)
      
```

## 自定义样式和文字
- 1.在app.vue中定义一个样式
```css
.create-isLoading {
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    .el-loading-spinner {
      background: rgba(0, 0, 0, 0.7);
      padding: 2rem;
      border-radius: 0.4rem;
      width: auto;
      text-align: center;
      position: center;
      font-size: 4rem;
      i {
        color: #eee;
      }

      .el-loading-text {
        color: #eee;
        font-size: 2rem;
      }
    }
  }
```
- 2.在需要的页面中使用loading
```js
	showLoading: function (text) {
      const loading = this.$loading({
        lock: true,
        customClass: 'create-isLoading', //  *这里设置他的class名称,这里最重要
        text: text,
        spinner: 'el-icon-loading',
        background: 'rgba(0, 0, 0, 0)'
      })
      return loading
    },
    hideLoading: function (loading) {
      loading.close()
    }
```