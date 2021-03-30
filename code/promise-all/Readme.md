---
title: "如何手写实现 Promise.all"
date: 2021-03-26
---

有一次头条面试，一道手写题目是：如何手写实现 `promise.all`。

我从来没有想过要手写实现 promise.all 函数，稍微一想，大概就是维护一个数组，把所有 promise 给 resolve 了之后都扔进去，这有啥子好问的。没想到，一上手还稍微有点棘手。

先来看一个示例吧:

``` js
await Promise.all([1, Promise.resolve(2)])
//-> [1, 2]

await Promise.all([1, Promise.reject(2)])
//-> Throw Error: 2
```

1. 传入一个数组，可包含 Promise，也可包含普通数据
1. 数组中 Prmise 并行执行
1. 但凡有一个 Promise 被 Reject 掉，Promise.all 失败

``` js
function pAll (promises) {
  return new Promise((resolve, reject) => {
    // 结果用一个数组维护
    const r = []
    const len = promises.length
    let count = 0
    for (let i = 0; i < len; i++) {
      // Promise.resolve 确保把所有数据都转化为 Promise
      Promise.resolve(promises[i]).then(o => { 
        // 因为 promise 是异步的，保持数组一一对应
        r[i] = o;

        // 如果数组中所有 promise 都完成，则返回结果数组
        if (++count === len) {
          resolve(r)
        }
        // 当发生异常时，直接 reject
      }).catch(e => reject(e))
    }
  })
}
```

为了测试，实现一个 sleep 函数

``` js
const sleep = (seconds) => new Promise(resolve => setTimeout(() => resolve(seconds), seconds))
```

以下示例进行测试，没有问题

``` js
pAll([1, 2, 3]).then(o => console.log(o))
pAll([
  sleep(3000),
  sleep(2000),
  sleep(1000)
]).then(o => console.log(o))
pAll([
  sleep(3000),
  sleep(2000),
  sleep(1000),
  Promise.reject(10000)
]).then(o => console.log(o)).catch(e => console.log(e, '<- Error'))
```