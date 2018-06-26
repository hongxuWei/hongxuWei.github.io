---
title: 想开个 ES6 的坑
date: 2018-04-20 21:32:00
tags: [JavaScript, Promise]
categories: JavaScript ES6
---

# ES6 Promise

`Promise` 是 ES6 的异步编程解决方案。

Promise 有 3 种状态 `pending` （正在进行）`fulfilled` (成功) `rejected` (失败)。 只有异步操作的结果可以决定状态。

其中 这3种状态的转换关系是 `pending` -> `fulfilled`, `pending` -> `rejected`. 而 fulfilled 和 rejected 这两种状态一旦形成就不可改变。

一图胜千言
![Promise图解](promise.png)

## 1. 用法

Promise 对象是一个构造函数，用来生成 Promise 实例。

```JavaScript
const promise = new Promise((resolve, reject)=>{
    /* do some thing */
    if(/* promise success */) {
        resolve(value);
    } else {
        reject(error)
    }
})
```

`resolve` 函数将 `Promise` 对象状态由 `pending` 变为 `fulfilled` 状态， 在异步操作成功时调用，并将异步操作的结果作为参数传递。

`reject` 函数的作用是，将 `Promise`对象的状态从 `pending` 变为 `rejected` 状态，在异步操作失败时调用，并将异步操作报出的错误作为参数传递。

## 2. Promise.prototype.then()
Promise 实例生成之后就可以调用 then 方法分别指定 fulfilled 状态和 rejected 状态的回调函数。

```JavaScript
promise.then((value)=>{
    /* do some thing when success */
}, (error)=>{
    /* do some thing when failed */
})
```
如上所示，then 方法可以接受两个参数，分别对应的是 `fulfilled` 状态的回调函数和 `rejected` 状态的回调函数。其中 `rejected` 状态的回调函数是可选参数。

## 3. Promise.prototype.catch()
相当于 `promise.then(null, rejection)` 用于指定异步操作失败时候的回调函数。

**注意：如果 Promise 的状态已经是fulfilled的话，再抛出错误则不会调用 rejection函数**

**Promise对象的错误具有“冒泡”性质，会一直向后传递，知道被捕获为止**

## 4. Promise.prototype.finnaly()
`finally` 方法用于不论 Promise 对象的状态如何，都会执行的操作。（ES2018)

## 5. Promise.all()
用于将多个 Promise 实例，包装成一个新的 Promise 实例。

```JavaScript
const p = Promise([p1, p2, p3]);
```
Promise.all 方法接受一个数组作为参数，数组内都是 Promise 实例。 如果不是就会调用 Promise.resolve 方法，将参数转为 Promise 实例。

（假设成功为true, 失败为false）
`p = p1 && p2 && p3` 即 p1, p2, p3的状态都是 fulfilled 时 p 的状态才是 fulfilled。只要其中一个状态是 rejected, 那么 p 的状态就为 rejected。

## 6. Promise.race()
Promise.race 方法同 Promise.all 类似。只不过正如方法名一样，竞争态的 Promise，一旦其中有任何一个 Promise 的状态确定了，无论是 fulfilled 还是 rejected Promise.race 的状态也就和其一样。

## 7. Promise.resolve()
将现有对象转为 Promise 对象
* 参数是 `Promise` 实例，该方法不做任何处理，直接返回该实例。
* 参数是一个 `thenable` 对象，该方法会将对象转为 Promise 对象，然后执行 thenable 对象的 then 方法。
* 参数不是具有 then 方法的对象或根本不是对象，该方法返回一个新的 Promise 对象，状态为 `fulfilled`。
* 没有参数，该方法返回一个新的 Promise 对象，状态为 `fulfilled`。

## 8. Promise.reject()
该方法会返回一个新的 Promise 实例，该实例的状态为 rejected。

**注意：该方法的参数，会作为reject的理由变成后续方法的参数。**

```JavaScript
const thenable = {
    then(resolve, reject) {
        reject('error');
    }
}
Promise.reject(thenable).catch(e=>{
    console.log(e === thenable)
})
// true
```