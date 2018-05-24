---
title: React 1
date: 2018-05-22 14:01:33
tags: React
categories: React
---
# React

React是Facrbook内部的一个JavaScript类库，已于1年开源，可用于创建Web用户交互界面。它引入了一种新的方式来处理浏览器DOM。那些需要手动更新DOM、费力地记录每一个状态的日子一去不复返了——这种老舅的方式既不具备扩展性，又很难加入新的功能，就算可以，也是有着冒着很大的风险。React使用很新颖的方式解决了这些问题。你只需要声明地定义各个时间点的用户界面，而无序关系在数据变化时，需要更新哪一部分DOM。在任何时间点，React都能以最小的DOM修改来更新整个应用程序。

React引入了一些激动人心的新概念，向现有的一些最佳实践发起了挑战。学习这些概念，将帮助你理解它们的优势，创建具备高扩展性的单页面应用（SPA）。React把主要的注意力放在了应用的“视图”部分，没有限定与服务端交互和代码组织的方式。

## Why React?

为什么要使用 React ? 老夫就是 jQuery 一梭子去撸不可以么？

😢还真不方便。产品经理说，这里需要给我改个东西。吭哧吭哧用 jQuery 写了半天，然后产品经理说，客户又改需求了！这种事情当然在实际开发中经常遇到，怎么办？难道真的像下面这样？

![PM & RD](pm&rd.png)

当然是用我们的技术去应对这种情况的发生。让开发变得更简单更高效，后期维护方便。上午改需求下午出 Release 这才是积极的应对方式🌞。

那 `React` 有哪些优势呢？

#### 1. 大名鼎鼎的 Virtual DOM

对浏览器中的 DOM 操作开销是特别大的。正常情况下一个状态改变，我们只需要对网页中需要改变的 DOM 做一些操作。

然而平常开发过程中（我）几乎不考虑对 DOM 操作有什么大的开销。而 React 会帮助我们做这些事情，用 React 内部自己的 diff 算法帮助我们最小化的操作 DOM。

#### 2. 组件化

组件化开发的优点想来不用多说。

高复用，UI 与 业务低耦合。 `React` 通过将每个功能相对独立的模块定义成组件，通过组件的组合嵌套的方式构成大的组件，完成整体构筑。

就像是砌房子一样。窗户是窗户，门是门，墙是墙，三者独立也是一个完整的东西，三者组合起来构成房屋。（家徒四壁）

`React` 的组件化使得开发变得更加方便，可维护，使得代码复用率更高。一切都是 component。

#### 3. 数据绑定

`React` 的数据绑定是单向的。数据和视图进行了绑定，当数据变化时，视图自动更新，不用再操作，提升开发效率。

**好吧其实也没多少。甚至说只有一点就是组件化**

## Hello JSX
**JSX** 语法，像是在 `JavaScript` 代码里直接写 *XML* 的语法，实质上这只是一个语法糖，每一个 *XML* 标签都会被 **JSX** 转换工具转换成纯 `JavaScript` 代码，React 官方推荐使用 **JSX** ， 当然你想直接使用纯 `JavaScript` 代码写也是可以的，只是使用 **JSX** ，组件的结构和组件之间的关系看上去更加清晰。

```JavaScript
//使用JSX
React.render(
    <div>
        <div>
            <div>content</div>
        </div>
    </div>,
    document.getElementById('example')
);

//不使用JSX
React.render(
    React.createElement('div', null,
        React.createElement('div', null,
            React.createElement('div', null, 'content')
        )
    ),
    document.getElementById('example')
);
```

其实我们在 **JSX** 里写一个 *XML* 标签，就是在调用 `React.createElement` 这个方法，并返回一个 `ReactElement` 对象。

### JSX 中嵌入表达式

可以用花括号把任意的 `JavaScript 表达式` 嵌入到 **JSX** 。

```JavaScript
function formatName(user) {
    return user.firstName + ' ' + user.lastName;
}

const user = {
    firstName: 'Harper',
    lastName: 'Perez'
};

const ele = (
    <h1> Hello, {formatName(user)}!</h1>
);

ReactDOM.render(
    ele,
    document.getElementById('root')
);
```

### JSX 也是一个表达式

编译之后，**JSX** 表达式就变成了常规的 `JavaScript 对象`

这意味着可以在 `if` 语句或是 `for` 循环中使用 **JSX** ，用它给变量赋值，当做参数接收，或则作为函数返回值。

```JavaScript
function getGreeting(user) {
    if (user) {
        return <h1>Hello, {formatName(user)}!</h1>
    }
    return <h1>Hello, Stranger.</h1>
}
```

### 用 JSX 指定属性值

可以使用双引号来指定字符串字面量作为属性值：

```Javascript
const element = <div tableIndex="0"></div>
```

也可以使用花括号嵌入一个 JavaScript 表达式作为属性值：

```JavaScript
const element = <img src={user.avatarUrl}></img>;
```

## 新开一栏

在写文章的时候还是会有一些地方不理解。比如

1. 为什么大家都说操作 DOM 很慢？你说慢就慢呀？慢在哪里？
2. Em... `React` 的啥 diff 算法？简单点怎么实现？如何通过 diff 算法去优化我们的组件？

后续会开新坑把这些问题一一弄清楚。然后和这里链接起来。
