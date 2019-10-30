---
title: apply call bind解析
date: 2019-10-30 11:00:00
tags: [JavaScript, apply, call, bind]
categories: JavaScript
---

## 1分钟关键速记
### 相同点：
1. 都可以改变函数执行上下文。
2. 第一个参数都是函数执行上下文

### 不同点：
1. **函数执行参数传参方式不同**，`apply` 是用数组而 `call`,和 `bind` 是用单独传参
2. **返回值不同**，`apply` 和 `call` 都是返回函数执行后的返回值而 `bind` 返回值是改变了函数执行上下文的函数

## Function.prototype.call
call() 方法使用一个指定的 this 值和单独给出的一个或多个参数来调用一个函数。

### 语法
fun.call(thisArg, arg1, arg2, ...)

#### thisArg
在 fun 函数运行时指定的 this 值。
```javascript
if (thisArg == undefined | null) {
	this = window
}
// 所有类型
if (thisArg == number | boolean | string | ...others) {
	this == new Number()|new Boolean()| new String() | new Others()
}
```
#### arg1, arg2, ...
指定的参数列表。


## Function.prototype.apply
apply() 方法使用一个指定的 this 值和一个数组组成的参数列表来调用一个函数。

### 语法
fun.call(thisArg, [argsArray])

#### thisArg
在 fun 函数运行时指定的 this 值。
方式同apply

#### argsArray
一个数组或者类数组对象，其中的数组元素将作为单独的参数传给 func 函数。如果该参数的值为 null 或  undefined，则表示不需要传入任何参数。从ECMAScript 5 开始可以使用类数组对象


## Function.prototype.bind 
bind()方法创建一个新的函数，在bind()被调用时，这个新函数的this被bind的第一个参数指定，其余的参数将作为新函数的参数供调用时使用。

### 语法
fun.bind(thisArg, arg1, arg2, ...)

#### thisArg
在 fun 函数运行时指定的 this 值。
方式同apply

#### arg1, arg2, ...
指定的参数列表。

## 进阶：实现 call, apply, bind
### call
```javascript
Function.prototype.myCall = function (context, ...args) {
  // 上下文判断及赋值
  if (context === null || context === undefined) {
    context = window
  } else {
    context = Object(context)
  }
  // 给 context 新增一个不可枚举的且唯一的属性
  // 隐式绑定 this
  const _this = this
  const localProperty = Symbol('localProperty')
  Object.defineProperty(context, localProperty, {
    enumerable: false,
    value: _this
  })
  // 执行函数
  const result = context[localProperty](...args)
  // 删除新增的属性
  delete context[localProperty]
  return result
}
```
### apply
```javascript
Function.prototype.myApply = function (context, args) {
  // 如果参数不是 array like 的就抛出错误
  if (!isArrayLike(args)) {
    throw new TypeError('CreateListFromArrayLike called on non-object')
  }
  // 上下文判断及赋值
  if (context === null || context === undefined) {
    context = window
  } else {
    context = Object(context)
  }
  // 给 context 新增一个不可枚举的且唯一的属性
  // 隐式绑定 this
  const _this = this
  const localProperty = Symbol('localProperty')
  Object.defineProperty(context, localProperty, {
    enumerable: false,
    value: _this
  })
  // 执行函数
  const result = context[localProperty](...args)
  // 删除新增的属性
  delete context[localProperty]
  return result
}
```
### bind

MDN polyfill

```javascript
if (!Function.prototype.bind) {
  Function.prototype.bind = function(oThis, ...aArgs) {
    if (typeof this !== 'function') {
      // closest thing possible to the ECMAScript 5
      // internal IsCallable function
      throw new TypeError('Function.prototype.bind - what is trying to be bound is not callable');
    }

    const fToBind = this;
    const fNOP = function() {};
    const fBound  = function(...fBoundArg) {
      // this instanceof fBound === true时,说明返回的fBound被当做new的构造函数调用
      return fToBind.apply(
        this instanceof fBound
          ? this
          : oThis,
          // 获取调用时(fBound)的传参.bind 返回的函数入参往往是这么传递的
        aArgs.concat(fBoundArg)
      );
    };

    // 维护原型关系
    if (this.prototype) {
      // 当执行Function.prototype.bind()时, this为Function.prototype 
      // this.prototype(即Function.prototype.prototype)为undefined
      fNOP.prototype = this.prototype; 
    }
    // 下行的代码使fBound.prototype是fNOP的实例,因此
    // 返回的fBound若作为new的构造函数,new生成的新对象作为this传入fBound,新对象的__proto__就是fNOP的实例
    fBound.prototype = new fNOP();

    return fBound;
  };
}
```

## 参考文档
* [MDN - call](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/call)
* [MDN - apply](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/apply)
* [MDN - bind](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/bind)