[TOC]
# 创建项目命令  
`vue-init webpack projectname`(不能有大写字母)
然后一路敲回车就行
# build命令
```shell
cd project_dir#跳转到vue项目目录下 
npm install  
npm run build  
```
然后就能在vue项目目录下生成一个dist目录，里面包含了一个static文件夹和index.html

# 如何让局域网类设备都能看到我的vue项目？
在package.json中的scripts下面的dev中添加--host 0.0.0.0:
```vue
"scripts": {
    "dev": "webpack-dev-server --host 0.0.0.0 --inline --progress --config build/webpack.dev.conf.js",
    ...
  },
```
然后执行：
`
npm run dev
`
即可，能通过电脑的ip地址加端口号访问了：
例：192.168.1.6:8080

# vue项目为什么老是启动失败？
使用`npm run dev`时，老是提示错误：
```vue
13 verbose stack Error: frontend@1.0.0 dev: `webpack-dev-server --inline --progress --config build/webpack.dev.conf.js`
13 verbose stack Exit status 1
13 verbose stack     at EventEmitter.<anonymous> (C:\Users\summer\AppData\Roaming\npm\node_modules\npm\node_modules\npm-lifecycle\index.js:332:16)
13 verbose stack     at emitTwo (events.js:126:13)
13 verbose stack     at EventEmitter.emit (events.js:214:7)
13 verbose stack     at ChildProcess.<anonymous> (C:\Users\summer\AppData\Roaming\npm\node_modules\npm\node_modules\npm-lifecycle\lib\spawn.js:55:14)
13 verbose stack     at emitTwo (events.js:126:13)
13 verbose stack     at ChildProcess.emit (events.js:214:7)
13 verbose stack     at maybeClose (internal/child_process.js:915:16)
13 verbose stack     at Process.ChildProcess._handle.onexit (internal/child_process.js:209:5)
```
这是因为webpack3.0的问题，将webpack使用2.9.1的版本即可
解决办法：
- 1.`npm uninstall webpack-dev-server`
- 2.`npm install webpack-dev-server@2.9.1`
- 3.`npm run dev`

> 参考地址：https://www.cnblogs.com/zdz8207/p/vue-webpack-dev-server.html