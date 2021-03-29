### 1.
```
new Promise((resolve,reject)=>{
    resolve(...),
    ....
    reject(...)
})
```

### 2.
```
new Promise((resolve) => {
    resolve(...),
    ....
},(reject)=>{
    ....
    reject(...)
})
```

### 3.
```
return Promise.resolve(data)
或
return Promise.reject(error)
```