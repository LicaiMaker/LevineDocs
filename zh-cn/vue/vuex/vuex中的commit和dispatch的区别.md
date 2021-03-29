
- 1.store.dispatch('actions的方法', arg), 调用actions里的方法。 
- 2.commit 同步操作 this.store.commit('mutations的方法', arg)，调用mutations里的方法

> 举个列子
```
import { login, logout} from "@/api/user";

const state = {
  token: '',
};

const mutations = {
  SET_TOKEN: (state, token) => {
    state.token = token;
  },
};

const actions = {
  // 用户登出
  logout({ commit, dispatch, state }) {
  // 使用promise将结果传到前端页面，方便前端页面进行相应的展示，这是将数据层和页面分离
    return new Promise((resolve, reject) => {
          // 调用本模块的mutations方法
          commit("SET_TOKEN", ""); 
          // 调用其他模块的actions的方法，记住：需要添加{root:true}，不然就找不到这个方法了
          dispatch("app/delAllCachedViews", {}, { root: true });
          resolve();
    });
  },
};

export default {
  namespaced: true,
  state,
  mutations,
  actions,
};
```
> 注：vuex里面的actions,mutations,state的可以被单独提出一个js文件，如actions.js,mutations.js,state.js，再引入这些js文件即可