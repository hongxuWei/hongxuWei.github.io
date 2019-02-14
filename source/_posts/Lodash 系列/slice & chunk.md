---
title: Lodash.slice & Lodash.chunk 源码阅读
date: 2019-02-14 16:30:00
tags: [JavaScript, Lodash, slice, chunk]
categories: JavaScript
thumbnail: /img/lodash.jpeg
---
# Lodash 源码阅读（二）

## slice

参数 `array` 为 数组类型， `start` 和 `end` 数字。

输出数组第 `start` 个值到 第 `end` 个值组成的数组，非法时时空数组。（`start`, `end` 不是索引值，而是*字面意义*的数组中的`第 N 个值`）

```JavaScript
function slice(array, start, end) {
  // array 长度为 0 或不存在直接返回空数组
  let length = array == null ? 0 : array.length
  if (!length) {
    return []
  }
  // start 默认值 0
  start = start == null ? 0 : start
  // end 默认值为 数组长度
  end = end === undefined ? length : end
  if (start < 0) {
    // start < 0 时的特殊处理
    // 目的使 start 在 0 ~ length 的范围内
    start = -start > length ? 0 : (length + start)
  }
  // end 大于数组长度时修改值为数组长度
  end = end > length ? length : end
  // end < 0 时的特殊处理
  if (end < 0) {
    end += length
  }
  // start 和 end 都在范围内后判断 length 的长度，如果 start > end 非法则输出空数组
  length = start > end ? 0 : ((end - start) >>> 0)
  // start 取整，因为 start 在小于零时已特殊处理为正数。若 start 此时为负数则不可用 >>> 取整
  start >>>= 0
  let index = -1
  // 分配内存
  const result = new Array(length)
  // index 0 ~ length
  while (++index < length) {
    // 循环赋值
    result[index] = array[index + start]
  }
  return result
}
```

## chunk

将一个数组按固定大小(`size`)分割，例如：
* chunk(['a', 'b', 'c', 'd'], 2) 输出为 [['a', 'b'], ['c', 'd']]
* chunk(['a', 'b', 'c', 'd'], 3) 输出为 [['a', 'b', 'c'], ['d']]

```JavaScript
function chunk(array, size) {
  // size 最小为 0
  size = Math.max(size, 0)
  const length = array == null ? 0 : array.length
  // length ”小于“ 1 或 size ”小于“ 1 返回空数组
  if (!length || size < 1) {
    return []
  }
  let index = 0
  let resIndex = 0
  // 分配内存，数组长度 按 length / size 向上取整
  const result = new Array(Math.ceil(length / size))
  while (index < length) {
    // 循环为 result 赋值
    result[resIndex++] = slice(array, index, (index += size))
  }
  return result
}
```

##### 浏览器调用

因为我没有找到在线使用 `lodash` 的网站，下面这段代码只是为了方便自己在浏览器里调用 `lodash` 的方法，加深理解。 

```Javascript
var script = document.createElement('script');
script.src = "https://cdn.bootcss.com/lodash.js/4.17.11/lodash.js";
document.body.appendChild(script);
```