---
title: Lodash.compact 源码阅读
date: 2019-02-18 14:00:00
tags: [JavaScript, Lodash, compact]
categories: JavaScript
thumbnail: /img/lodash.jpeg
---
# Lodash 源码阅读（三）

## compact

将一个数组变为紧凑数组

Creates an array with all falsey values removed. The values `false`, `null`, `0`, `""`, `undefined`, and `NaN` are falsey.

其思想就是将传进来的数组做一个循环，把所有不是 falsey 的部分挨个放到新的数组中。

```JavaScript
function compact(array) {
  var index = -1,
      length = array == null ? 0 : array.length,   // 对 length 有效值做处理
      resIndex = 0,
      result = [];

  // 对传参的数组做循环
  while (++index < length) {
    var value = array[index];
    if (value) {
      // 将非 falsey 的值放到新数组 result 里
      result[resIndex++] = value;
    }
  }
  return result;
}
```

## 自己的简单实现

```JavaScript
function compact(array) {
  // array
  if (array == null) {
    return [];
  }
  return array.filter(value => value);
}
```
