```
// common.js
const state = {
    openId: ''
}
export default {
    namespaced: true,
    state
}
```
// 使用rootState来访问即可
```javascript
// another.js
const actions = {
func2 ({state, commit, rootState}, phoneNumber) {
    // 请求获取验证码接口
    // let openId = state.common.openId
    // console.log('openId', this.getters.openId(rootState))
    console.log('opneid', rootState.common.openId)
}  
}
export default {
    namespaced: true,
    actions
}
```