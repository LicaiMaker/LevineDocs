## async
> 1、async 作为一个关键字放到函数前面，用于表示函数是一个异步函数,async 函数返回的是一个promise 对象(可以使用then和catch等来获取输出结果了)
```javascript
async function timeout() {
    return 'hello world'
}
timeout().then(result => {
    console.log(result);
})
console.log('虽然在后面，但是我先执行');
```
> 2、Promise 有一个resolved，这是async 函数内部的实现原理。如果async 函数中有返回一个值 ,当调用该函数时，内部会调用Promise.solve() 方法把它转化成一个promise 对象作为返回
```
async function timeout(flag) {
    if (flag) {
        return 'hello world'
    } else {
        throw 'my god, failure'
    }
}
console.log(timeout(true))  // 调用Promise.resolve() 返回promise 对象。
console.log(timeout(false)); // 调用Promise.reject() 返回promise 对象。
```
如果函数内部抛出错误， promise 对象有一个catch 方法进行捕获。
```
timeout(false).catch(err => {
    console.log(err)
})
```

## await
> await后面可以放任何表达式，不过我们更多的是放一个返回promise 对象的表达式。注意await 关键字只能放到async 函数里面
```
function doubleAfter2seconds(num) {
    return new Promise((resolve, reject) => {
        setTimeout(() => {
            resolve(2 * num)
        }, 2000);
    } )
}
```
```
async function testResult() {
    let result = await doubleAfter2seconds(30);//会阻塞
    console.log(result);
}
现在调用testResult 函数
testResult();
```
> 参考地址：https://juejin.im/post/5c185c96518825721e5f2d50

- 有一种特殊情况下(后面的请求需要拿到前面的请求的返回值)，也可以使用async和await，最后将多个请求的最终结果返回,链式调用
```
getLocation: function(){
    return new Promise((resolve,reject)=>{
        api.get1().then(async res=>{
            var res1=await api.get2(res)
            var res2=await api.get3(res1)
            resolve(res2)
        }).catch(err=>{
            reject(err)
        })
    })
}
实现了用‘异步函数实现了同步的效果’
```