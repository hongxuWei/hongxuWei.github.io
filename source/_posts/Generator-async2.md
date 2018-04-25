---
title: Generator & async (2)
date: 2018-04-25 21:00:59
tags: Generator async
categories: ES6 Generator async
---
# Generator

## 4. for...of 循环

`for...of` 循环可以自动遍历 Gnerator 函数生成的 `Iterator` 对象，而且这个时候不需要调用 `next` 方法。

```JavaScript
function* foo() {
    yield 1;
    yield 2;
    yield 3;
    return 4
}

for(let value of foo()) {
    console.log(value);
}
/* 1 2 3 */
```
依次显示3个 `yield` 表达式的值。
**Note: 一旦 `Next` 方法返回的对象中 `done` 属性为 `true`， `for...of` 循环就会终止，且不包含该返回对象，所以上述代码 `return` 语句中返回的 `4` 不包含在 `for...of` 循环里**

```JavaScript
// 循环打印斐波那契数列
function* fibonacci() {
    let [prev, curr] = [0, 1];
    for(;;) {
        [prev, curr] = [curr, prev + curr];
        yield curr;
    }
}
for (let n of fibonacci()) {
    if (n > 1000) break;
    console.log(n);
}
```

除了 for...of 之外， 扩展运算符（...）、结构赋值和 `Array.from` 方法内部调用的都是遍历器接口，所以它们都可以将 Generator 函数返回的 Iterator 对象作为参数。

```JavaScript
function* num() {
    yield 1
    yield 2
    return 3
}
// 扩展运算符
[...num()] //[1, 2]

Array.from(num()) //[1, 2]

let [x, y] = num();
x // 1
y // 2

for (let n of num()) {
    console.log(n)
}
// 1
// 2
```

## 5. Generator.prototype.throw()

在函数体外抛出错误，然后在 Geneartor 函数体内部捕获。

**Note: `throw` 方法会默认执行一次 `next`**

## 6. Generator.prototype.return()

可以返回给定的值，并终结遍历 Generator 函数。

**Note: 如果函数内部有 `try...finally` 代码块，那么 `return` 方法会推迟到 `finally` 代码块执行完再执行**

## 7. next() throw() return() 的共同点

`next()` `throw()` `return()` 三个方法本质是同一件事，可以放在一起理解，它们的作用都是让 Generator 函数恢复执行，并且使用不同的语句替换 `yield` 表达式。

`next()` 是将 `yield` 表达式替换成一个值。

`throw` 是将 `yield` 表达式替换成 `throw` 语句

`return` 是将 `yield` 表达式替换成 `return` 语句

## 8. yield* 表达式

如果在 Generator 函数内部抵用另外一个 Generator 函数，默认情况下是没有效果的。这时候就需要用到 `yield*` 表达式，用来在一个 Generator 函数里面执行另一个 Generator 函数。

**任何数据结构只要有 Iterator 接口，就可以被`yield*` 遍历。**