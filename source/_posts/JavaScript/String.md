---
title: JS 基础之：String
date: 2018-07-12 08:57:43
tags: [JavaScript]
categories: JavaScript
---

之前也有总结和重新学习 **String** 的所有方法。长时间不去记忆又有所忘记。今天用博客的方式记录下来，最后用一个思维导图的形式记下来串联起来，巩固自己的记忆。

## 属性

### String.length

返回一个字符串的长度。这个无需多讲。只是要记住静态的 `String.length` 值为 `1`。

## 方法

### String.prototype.indexOf()

**语法**
> **str.indexOf(searchValue[, fromIndex])**

**indexOf()** 方法返回调用 String 对象中从 fromIndex 开始从左向右搜索直到第一次出现指定值的索引位置。如果未找到，返回 -1。

`searchValue`
表示需要被查找的字符串

`fromIndex` 
表示从 `str` 的这个位置开始向后查找，可以是任意整数（如果是小数则只保留整数部分）。默认值是 `0`。如果该值小于 `0` 则当做 `0`，如果该值大于字符串长度则返回 `-1`，除非 `searchValue = ""` 此时返回 `str.length`。

```JavaScript
// fromIndex 的内部机制可以用如下伪代码表示：
String.prototype.indexOf = function(searchValue, fromIndex) {
    fromIndex = fromIndex || 0; // 默认值为 0

    // 如果 fromIndex 大于字符串长度
    // 则判断需查找字符串是不是空字符串
    // 如果是则返回 str.length，如果不是则返回 -1。
    if (fromIndex > this.length) {
        return searchValue === "" ? this.length : -1;
    }

    // 如果 fromIndex 小于 0 则当做 0
    if (fromIndex < 0) { 
        fromIndex = 0
    }
    
    fromIndex = fromIndex >>> 0; // 取整
}
```

---

### String.prototype.lastIndexOf()

与 `indexOf` 相似不过是从右向左搜索。

**语法**
> **str.lastIndexOf(searchValue[, fromIndex])**

**lastIndexOf()** 方法返回调用 String 对象中从 fromIndex 开始从右向左搜索直到第一次出现指定值的索引位置。如果未找到，返回 -1。

`searchValue`
表示需要被查找的字符串

`fromIndex` 
表示从 `str` 的这个位置开始向后查找，可以是任意整数（如果是小数则只保留整数部分）。默认值是 `str.length`。如果该值小于 `0` 则返回 `-1`，除非 `searchValue = ""` 此时返回 `0`。如果该值大于字符串长度当做 `str.length`。

<!--```JavaScript
// fromIndex 的内部机制可以用如下伪代码表示：
String.prototype.lastIndexOf = function(searchValue, fromIndex) {
    fromIndex = fromIndex || this.length; // 默认值为 str.length

    // 如果 fromIndex 小于 0
    // 则判断需查找字符串是不是空字符串
    // 如果是则返回 0，如果不是则返回 -1。
    if (fromIndex < 0) {
        return searchValue === "" ? this.length : -1;
    }

    // 如果 fromIndex 大于 str.length 则当做 str.length
    if (fromIndex > str.length) { 
        fromIndex = str.length;
    }

    fromIndex = fromIndex >>> 0; // 取整

}
```-->

---

### String.prototype.includes()

**语法**
> **str.includes(searchString[, position])**

`includes()` 方法用于判断一个字符串是否包含在另一个字符串中，根据情况返回 true 或 false。

`searchString`
要搜索的子字符串。

`position`
从当前字符串的该索引位置开始搜寻子字符串，默认值为0。

**Polyfill**
```JavaScript
if(!String.prototype.includes) {
    String.prototype.includes = function(searchString, position) {
        'use strict';
        if (typeof position !== 'number') {
            position = 0;
        }
        if (position + searchString.length > this.length) {
            return false;
        } else {
            return this.indexOf(searchString, position) !== -1;
        }
    }
}
```

---

### String.prototype.startsWith()

**语法**
> **str.startsWith(searchString [, position]);**

`startsWith()` 方法用来判断当前字符串是否是以另外一个给定的子字符串开头。

`searchString`
要搜索的子字符串

`position`
在 str 中搜索 searchString 的开始位置，默认值为 `0`，也就是真正的字符串开头处。

**Polyfill**

```JavaScript
if (!String.prototype.startsWith) {
  (function() {
    'use strict'; // needed to support `apply`/`call` with `undefined`/`null`
    var defineProperty = (function() {
      // IE 8 only supports `Object.defineProperty` on DOM elements
      try {
        var object = {};
        var $defineProperty = Object.defineProperty;
        var result = $defineProperty(object, object, object) && $defineProperty;
      } catch(error) {}
      return result;
    }());
    var toString = {}.toString;
    var startsWith = function(search) {
      if (this == null) {
        throw TypeError();
      }
      var string = String(this);
      if (search && toString.call(search) == '[object RegExp]') {
        throw TypeError();
      }
      var stringLength = string.length;
      var searchString = String(search);
      var searchLength = searchString.length;
      var position = arguments.length > 1 ? arguments[1] : undefined;
      // `ToInteger`
      var pos = position ? Number(position) : 0;
      if (pos != pos) { // better `isNaN`
        pos = 0;
      }
      var start = Math.min(Math.max(pos, 0), stringLength);
      // Avoid the `indexOf` call if no match is possible
      if (searchLength + start > stringLength) {
        return false;
      }
      var index = -1;
      while (++index < searchLength) {
        if (string.charCodeAt(start + index) != searchString.charCodeAt(index)) {
          return false;
        }
      }
      return true;
    };
    if (defineProperty) {
      defineProperty(String.prototype, 'startsWith', {
        'value': startsWith,
        'configurable': true,
        'writable': true
      });
    } else {
      String.prototype.startsWith = startsWith;
    }
  }());
}
```

---

### String.prototype.endsWith()

**语法**
> **str.endsWith(searchString [, position]);**

`endsWith()` 方法用来判断当前字符串是否是以另外一个给定的子字符串结尾的。

`searchString`
要搜索的子字符串

`position`
在 str 中搜索 searchString 的结束位置，默认值为 `str.length`，也就是真正的字符串结尾处。

**Polyfill**

```JavaScript
if (!String.prototype.endsWith) {
	String.prototype.endsWith = function( searchString, position) {
		if (position === undefined || position > this.length) {
			position = this.length;
		}
		return this.substring(position - searchString.length, position) === searchString;
	};
}
```

---

### String.prototype.slice()

**语法**
> **str.slice(beginSlice[, endSlice])**

slice() 方法提取一个字符串的一部分，并返回一新的字符串。

`beginSlice`
从该索引（以 0 为基数）处开始提取原字符串中的字符。如果值为负数，会被当做 `str.length + beginSlice`，如果是小数则保留整数部分，如果超出 `str.length` 则返回空字符串。

`endSlice`
在该索引（以 0 为基数）处结束提取字符串。如果省略该参数，slice会一直提取到字符串末尾。如果该参数为负数，则被看作是 `str.length + endSlice`，如果是小数则保留整数部分，如果超出 `str.length` 则当做 `str.length`。

*注意：提取的字符包括 `beginSlice` 位置的字符但是不包括 `endSlice` 位置的字符，如果 `endSlice <= beginSlice` 则返回空字符串。*

常用操作：配合 `Object.prototype.toString` 判断一个变量的类型

```JavaScript
function type(v) {
    return Object.prototype.toString.call(v).slice(8, -1);
}
```

<!--```JavaScript
// 这两个参数的内部机制可以用如下伪代码表示：
String.prototype.slice = function(beginSlice, endSlice) {
    let strLen = str.length;

    endSlice = endSlice || strLen;
    // 如果 endSlice <= beginSlice 则返回空字符串
    if (endSlice <= beginSlice) {
        return "";
    }
    // 如果 beginSlice 超出 str.length 则返回空字符串
    if (beginSlice >= strLen) {
        return "";
    }

    // 如果 endSlice 则当做 str.length
    if (endSlice > strLen) {
        endSlice = strLen;
    }

    // 如果参数值为负数则被当作 str.length + beginSlice
    if(beginSlice < 0) {
        beginSlice += strLen
    }
    if(endSlice < 0) {
        endSlice += strLen
    }

    // 取整
    beginSlice = beginSlice >>> 0;
    endSlice = endSlice >>> 0;

}
```-->

---

### String.prototype.split()

**语法**
> **str.split([separator[, limit]])**

`split()` 方法使用指定的分隔符字符串将一个字符串分割成字符串数组。 

`separator`
表示每个拆分的位置。`separator` 可以是一个字符串或正则表达式。如果纯文本分隔符包含多个字符，则必须找到整个字符串来表示分割点。如果省略或在 `str` 中不出现分隔符，则返回的数组包含一个由整个字符串组成的元素。如果分隔符为空字符串，则将 `str` 原字符串中每个字符的数组形式返回。

`limit`
一个整数（如果是小数时，只保留整数部分，如果是负值则相当于无限制），限定返回的字符串数组的长度。当提供此参数时，`split` 方法会在指定分隔符的每次出现时分割该字符串，在数组长度达到限制长度时返回，后面如果还有剩下的字符串也会忽略掉。


*Note: 如果 separator 包含捕获括号（capturing parentheses），则其匹配结果将会包含在返回的数组中。*

```JavaScript
var str = "Hello 1 word.";

str.split(/(\d)/);   // ["Hello ", "1", " word."]
str.split(/\d/);     // ["Hello ", " word."]
str.split("");       // ["H", "e", "l", "l", "o", " ", "1", " ", "w", "o", "r", "d", "."]
str.split("", -1);   // ["H", "e", "l", "l", "o", " ", "1", " ", "w", "o", "r", "d", "."]
str.split("" , 3);   // ["H", "e", "l"]
str.split("" , 3.5); // ["H", "e", "l"]
```

---

### String.prototype.replace()

**语法**
> **str.replace(regexp|substr, newSubStr|function)**

`replace()` 方法返回一个由替换值替换一些或所有匹配的模式后的新字符串。模式可以是一个字符串或者一个正则表达式, 替换值可以是一个字符串或者一个每次匹配都要调用的函数。

*Note: 原字符串不会改变。*

`regexp|substr`
所匹配的内容会被第二个参数的返回值替换掉。

`newSubStr`
用于替换掉第一个参数在原字符串中的匹配部分的字符串。该字符串中可以内插一些特殊的变量名。

`function`
一个用来创建新子字符串的函数，该函数的返回值将替换掉第一个参数匹配到的结果。

#### 使用 `newSubStr` 参数

| 变量名 | 值 |
| --- | --- |
| $$ | 插入一个 `"$"` |
| $& | 插入匹配的子串 |
| $` | 插入当前匹配子串左边的内容 |
| $' | 插入当前匹配子串右边的内容 |
| $n | 第一个参数是 RegExp 对象，并且 n 是个小于100的非负整数，那么插入第 n 个括号匹配的字符串 |

#### 指定一个函数作为参数

你可以指定一个函数作为第二个参数。在这种情况下，当匹配执行后， 该函数就会执行。 函数的返回值作为替换字符串。 (注意:  上面提到的特殊替换参数在这里不能被使用。) 另外要注意的是， 如果第一个参数是正则表达式， 并且其为全局匹配模式， 那么这个方法将被多次调用， 每次匹配都会被调用。

| 变量名 | 值 |
| --- | --- |
| match | 匹配的子串。（对应于上述的$&。） |
| p1,p2, ... | 如果 replace() 方法的第一个参数是一个 RegExp 对象，则代表第n个括号匹配的字符串。 |
| offset | 匹配到的子字符串在原字符串中的偏移量。 |
| string | 被匹配的原字符串。 |

---

### String.prototype.match()

**语法**
> **str.match(regexp)**

`match()` 方法检索匹配项，并以数组的方式返回结果。

`regexp`
一个正则表达式对象。如果传入一个非正则表达式对象，则会隐式地使用 new RegExp(obj) 将其转换为一个 RegExp。如果不传参数，那么会返回一个包含空字符串的 Array ：[""]（具体结果会因浏览器差异而不同，但是都返回数组且第一项为空字符串）。

```JavaScript
var str1 = "NaN means not a number. Infinity contains -Infinity and +Infinity in JavaScript.",
    str2 = "My grandfather is 65 years old and My grandmother is 63 years old.",
    str3 = "The contract was declared null and void.";

str1.match("number");   // "number" 是字符串。返回["number"]
str1.match(NaN);        // NaN的类型是number。返回["NaN"]
str1.match(Infinity);   // Infinity的类型是number。返回["Infinity"]
str1.match(+Infinity);  // 返回["Infinity"]
str1.match(-Infinity);  // 返回["-Infinity"]
str2.match(65);         // 返回["65"]
str2.match(+65);        // 有正号的number。返回["65"]
str3.match(null);       // 返回["null"]
```

---

### String.prototype.search()

**语法**
> **str.search(regexp)**

`search()` 方法检索匹配项，并以返回首次匹配项的索引，如果没有返回 -1。

`regexp`
一个正则表达式对象。如果传入一个非正则表达式对象，则会隐式地使用 new RegExp(obj) 将其转换为一个 RegExp。

---

### String.prototype.substr()

**语法**
> **str.substr(start[, length])**

`substr()` 方法返回一个字符串中从指定位置开始到指定字符数的字符。

`start`
开始提取字符的位置。如果为负值，则被看作 `str.length + start`，如果为小数则保留整数部分。

`length`
提取的字符数，如果为小数则保留整数部分，如果 `start + length > str.length` 则返回截取到字符串末尾，如果不传值默认截取到字符串末尾。

---

### String.prototype.substring()

**语法**
> **str.substring(indexStart[, indexEnd])**

`substring()` 方法返回一个字符串在开始索引到结束索引之间的一个子字符串（包括开始索引位置的字符不包括结束索引位置的字符）。

`indexStart`
开始提取字符的位置。

`indexEnd`
结束提取字符的位置。

这两个参数有如下规则：
* 如果 `indexStart == endStart` 返回空字符串
* 如果省略 `indexEnd`，返回子字符串直到字符串末尾
* 任一参数小于 `0` 或为 `NaN` 则当做 `0`
* 任一参数大于 `str.length` 则当做 `str.length`
* 如果 `indexStart > indexEnd` 则两参数值互换

---

### String.prototype.repeat()

**语法**
> str.repeat(count)

`repeat()` 方法返回一个新字符串，该字符串包含被连接在一起的指定次数的字符串。

`count`
介于0和正无穷大之间的整数 : [0, +∞) 。表示在新构造的字符串中重复了多少遍原字符串。

*Note: `count` 不在指定范围内会报错，且如果构成的字符串超过了能够容纳的最大字符串长度也会报错，即内存溢出。*

**Polyfill**

```JavaScript
if (!String.prototype.repeat) {
  String.prototype.repeat = function(count) {
    'use strict';
    if (this == null) {
      throw new TypeError('can\'t convert ' + this + ' to object');
    }
    var str = '' + this;
    count = +count;
    if (count != count) {
      count = 0;
    }
    if (count < 0) {
      throw new RangeError('repeat count must be non-negative');
    }
    if (count == Infinity) {
      throw new RangeError('repeat count must be less than infinity');
    }
    count = Math.floor(count);
    if (str.length == 0 || count == 0) {
      return '';
    }
    // 确保 count 是一个 31 位的整数。这样我们就可以使用如下优化的算法。
    // 当前（2014年8月），绝大多数浏览器都不能支持 1 << 28 长的字符串，所以：
    if (str.length * count >= 1 << 28) {
      throw new RangeError('repeat count must not overflow maximum string size');
    }
    var rpt = '';
    for (;;) {
      if ((count & 1) == 1) {
        rpt += str;
      }
      count >>>= 1;
      if (count == 0) {
        break;
      }
      str += str;
    }
    return rpt;
  }
}
```

---

### String.prototype.padStart()

**语法**
> str.padStart(targetLength [, padString])

`padStart()` 方法用另一个字符串填充当前字符串(如果需要的话则重复填充)，返回填充后达到指定长度的字符串。从当前字符串的开始（左侧）开始填充。

`targetLength`
当前字符串需要填充到的目标长度。如果这个数值小于当前字符串的长度，则返回当前字符串本身。

`padString`
填充字符串。如果字符串太长，使填充后的字符串长度超过了目标长度，则只保留最左侧的部分，其他部分会被截断。默认值为空格。

**Polyfill**

```JavaScript
if (!String.prototype.padStart) {
    String.prototype.padStart = function padStart(targetLength,padString) {
        targetLength = targetLength >> 0; //floor if number or convert non-number to 0;
        padString = String(padString || ' ');
        if (this.length > targetLength) {
            return String(this);
        } else {
            targetLength = targetLength - this.length;
            if (targetLength > padString.length) {
                //append to original to ensure we are longer than needed
                padString += padString.repeat(targetLength / padString.length);
            }
            return padString.slice(0, targetLength) + String(this);
        }
    };
}
```

---

### String.prototype.padEnd()

**语法**
> str.padEnd(targetLength [, padString])

`padEnd()` 方法会用一个字符串填充当前字符串（如果需要的话则重复填充），返回填充后达到指定长度的字符串。从当前字符串的末尾（右侧）开始填充。

`targetLength`
当前字符串需要填充到的目标长度。如果这个数值小于当前字符串的长度，则返回当前字符串本身。

`padString`
填充字符串。如果字符串太长，使填充后的字符串长度超过了目标长度，则只保留最左侧的部分，其他部分会被截断。默认值为空格。

**Polyfill**

```JavaScript
if (!String.prototype.padEnd) {
    String.prototype.padEnd = function padEnd(targetLength,padString) {
        targetLength = targetLength >> 0; // floor if number or convert non-number to 0;
        padString = String(padString || ' ');
        if (this.length > targetLength) {
            return String(this);
        } else {
            targetLength = targetLength-this.length;
            if (targetLength > padString.length) {
                // append to original to ensure we are longer than needed
                padString += padString.repeat(targetLength / padString.length);
            }
            return String(this) + padString.slice(0, targetLength);
        }
    };
}
```

一图胜千言
![String 思维导图](String.png)

下面是 xmind 文件

[String.xmind](String.xmind)
