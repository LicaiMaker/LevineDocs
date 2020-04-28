## vue去掉#号与刷新vue的当前页面
在vue中如果想要去掉访问路径中#号，可以在router中设置一下mode即可
```js
export default new Router({
    mode: 'history',
    routes: [...]
})    
```
虽然井号去掉了，但是却不能刷新页面了，一旦刷新或者在浏览器中访问该页面都会报404错误。
此时需要nginx来解决。一共有两种nginx配置方法来解决。
- 1 
```nginx
 location / {
        root /home/OldManInfo_Flask/frontend/dist;
        index index.html;
        try_files $uri $uri/ /index.html;
        autoindex on;
        autoindex_exact_size on;
        autoindex_localtime on;
    }

```
> 使用try_files  try_files $uri $uri/ /index.html;如过我的vue有页面请求url是：http://www.example.com/page,因为vue是单页面应用，如过当前的page文件找不到，就找page/目录，再找不到就找首页，将交给首页的路由处理

- 2.
```nginx
    location / {
        root /home/OldManInfo_Flask/frontend/dist;
        index index.html;
        try_files $uri $uri/ @router;
        autoindex on;
        autoindex_exact_size on;
        autoindex_localtime on;
    }

    location @router {
       rewrite ^.*$ /index.html last;
    }

```
> 直接显示的指定index的路由去处理这个url