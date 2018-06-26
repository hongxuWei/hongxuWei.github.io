---
title: LeetCode1
date: 2018-06-19 11:08:42
tags: [算法, LeetCode]
categories: 算法
thumbnail: /img/LeetCode1.png
---
# LeetCode 算法题（第一弹）

之前小伙伴面试遇过这个[算法题](https://hongxuwei.github.io/2018/06/12/%E5%85%A8%E6%8E%92%E5%88%97%E9%97%AE%E9%A2%98/)。 让我想到了一句话

> 前端程序员，首先也要是一个程序员。

一直以来都在看一些工作当中要用到的知识，反而忽视了程序员应该具备的一些基本素质。由于是非科班出身，基础反而薄弱，之前有看编译原理，但是虐的我好难啊，等我有所理解再来开新坑。

我自己面试的时候也会有遇到算法题，但是一般想出的都是常规的暴力解法，优化的解法一时也想不出，编程就像是做数学应用题一样，要多练才有思路，所以打算每天做一点点算法题。

## #1两数之和

给定一个整数数组和一个目标值，找出数组中和为目标值的两个数。

你可以假设每个输入只对应一种答案，且同样的元素不能被重复利用。

**示例：**
> 给定 nums = [2, 7, 11, 15], target = 9
> 因为 nums[0] + nums[1] = 2 + 7 = 9
> 所以返回 [0, 1]


### 解法一：暴力法

暴力法很简单。遍历每个元素 `x`，查找是否存在一个值与 `target - x` 相等的目标元素。

*Note: 但是这里有一个坑就是当输入是 `nums = [3, 4, 5, 4], target = 8` 的时候要注意 `x` 和 `target - x` 的值都是 `4`，输出要过滤掉第二次查找到的是 `x` 本身。*


```JavaScript
/**
 * @param {number[]} nums
 * @param {number} target
 * @return {number[]}
 */
var twoSum = function(nums, target) {
    const len = nums.length;
    for (let i = 0 ; i < len; i++) {
        let index = nums.indexOf(target - nums[i]);
        if(index > -1 && index !== i) {
            return [i, index];
        }
    }
};
```


### 解法二： 哈希表法

从哈希表中查找到一个元素的查找时间可以近似为 `O(1)`<sup>[[1]](#qa1)</sup>。哈希表它支持以近似恒定的时间进行快速查找。用“近似”来描述，是因为一旦出现冲突，查找用时可能会退化到 `O(n)`。但只要仔细地挑选哈希函数，在哈希表中进行查找的用时应当被摊销为 `O(1)`。所以可以以空间换取时间的方式优化运行时间。

用哈希表保存数组中的目标元素和索引，如果存在目标元素，找到他的索引就可以了。当然说的这么玄乎，哈希表其实就是JavaScript中对象的存储方式。所以JS的实现就比较简单啦。

```JavaScript
/**
 * @param {number[]} nums
 * @param {number} target
 * @return {number[]}
 */
var twoSum = function(nums, target) {
    let hash = {};
    let num, pair;
    const len = nums.length;

    for(let i = 0; i < len; i++) {
        num = nums[i];
        pair = target - num;
        if ( hash.hasOwnProperty(pair) ) {
            return [hash[pair], i];
        }
        hash[num] = i;
    }
};
```


## #2 两数相加

给定两个非空链表来表示两个非负整数。位数按照逆序方式存储，它们的每个节点只存储单个数字。将两数相加返回一个新的链表。

你可以假设除了数字 0 之外，这两个数字都不会以零开头。

**示例：**
> **输入：**(2 -> 4 -> 3) + (5 -> 6 -> 4)
> **输出：**7 -> 0 -> 8
> **原因：**342 + 465 = 807

### 解题思路

同时遍历两条链表，相同权重的每一位相加，超过9就添加进位，遍历下一个节点时将进位加入下一位。

这题的解法我目前也就只想到这点。要注意因为节点长度不同可能会导致遍历时需要判断是否某条链表已经结束，如果结束那这条链表的这一位需要置0。

```JavaScript
/**
 * Definition for singly-linked list.
 * function ListNode(val) {
 *     this.val = val;
 *     this.next = null;
 * }
 */
/**
 * @param {ListNode} l1
 * @param {ListNode} l2
 * @return {ListNode}
 */
var addTwoNumbers = function(l1, l2) {
    let carry = 0;
    let result;
    let list;

    while (l1 || l2) {
        const val1 = l1 ? l1.val : 0;
        const val2 = l2 ? l2.val : 0;

        l1 = l1 && l1.next;
        l2 = l2 && l2.next;

        const val = val1 + val2 + carry;
        carry = val > 9 ? 1 : 0;

        const node = new ListNode(val % 10);

        if (list) {
            list.next = node;
        }

        list = node;

        result = result || list;
    }

    if (carry) {
        list.next = {
            val: 1,
            next: null,
        };
    }

    return result;
};
```
## #3 无重复字符的最长子串
给定一个字符串，找出不含有重复字符的最长子串的长度。

**示例：**

> 给定 `"abcabcbb"` ，没有重复字符的最长子串是 `"abc"` ，那么长度就是3。
> 给定 `"bbbbb"` ，最长的子串就是 `"b"` ，长度是1。
> 给定 `"pwwkew"` ，最长子串是 `"wke"` ，长度是3。请注意答案必须是一个子串，`"pwke"` 是 子序列  而不是子串。

### 解法一：暴力解法

我的思路是，从某一位 `start` 开始遍历这个字符串，之后再遍历 `start` 之后的字符串，找到第一次重复的位置，此次循环结束。

也就是找到这个字符串的所有不重复子字符串。

比如 `aabca`， 循环依次找到 `a` `abc` `bca` `ca` `a`。

这里我第一个循环并不是到字符串的最后，而是到总字符串长度减去目前已知的最大字符串长度。这样可以减少一些多余的查询。（但是依然很慢哈哈哈）

循环 `aabca` 只会执行两次 当找到 `abc` 后， `ret = 3`，后续的查找是无意义的，所以停止并返回值。

```JavaScript
/**
 * @param {string} s
 * @return {number}
 */
var lengthOfLongestSubstring = function(s) {
    let cache = [];
    const len = s.length;
    var ret = 0;
    for(let start = 0; start < len - ret; start++) {
        for(let j = start; j < len; j++) {
            if(cache.indexOf(s[j]) > -1) {
                break;
            }
            cache.push(s[j]);
        }
        ret = ret > cache.length ? ret : cache.length;
        cache = [];
    }
    return ret;
};
```

### 解法二：滑动窗口法

这种解法就比较有意思了。

思路就是：定义一个窗口，设定窗口的起始位置。向后查找字符，如果遇到重复值窗口位置挪动到重复值第一次出现的位置的后面，这样窗口中的字符串就又是不重复的了，记录窗口的最大宽度，遍历完所有字符后返回最大窗口宽度。

<p class="center">![SlideWindow](SlidingWindow.PNG)</p><p class="center">滑动窗口法</p>

```JavaScript
/**
 * @param {string} s
 * @return {number}
 */
var lengthOfLongestSubstring = function(s) {
    if (!s.length) return 0;
    const len = s.length;
    let max = flag = 0, hash = {};
    for(let i = 0; i < len; i++) {
        if(hash[s[i]] !== undefined) {
            flag = Math.max(hash[s[i]], flag);
        }
        hash[s[i]] = i + 1;
        max = Math.max(i - flag + 1, max);
    }
    return max;
};
//--------------------------------------------------------
/**
 * @param {string} s
 * @return {number}
 */
var lengthOfLongestSubstring = function(s) {
    if (!s.length) return 0;
    const len = s.length;
    let max = 1, flag = 0;
    for(let i = 0; i < len; i++) {
        
        let index = s.indexOf(s[i], flag) 
        if (index !== -1 && index < i) flag = index + 1;
        
        max = Math.max(max, i - flag + 1)
    }
    return max
};
```



## Q&A
<p id="qa1">1. 为什么说从哈希表中查找到一个元素的查找时间可以近似为 `O(1)`</p>

## 原题地址

[#1 两数之和](https://leetcode-cn.com/problems/two-sum/description/)
[#2 两数相加](https://leetcode-cn.com/problems/add-two-numbers/description/)
[#3 无重复字符的最长子串](https://leetcode-cn.com/problems/longest-substring-without-repeating-characters/description/)
