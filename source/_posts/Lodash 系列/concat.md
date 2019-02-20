---
title: Lodash.concat 源码阅读
date: 2019-02-19 15:00:00
tags: [JavaScript, Lodash, concat]
categories: JavaScript
thumbnail: /img/lodash.jpeg
---
# Lodash 源码阅读（四）

## concat

该方法的效果同 Array.concat

var array = [1];
var other = _.concat(array, 2, [3], [[4]]);
console.log(array.concat(2, [3], [[4]]));  // => [1, 2, 3, [4]]
console.log(other);  // => [1, 2, 3, [4]]

```JavaScript
import arrayPush from './_arrayPush.js';
import baseFlatten from './_baseFlatten.js';
import copyArray from './_copyArray.js';
import isArray from './isArray.js';

function concat() {
  var length = arguments.length;
  if (!length) {
    return [];
  }
  var args = Array(length - 1),
      array = arguments[0],
      index = length;

  while (index--) {
    args[index - 1] = arguments[index];
  }
  return arrayPush(isArray(array) ? copyArray(array) : [array], baseFlatten(args, 1));
}
```

### 工具函数 

#### _arrayPush

param1: `array`

param2: `values`

两个参数均为数组，将第二个数组中每个值 push 到 第一个参数后面

##### 使用示例

```Javascript
var array = [1, 2, 3, 4];
var values = [5, 6, 7, [8], "9"];
console.log(arrayPush(array, values)); // [1, 2, 3, 4, 5, 6, 7, [8], "9"]
```

##### 源码

```Javascript
function arrayPush(array, values) {
  var index = -1,
      length = values.length,
      offset = array.length;
  // 循环将 values 里的值挨个放到 array 中并返回 array
  while (++index < length) {
    array[offset + index] = values[index];
  }
  return array;
}
```

#### _copyArray

param1: `source` 为待复制（浅复制）的数组

param2: `array`  为目标数组或为空

##### 使用示例

```Javascript
copyArray([1, 2, 3]); // [1, 2, 3]
```

##### 源码

```Javascript
function copyArray(source, array) {
  var index = -1,
      length = source.length;

  array || (array = Array(length));
  while (++index < length) {
    // 浅复制
    array[index] = source[index];
  }
  return array;
}
```

#### _baseFlatten

param1: `array` 想要扁平化的数组

param2: `depth` 扁平化的深度

param3: `predicate` 判断是否需要扁平化的函数

param4: `isStrict`

param5: `result`

##### 使用示例

```Javascript
baseFlatten([1, [2], 3, [4, 5, [6, 7]]], 1); // [1, 2, 3, 4, 5, [6, 7]]
```

##### 源码

```Javascript
import arrayPush from './_arrayPush.js';
import isFlattenable from './_isFlattenable.js';

function baseFlatten(array, depth, predicate, isStrict, result) {
  var index = -1,
      length = array.length;

  predicate || (predicate = isFlattenable);
  result || (result = []);

  while (++index < length) {
    var value = array[index];

    // 如果当前深度大于 0 且当前值是可以扁平化的
    if (depth > 0 && predicate(value)) {
      if (depth > 1) {
        // 递归去扁平化数组
        // Recursively flatten arrays (susceptible to call stack limits).
        baseFlatten(value, depth - 1, predicate, isStrict, result);
      } else {
        // depth === 1 时该函数完成了扁平化，进行赋值操作
        arrayPush(result, value);
      }
    } else if (!isStrict) { // 限制哪些没有通过扁平化测试的值，如果不是严格型的就赋值，如果是严格型的就丢弃
      result[result.length] = value;
    }
  }
  return result;
}
```

#### _isFlattenable

param1: `value` 待判断是否可以做扁平化操作的值

##### 源码

```Javascript
import Symbol from './_Symbol.js';
import isArguments from './isArguments.js';
import isArray from './isArray.js';

/** Built-in value references. */
var spreadableSymbol = Symbol ? Symbol.isConcatSpreadable : undefined;

function isFlattenable(value) {
  return isArray(value) || isArguments(value) ||
    !!(spreadableSymbol && value && value[spreadableSymbol]);
}

```