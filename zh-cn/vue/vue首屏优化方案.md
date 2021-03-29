在vue中SPA项目中首页加载很慢，特别影响体验，下面来讲讲几个优化方面
### 1. 去掉.map文件
.map文件是为了我们在调试时可以定位到源码，但在生产环境中，我们是不需要的，况且，map文件非常大。在vue项目中的config/index.js中，将build选项中的`productionSourceMap`设为false：
```
// config/index.js
module.exports = {
  dev:{
     ...
  },
  build: {
       /**
     * Source Maps
     */
    // 设为false
    productionSourceMap: false,
    // https://webpack.js.org/configuration/devtool/#production
    devtool: '#source-map',

  }
```
这样重新编译`npm run build`，就能发现每个文件下就没有了map文件，大大的压缩了包的体积
### 2.开启gzip，压缩文件
使用gzip将大文件压缩，也会缩减包的大小。
- a.首先，在webpack中开启gzip功能  
同样是在`config/index.js`中build的下,将`productionGzip`设为true：
```javascript
// config/index.js
module.exports = {
  dev:{
     ...
  },
  build: {
       /**
     * Source Maps
     */
    // 设为true
    productionGzip: true,
    productionGzipExtensions: ['js', 'css'],
    ...
  }
```
- b.安装compression-webpack-plugin插件
```
npm install -D compression-webpack-plugin
```
> 注意：这里的compression-webpack-plugin版本应该怎么选呢？   

在vue2.x版本中使用1.1.11，vue3.x中可以选取最新版，我的项目使用的是vue2.x，所以我的安装命令如下：
```
npm install -D compression-webpack-plugin@1.1.11
```
另外在高版本compression-webpack-plugin中，先关配置还不一样
- c.配置CompressionWebpackPlugin插件相关参数
在webpack.prod.conf.js中：
```
//webpack.prod.conf.js
if (config.build.productionGzip) { // config/index.js中的build选项中的productionGzip
  const CompressionWebpackPlugin = require('compression-webpack-plugin')

  webpackConfig.plugins.push(
    new CompressionWebpackPlugin({
      asset: '[path].gz[query]',// 在高版本中的compression-webpack-plugin中要将asset改为filename
      algorithm: 'gzip',
      test: new RegExp(
        '\\.(' +
        config.build.productionGzipExtensions.join('|') +
        ')$'
      ),
      threshold: 10240, // 大于10k的文件将被压缩
      minRatio: 0.8,
      deleteOriginalAssets: false //是否删除源文件(dist目录中会存在源文件和被压缩后的gz文件,建议不要删除源文件)
    })
  )
}
```
重新编译后,`npm run build`后，发现dist目录中的static文件夹下面，大于10k的文件都生成了一个额外的gz文件。
- d.配置nginx

虽然前端项目中已经生成了压缩文件，但是需要nginx配置，通过nginx的配置，让浏览器直接解析.gz文件：
```
# 注意是在server里面，只针对这个server实例；如果将gzip的配置放在http{}中的话，就针对所有实例生效了
server{
    listen 80;
    ....
    gzip on;
    gzip_buffers 32 4K;
    gzip_comp_level 6;
    gzip_min_length 100;
    gzip_types application/javascript text/css text/xml;
    gzip_disable "MSIE [1-6]\."; #配置禁用gzip条件，支持正则。此处表示ie6及以下不启用gzip（因为ie低版本不支持）
    gzip_vary on;
}
```
然后重启nginx即可。
> 那么如何查看，nginx的gzip生效与否呢？

打开浏览器，访问你的网站，按F12，点击network，查看你刚刚在前端打包生成的gz文件，选中后查看response headers中的`Content-Encoding`参数的值是 `gzip`则表示nginx中的gzip配置生效。
> 参考地址：https://blog.csdn.net/qq_38143787/article/details/109155287
### 路由懒加载
在vue-router添加路由时候，使用懒加载方式：
```js
// router/index.js
// 懒加载方式简单写法
const Auth = () => import('@/components/Auth')
Vue.use(Router)

export default new Router({
  mode: 'history',
  routes: [
    {
      path: '/',
      name: 'Auth',
      component: Auth
    }]
})
```

另外，有一个非常高级的优化系列方案的文档：https://zhuanlan.zhihu.com/p/217731729