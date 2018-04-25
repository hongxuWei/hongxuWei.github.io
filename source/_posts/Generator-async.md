---
title: Generator & async （1）
date: 2018-04-21 22:02:36
tags: Generator async
categories: ES6 Generator async
---
# Generator

`Generator` 是 ES6 提出一种异步解决方案。

形式上 Generator 函数和普通函数有两个不同点。
* `function` 关键字和函数名之间有一个星号，如： `function* demo`;
* 内部函数使用 `yield` 表达式，定义不同的内部状态。

```JavaScript
function* demo(){
    yield 'hi';
    yield 'second';
    return 'end';
}

var test = demo();
test.next(); //{value: 'hi', done: false}
test.next(); //{value: 'second', done: false}
test.next(); //{value: 'end', done: true}
test.next(); //{value: undefined, done: true}
```
调用 `demo` 后函数并不会执行，返回的也不是函数运行的结果，而是一个指向内部状态的指针，只有当调用遍历器对象的 `next()` 方法，内部指针就从函数头或者上一次停下来的地方执行，知道遇到下一个 yield 表达式或者 return 语句为止。

## 1. yield 表达式
类似 Generator 函数的一个内部暂停标志。

Generator 函数就像一条马路，而 yield 是这条马路上的红绿灯路口，next 方法是站在每个路口指挥的交警。只有交警同意车辆通过后，才会执行相应的函数。

next 方法的运行逻辑有以下几点：
* 遇到 `yield` 就暂停后面的操作，并将紧跟在 `yield` 后的表达式的值，作为返回对象的 `value` 属性值
* 下次调用 `next` 方法是，再继续向下执行， 知道遇到下一个 `yield` 表达式或者 `return` 语句
* 如果函数没有 `return` 语句，那么返回对象的 `value` 属性值为 `undefined`

`yield` 和 `return` 的不同点在于，一个函数的 `return` 语句最多只能执行一次，而 `yield` 可以执行多次。 `yield` 会使函数暂停执行，下次执行的时候会从该位置继续向后执行，而 `return` 不具有记忆性。

**注意：Generator 函数可以不必使用 yield。但是如果使用 yield 就必须在 Generator 函数里面，否则会报错。<br>`yield` 表达式如果用在另一个表达式之中，必须添加在圆括号里面。 `console.log('Hello' + (yield));`**


## 2. Generator 与 Iterator

任意一个对象的 `Symbol.iterator` 方法，等于该对象的遍历器生成函数，调用该函数会返回该对象的一个遍历器对象。

Generator 函数执行后，返回一个遍历器对象。该对象本身也具有 `Symbol.iterator` 属性，执行后返回自身。

```JavaScript
function* demo(){
    // do some thing
}
var g = demo();
g[Symbil.iterator] === g; // true
```

## 3. next 方法的参数

`yield` 表达式本省没有返回值，或者说总是返回 `undefined`。 `next` 方法可以带一个参数，该参数就会被当做上一个 `yield` 表达式的返回值。

```JavaScript
function* f(){
    for( var i = 0; true; i++){
        var reset = yield i;
        if(reset) {
            i = -1;
        }
    }
}

var g = f();
g.next(); // {value: 0, done: false}
g.next(); // {value: 1, done: false}
g.next(true); // {value: 0, done: false}
```