---
title: React 1
date: 2018-05-22 14:01:33
tags: React
categories: React
thumbnail: /img/React.png
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

#### ...

**好吧其实也没多少。甚至说只有一点就是组件化，解决业务中的痛点。**

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

***JSX 使用时应该注意自闭合，class 关键字用 className 替代，单容器包裹等。***


## Components & Props

组件化将 UI 分为一个个独立可复用的小部件。

### 函数式组件和类组件

定义组件的方式有两种，一个是写一个 JavaScript 函数或是用 ES6 来定义一个组件。

下面这两种组件在 `React` 看来是等价的。（注意下面组件的命名，首字母大写）

```JavaScript
function Welcome(props) {
    return <h1>Hello, {props.name}</h1>;
}
```

```JavaScript
class Welcome extends React.Component {
    constructor(props) {
        super(props);
    }
    render() {
        return <h1>Hello, {this.props.name}</h1>;
    }
}
```
### 渲染一个组件

React 元素可以是

```JavaScript
const element = <div />;
```

也可以是用户自定义的组件

```JavaScript
const element = <Welcome name="Sara" />; //注意自闭合
```

其中 JSX 的属性以一个单独对象 `props` 传递给对应的组件。

***Note: `props` 是只读的，React 组件都必须是纯函数，并且禁止修改自身 props。***


## 状态和生命周期

有下面这样一个组件每秒会更新时间

```JavaScript
class Clock extends React.Component {
    constructor(props) {
        super(props);
        this.state = {
            date: new Date(),
            name: "Sara"
        };
    }

    componentDidMount() {
        this.timerID = setInterval(
            () => this.tick(),
            1000
        );
    }

    componentWillUnmount() {
        clearInterval(this.timerID);
    }

    tick() {
        this.setState({date: new Date()})
    }

    render() {
        return (
            <div>
                <h1>Hello, {this.state.name}.</h1>
                <h2>Time is {this.state.date.toLocalTimeString()}.</h2>
            </div>
        );
    }
}
```

### state

关于 state 有下面几点要注意

#### 1. 不要直接修改 state 而是调用 setState 去更新

```JavaScript
this.state.name = "Bob"; // 错误，这样视图不会更新
this.setState({name: "Bob"}); // OK
```

#### 2. state 更新可能是异步的

React 为了优化性能会将多个 `setState()` 调用合并。

`this.props` 和 `this.state` 可能是异步更新，所以不能依赖他们的值去计算下一个 **state**

```JavaScript
this.setState({
    counter: this.state.counter + this.props.increment // 可能会导致更新失败
});

/* 要解决这个问题，可以使用另一种 setState 的形式，它接受一个函数而不是一个对象。
这个函数将接收前一个状态作为第一个参数，应用更新时的 prop 作为第二个参数 */

this.setState((prevState, props) => ({
    couter: prevSatete.counter + props.increment //OK
}))
```

### 生命周期 lifecycle

嗯哼，一图胜千言。

![生命周期](component-lifecycle.jpg)

## React 的数据流和组件间的通信

React 是单向数据流，数据主要从父节点传递到子节点 （*props*）

如果父级的某个 *props* 改变了，React 会重新渲染所有的子节点。

**props 和 state**

尽可能使用 `props` 当做数据源， `state` 用来存放状态值

即通常用 `props` 传递大量数据， `state` 用于存放组件内部一些简单的定义数据。

### 那么组件间是怎么通信的呢？

组件间通信方式也就那么几种

* props
* props 回调
* context 对象
* 事件订阅

#### 1. 父子组件通信

* **父组件更新状态 —— props ——> 子组件更新**
* **子组件更新  —— props 回调 ——> 调用回调函数更新**

#### 2. 兄弟组件通信

* **兄弟组件更新  —— props 回调 ——> 调用回调函数更新**
* **context方式更新**
* **事件订阅**

## 处理事件

React 处理事件与在 DOM 元素上处理事件的区别

* React 事件使用驼峰命名，而不是全小写。
* 通过 JSX 传递一个函数作为事件处理程序，而不是字符串。
* 阻止默认行为必须明确调用 *preventDeafult* 而不是 *return false*。


```JavaScript
class Toggle extends React.Component {
    constructor(props) {
    super(props);
        this.state = {isToggleOn: true};

        // 这个绑定是必要的，使`this`在回调中起作用
        this.handleClick = this.handleClick.bind(this);
    }

    handleClick() {
        this.setState(prevState => ({
            isToggleOn: !prevState.isToggleOn
        }));
    }

    render() {
        return (
            <button onClick={this.handleClick}>
                {this.state.isToggleOn ? 'ON' : 'OFF'}
            </button>
        );
    }
}

ReactDOM.render(
    <Toggle />,
    document.getElementById('root')
);
```

**在 JSX 回调中必须注意 *this* 的指向问题(line 7)**

## 条件渲染

[讲真没什么可写的，丢个链接吧。](https://reactjs.org/docs/conditional-rendering.html)

## Lists & Key

JavaScript 中转换列表

```JavaScript
const number = [1, 2, 3, 4, 5];
const double = number.map((number) => number * 2);
```
在 React 中 转换数组为元素列表的方式和上述方法基本相同

### 多组件渲染

```JavaScript
const number = [1, 2, 3, 4, 5];
const listItems = number.map((number) => <li>{number}</li>);

ReactDOM.render(
    <ul>{listItems}</ul>,
    document.getElementById('root')
)
```

### 基本列表组件

通常情况下，我们会在一个组件中渲染列表，重构前面的例子到一个组件，它接受一个 numbers 数组，并输出一个元素的无序列表。

```JavaScript
function NumberList(props) {
  const numbers = props.numbers;
  const listItems = numbers.map((number) =>
    <li key={number.toString()}>
      {number}
    </li>
  );
  return (
    <ul>{listItems}</ul>
  );
}

const number = [1, 2, 3, 4, 5];
ReactDOM.render(
    <NumberList numbers={numbers} />,
    document.getElementById('root')
)
```

**注意第4行加上了 *key* 如果不加的话会有一个 Wraning: a key should be provided for list items**

### Keys

键(Keys) 帮助 React 标识哪个项被修改、添加或者移除了。数组中的每一个元素都应该有一个唯一不变的键(Keys)来标志

挑选 key 最好的方式是使用一个在它的同辈元素中不重复的标识字符串。多数情况你可以使用数据中的 id 作为 keys

当要渲染的列表项中没有稳定的 id 时，你可以使用数据项的索引值作为 key 的最后选择

而 keys 只在数组的上下文中存在意义

```JavaScript
function ListItem(props) {
    const value = props.value;
    return (
        // 错误！不需要在这里指定 key：
        <li key={value.toString()}>
        {value}
        </li>
    );
}

function NumberList(props) {
    const numbers = props.numbers;
    const listItems = numbers.map((number) =>
    // 错误！key 应该在这里指定：
        <ListItem value={number} />
    );
    return (
        <ul>
        {listItems}
        </ul>
    );
}

const numbers = [1, 2, 3, 4, 5];
ReactDOM.render(
    <NumberList numbers={numbers} />,
    document.getElementById('root')
);
```

**键是React的一个内部映射，但其不会作为 *props* 传递给组件的内部。如果你需要在组件中使用相同的值，可以明确使用一个不同名字的 prop 传入。如：**

```JavaScript
const content = posts.map((post) =>
    <Post
        key={post.id}
        id={post.id}
        title={post.title} />
);
```

## 新开一栏 Q&A

在写文章的时候还是会有一些地方不理解。比如

1. 为什么大家都说操作 DOM 很慢？你说慢就慢呀？慢在哪里？
2. Em... `React` 的啥 diff 算法？简单点怎么实现？如何通过 diff 算法去优化我们的组件？

后续会开新坑把这些问题一一弄清楚。然后和这里链接起来。
