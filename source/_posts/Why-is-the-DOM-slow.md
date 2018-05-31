---
title: Why is the DOM slow?
date: 2018-05-29 16:21:27
tags: Q&A
categories: Q&A
---

# 为什么大家都说 DOM 操作很慢

之前在学习 *React* 的时候看到 *React* 的优势的时候就说， *React* 的 diff 算法，会“最小化”处理 DOM。因为操作 DOM 的开销太大了。频繁操作 DOM 会有性能问题。

那么今天就来了解下。为什么操作 DOM 会很慢。

## 什么是 DOM (Document Object Model)

DOM 是一个独立于语言的，使用 *XML* 和 HTML 文档操作的接口。在浏览器中主要与 HTML 文档打交道。 DOM APIs 主要用于访问这些文档中的数据。

DOM 是个 API 浏览器通常要求 DOM 实现和 JavaScript 实现保持相互独立。例如 Chrome 中 使用 WebKit 的 WebCore 库渲染页面，但是具有一个分离的 JavaScript 引擎 V8

> Inherently Slow
> 这对性能意味着，两个独立的部分以功能接口连接就会带来性能损耗。一个恒形象的比喻是把 DOM 看成一个岛屿，把 JavaScript 看成另一个岛屿，两者之间以一座收费桥连接。每次 JavaScript 需要访问 DOM 时，需要过桥，交一次“过桥费”。 DOM 操作越多，费用越高。一般建议是尽量减少过桥次数。------ 《高性能 JavaScript》

*Note: 目前大多数的浏览器对此做的优化已经足够多，访问 DOM 的最主要的开销也不再次，而是每次 DOM 操作的“后遗症”------浏览器的重排（reflow） 和 重绘（repaint）*


## DOM 访问和修改

访问一个 DOM 元素的代价就是交一次“过桥费”。修改元素的代价可能更多，因为它经常导致浏览器重新计算页面结构变化。

访问或修改元素最差的情况是使用循环去做，特别是是 HTML 集合中使用。比较下面两个例子。

```JavaScript
function innerHTMLLoop() {
    for(var count = 0; count < 1000; count++) {
        /* 在循环中访问 DOM 和修改 DOM。操作了 1000 次
           每次先获得 DOM innerHTML 中的内容，然后去更新它。*/
        document.getElementById().innerHTML += "a";
    }
}
```

```JavaScript
function innerHTMLLoop() {
    var content = "";
    for(var count = 0; count < 1000; count++) {
        content += "a";  // 循环保存变量，然后一次性写入。
    }
    // 只访问和修改了一次 DOM
    document.getElementById().innerHTML += content;
}
```

在所有浏览器中，第二张方法的运行速度都比一次快得多。

## HTML 集合

HTML 集合是用于存放 DOM 节点引用的类数组对象。下列函数返回值就是：

* document.getElementsByName()
* document.getElementsByClassName()
* document.getElementsByTagName()

下列属性也是

* document.images
* doucment.links
* document.forms
* document.forms[0].elements
* document.anchors

HTML 集合实际上是在查询文档，当更新信息时，每次都会重复执行查询操作。

来看看下面的代码有什么问题吧。

```JavaScript
var divs = document.getElementsByTagName('div');
for (var i = 0; i < divs.length; i++) {
    document.body.appendChild(document.createElement('div'));
}
```

循环中的判断条件是 `divs` 的长度，但是每次循环的时候会自动添加一个 `div元素` 到 body 中，所以每次查询 `length` 都会改变，导致死循环。

而遍历数组明显会快于同样大小的 HTML 集合。所以需要多次用到同一个 HTML 集合的时候可以考虑将集合中的元素拷贝到一个数组中，然后去遍历那个数组。


## 重绘和重排

首先要了解浏览器如何展示一个网页的。

### 浏览器的构成

浏览器的主要构成如下：

<p class="center">![Layers](layers.png)</p>

<p class="center">浏览器主要组件</p>

包含了：

* **The user interface**: 包含了地址栏，前进后退按钮...等等除了请求页面窗口之外的地方。
* **The browser engine**: 请求和操作渲染引擎的接口
* **The rendering engine**: 负责展现请求页面。例如请求内容是个 HTML 它负责解析 HTML 和 CSS 并且显示
* **Networking**: 完成网络调用。例如 HTTP 请求，它是平台无关的接口可以在不同平台下使用。
* **UI backend**: 绘制想多选框和窗口等基本组件，具有不特定于某个平台的通用接口，底层调用用户接口
* **JavaScript interpreter**: 解释执行 JS 代码
* **Data storage**: 属于持久层，浏览器需要在硬盘中保存像 cookies 的各种数据。

#### Rendering engine 所做的工作

渲染引擎搜先通过网络获得所请求文档的内容，通常以 8K 分块的方式完成。

下面是渲染引擎在获取内容后的基本流程：

<p class="center">![mainflow](mainflow.png)</p>

<p class="center">解析 HTML 构建 DOM 树 → 构建渲染树 → 布局渲染树 → 绘制渲染树</p>

<p class="center">渲染引擎基本流程</p>

获取内容后，浏览器开始解析 HTML 并将标签转化为 "Content tree(DOM tree)" 中的 DOM 节点。接着，他解析外部 CSS 文件以及 style 标签中的样式信息，这些样式信息和 HTML 中的可见性指令将用来构建 Render tree

Render tree 为每个需要显示的 DOM 树节点存放至少一个节点（隐藏 DOM 元素在渲染树中没有对应节点）。渲染树上的节点称为“框”或“盒”，符合 CSS 模型的定义，将元素看做一个具有填充、边距、边框和位置的盒。Render tree 由一些包含颜色和大小等属性的矩形组成，它们将按正确的顺序显示到屏幕上。


Render tree 构建好了之后，将会执行布局过程，它将确定每个节点在屏幕上的确切坐标。下一步就是绘制，遍历 Render tree 并使用 UI backend 绘制每个节点。

这个过程是逐步完成的，为了更好地用户体验，渲染引擎将会尽可能早的将内容呈现在屏幕上，不会等所有的 HTML 都解析完成之后再去构建和布局 Render tree。它是解析完一部分内容后就显示一部分内容，同时可能还在通过网络下载其余内容。

<p class="center">![Webkit flow](webkitflow.png)</p>

<p class="center">Webkit 渲染引擎主流程</p>

<p class="center">![Gecko flow](geckoflow.jpg)</p>

<p class="center">Geoko 渲染引擎主流程</p>

影响页面展示的因素有许多，比如 link 位置会影响首屏显示.这里只讨论 layout 相关内容。

paint 是一个耗时的过程，但是 layout 是一个更耗时的过程。

当 DOM 改变影响到元素的几何属性（宽和高）———— 例如改变了边框宽度或是在段落中添加文字，将发生一系列后续动作 ———— 浏览器需要重新计算元素的几何属性，而且其他元素的几何属性和位置也会因此改变受到影响。浏览器使渲染树上受到的影响的部分失效，然后重构渲染树。这个过程被称作重排（reflow）。重排完成时，浏览器会在一个重绘进程中重新绘制屏幕上受影响的部分。

不是所有的 DOM 改变都会影响几何属性。比如，改变一个元素的背景颜色不会影响它的宽度和高度。在这种情况下，只需要重绘，因为元素的布局没有改变。

重绘和重排是负担很重的操作，可能导致网页应用的用户界面是去响应。所以十分有必要尽可能减少此类事情的发生。

#### 浏览器何时重排

想要最小化重排次数，要先了解什么情况下浏览器会进行重排。

* **Page renders initially**: 页面初始渲染
* **Visible DOM elements are added or removed**: 可见元素的添加或删除
* **Elements change position**: 元素位置改变
* **Elements change size (because of a change in margin, padding, border thickness, width, height, etc.)**: 由于 margin, padding, border-width, width, height 等属性导致的元素大小改变
* **Content is changed (text changes or an image is replaced with on of a different size)**: 文本改变或图片被另一个不同尺寸的图片替换时导致的内容改变
* **Browser window is resized**: 浏览器窗口大小改变

根据改变的性质，渲染树上或大或小的一部分需要重新计算，某些改变可能导致重排整个页面：例如滚动条出现时。

#### 查询并刷新导致渲染树改变

因为计算量和每次重排有关，大多数浏览器通过队列化化修改和批量显示优化重排过程。然而，我们可能不经意之间强迫队列刷新并要求所有改变的部分立刻应用。当我们查询一些布局信息的时候会导致浏览器更新渲染树。

如下方法：

* offsetTop, offsetLeft, offsetWidth, offsetHeight
* scrollTop, scrollLeft, scrollWidth, scrollHeight
* clientTop, clientLeft, clientWidth, clientHieght
* getComputedStyle() ( currentStyle() in IE)

等等这些布局信息由这些属性和方法返回最新的数据，所以浏览器不得不运行渲染队列中待改变的项目并重新排版以返回正确的值。

*拓展： [会导致重排和重绘的 CSS 属性](https://csstriggers.com/)*

#### 如何优化？

下面的例子导致了三次重排

```JavaScript
// Read
var h1 = element1.clientHeight;

// Write (invalidates layout)
element1.style.height = (h1 * 2) + 'px';

// Read (triggers layout)
var h2 = element2.clientHeight;

// Write (invalidates layout)
element2.style.height = (h2 * 2) + 'px';

// Read (triggers layout)
var h3 = element3.clientHeight;

// Write (invalidates layout)
element3.style.height = (h3 * 2) + 'px';  
```

**批处理优化**

```JavaScript
// Read
var h1 = element1.clientHeight;  
var h2 = element2.clientHeight;  
var h3 = element3.clientHeight;

// Write (invalidates layout)
element1.style.height = (h1 * 2) + 'px';  
element2.style.height = (h2 * 2) + 'px';  
element3.style.height = (h3 * 2) + 'px'; 
```

**另外当需要对 DOM 元素进行多次修改时，可以通过以下步骤减少重绘和重排的次数**

1. 从文档中摘除该元素
2. 应用修改
3. 将元素带回文档流中

有三种基本方法可以将 DOM 从文档中摘除

* 隐藏元素，进行修改然后再显示它
* 使用一个文档片段在已存 DOM 之外创建一个子树，然后将它拷贝到文档中（document.createDocumentFragment()）
* 将原始元素拷贝到一个脱离文档的节点中，修改副本，然后覆盖原始元素

**将元素提出动画流**

显示和隐藏部分页面构成展开/折叠动画是一种常见的交互模式。它通常包括区域扩大的几何动画，将页面其他部分推向下方。

重排版有时只影响渲染树的一小部分，但也可以影响很大的一部分，甚至整个渲染树。浏览器需要重排
版的部分越小，应用程序的响应速度就越快。所以当一个页面顶部的动画推移了差不多整个页面时，将引
发巨大的重排版动作，使用户感到动画卡顿。渲染树的大多数节点需要被重新计算，它变得更糟糕。

**IE and :hover IE 和:hover**

自从版本 IE7 之后，可以在任何元素（严格模式）上使用 :hover 这个 CSS 伪选择器。然而，如果大量的元素使用了 :hover 那么会降低反应速度。此问题在 IE8 中更显著。

*拓展：[How browsers work](http://taligarsiel.com/Projects/howbrowserswork1.htm)*












