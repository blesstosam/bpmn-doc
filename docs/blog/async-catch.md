# 优雅捕获 async 错误

```
async function foo() {
    try {
        let res = await asyncFunc()
        try {
            let res2 = await anyncFunc2()
        } catch (e1) {
            // ...
        }
    } catch (e2) {
        // ...
    }
}
```

上述函数如果有多层嵌套, 在try catch 里嵌套 try catch 会变的很难看，可读性也会下降（参考回调地域）

如下，在网上发现的

```
async function errorCaptured(asyncFunc) {
    try {
        let res = await asyncFunc()
        return [null, res]
    } catch (e) {
        return [e, null]
    }
} 
```


上述写法就可以变成以下写法

```
async function foo() {
    let [err, res] = await errorCaptured(asyncFunc)
    if (err) {
        // ...
        return;
    }
    
    let [err, res] = await errorCaptured(asyncFunc2)
    if (err) {
        // ...
        return;
    }
}
```
