---
title: 'Wow BFC'
date: 2018-07-01 15:24:55
tags: CSS
categories: [CSS, BFC]
---
# 哇哦， BFC

经常用到和听到 BFC， 但是对于具体的 BFC 有哪些情况可以触发了解的还不是很全面，今天整理下对于 BFC 的理解。

## What's BFC

BFC (Block Formatting Context) 即块级格式化上下文。相似的还有 IFC (Inline), FFC (Flex), GFC (Grid)。

### BFC 布局规则

1. 内部的盒子会在垂直方向一个接一个的放置。
2. 盒子的垂直方向距离由 `margin` 决定。属于 `同一个 BFC` 的两个相邻盒子的 `margin` 会重叠，值由两个 `margin` 的最大值决定。
3. 每个元素的 `margin` 的左边，与包含块的 `border` 的左边相接触（由左向右的布局，否则相反）
4. BFC 的区域不会与 float 盒子重叠。 
5. BFC 是页面上的隔离的独立容器，容器内的子元素不会影响到外面的元素。
6. 计算 BFC 的高度时，浮动元素也参与计算

## 哪些元素会生成 BFC

* 根元素
* float 属性不为 none
* position 属性不为relative和static。
* display 属性为 inline-block, table-cell, table-caption, flex, inline-flex 中的任何一个
* overflow 不为 visible

## BFC 的作用和原理

### 1. 自适应两栏布局

```HTML
<style>
    .demo1 {
        position: relative;
        width: 300px;
        border: 1px solid #000;
    }
 
    .demo1 .aside {
        float: left;
        width: 100px;
        height: 150px;
        background: #f66;
    }
 
    .demo1 .main {
        height: 200px;
        overflow: hidden;
        background: #fcc;
    }
</style>
<div class="demo1">
    <div class="aside"></div>
    <div class="main"></div>
</div>
```

页面如下：

<div><style>.demo1 {width: 300px; position: relative;border: 1px solid #000;}.demo1 .aside {width: 100px; height: 150px; float: left; background: #f66;}.demo1 .main {height: 200px;background: #fcc;overflow: hidden;}</style><div class="demo1"><div class="aside"></div><div class="main"></div></div></div>

这里应用了 BFC 布局规则的第三条和第四条。

### 2. 清除内部浮动


```HTML
<style>
    .father {
        width: 300px;
        overflow: hidden;
        border: 5px solid #fcc;
    }
 
    .child {
        float: left;
        width:100px;
        height: 100px;
        border: 5px solid #f66;
    }
</style>

<div class="father">
    <div class="child"></div>
    <div class="child"></div>
</div>
```

页面如下：

<div><style>.father {overflow: hidden;border: 5px solid #fcc;width: 300px;}.child {border: 5px solid #f66;width:100px;height: 100px;float: left;}</style><div class="father"><div class="child"></div><div class="child"></div></div></div>

这里应用了 BFC 布局规则的第六条。

### 3. 防止垂直 margin 重叠

```HTML
<style>
    .demo3{
        border: 1px solid #000;
    }
    .demo3 .wrap {
        overflow: hidden;
    }
    .demo3 p {
        margin: 100px;
        line-height: 100px;
        text-align:center;
        color: #f55;
        background: #fcc;
    }
</style>
<div class="demo3">
    <p>Haha</p>
    <div class="wrap">
        <p>Hehe</p>
    </div>
</div>
```

页面如下:

<div><style>.demo3{border: 1px solid #000;}.wrap {overflow: hidden;}#post-content .demo3 p {margin: 100px;line-height: 100px;text-align:center;color: #f55;background: #fcc;}</style><div class="demo3"><p>Haha</p><div class="wrap"><p>Hehe</p></div></div>

这里应用了 BFC 布局规则的第二条。

---
## What's IFC

IFC 的 line box（线框）高度由其包含行内元素中最高的实际高度计算而来（不受到竖直方向的padding/margin影响)
IFC 中的 line box 一般左右都贴紧整个 IFC，但是会因为 float 元素而扰乱。float 元素会位于 IFC 与与 line box 之间，使得 line box 宽度缩短。 同个 IFC 下的多个 line box 高度会不同。 IFC 中时不可能有块级元素的，当插入块级元素时（如p中插入div）会产生两个匿名块与 div 分隔开，即产生两个 IFC，每个 IFC 对外表现为块级元素，与 div 垂直排列。

那么IFC一般有什么用呢？

水平居中：当一个块要在环境中水平居中时，设置其为 inline-block 则会在外层产生 IFC，通过 text-align 则可以使其水平居中。
垂直居中：创建一个 IFC，用其中一个元素撑开父元素的高度，然后设置其 vertical-align:middle，其他行内元素则可以在此父元素下垂直居中。

## What's GFC

GFC 当为一个元素设置 `display: grid` 的时候，此元素将会获得一个独立的渲染区域，我们可以通过在网格容器（grid container）上定义网格定义行（grid definition rows）和网格定义列（grid definition columns）属性各在网格项目（grid item）上定义网格行（grid row）和网格列（grid columns）为每一个网格项目（grid item）定义位置和空间。 

那么 GFC 有什么用呢，和 table 又有什么区别呢？

首先同样是一个二维的表格，但 GridLayout 会有更加丰富的属性来控制行列，控制对齐以及更为精细的渲染语义和控制。

## What's FFC

Flex Box 由伸缩容器和伸缩项目组成。通过设置元素的 `display: flex` 或 `inline-flex` 可以得到一个伸缩容器。

设置为 flex 的容器被渲染为一个块级元素，而设置为 inline-flex 的容器则渲染为一个行内元素。

伸缩容器中的每一个子元素都是一个伸缩项目。伸缩项目可以是任意数量的。伸缩容器外和伸缩项目内的一切元素都不受影响。简单地说，Flexbox 定义了伸缩容器内伸缩项目该如何布局。

---

参考文章：[前端精选文摘：BFC 神奇背后的原理](http://www.cnblogs.com/lhb25/p/inside-block-formatting-ontext.html)