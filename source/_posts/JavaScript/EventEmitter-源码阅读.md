---
title: EventEmitter 源码阅读
date: 2018-08-27 13:14:40
tags: [JavaScript, EventEmitter]
categories: JavaScript
---

上周看了 [EventEmitter](https://github.com/Olical/EventEmitter) 的源码，这篇博客打算做个小结（不是 Node 的源码，只是 github 上搜到的星星最多的项目，还有个项目 [EventEmitter3](https://github.com/primus/eventemitter3) 项目作者目前依然在维护，主打性能，看起来非常不错）

## 整体结构

```JavaScript
;(function(exports) {
    'use strict';

    function EventEmitter() {}

    var proto = EventEmitter.prototype;
    var originalGlobalValue = exports.EventEmitter;

    /**
     * 工具函数
     * indexOfListener
     * alias
     * isValidListener
     */

    /**
     * 方法实现
     */

    EventEmitter.noConflict = function() {
        exports.EventEmitter = originalGlobalValue;
        return EventEmitter;
    }

    if (typeof define === 'function' && define.amd) {
        define(function() {
            return EventEmitter;
        });
    } else if (typeof module === 'object' && module.exports){
        module.exports = EventEmitter;
    } else {
        exports.EventEmitter = EventEmitter;
    }

}(typeof window !== 'undefined' ? window : this || {}));
```

### 最外层的自执行

```JavaScript
;(function(exports){
  // some code  
}(typeof window !== 'undefined' ? window : this || {}));
```
**0. 闭包**

避免全区变量污染，私有作用域... 就是闭包的常规用途

**1. 开头的分号**

为了防止打包或则其他工具合并 js 代码并压缩的时候由于前面没有以分号结尾而出错，比如这种情况 `var a = someFunction`

如果不加分号则会变成 `var a = someFunction( function(){}() )`，这可能导致意外。

**2. context 以 export 为变量名传入函数**

可以在代码压缩时减小 size，将全局的 window 传入可以避免内部使用 window 进行查询时由作用域链向上多找一级。

**3. 类的实现**

利用函数的原型链模拟，先创建一个空函数 `function EventEmitter() {}`，然后将该类的方法挂载到 `EventEmitter.prototype` 上面。

**4. 对于冲突的处理**

```JavaScript
    var originalGlobalValue = exports.EventEmitter;

    EventEmitter.noConflict = function() {
        exports.EventEmitter = originalGlobalValue;
        return EventEmitter;
    }
```

先将全局的 `EventEmitter` 保存到一个变量 `originalGlobalValue` 中。执行 `noConflict` 后再将 `originalGlobalValue` 还给全局的 `EventEmitter`，并返回这个库所实现的 `EventEmitter`。

例如：
```JavaScript
var EventEmitter = function a() {}

// load myEventEmitter.js
// 此时 EventEmitter = myEventEmitter;
var b = EventEmitter.noConflict();

// 此时 EventEmitter = function a() {}
// b = myEventEmitter
```

**5. 对于模块化的处理**

```JavaScript
// 兼容 AMD
if (typeof define === 'function' && define.amd) {
        define(function() {
            return EventEmitter;
        });
    }
    // 兼容 CommonJS
    else if (typeof module === 'object' && module.exports){
        module.exports = EventEmitter;
    } else {
        exports.EventEmitter = EventEmitter;
    }
```

这个库的整体结构并不复杂，基本上看过 jQuery 源码或则对一些常用的模块化方法有基本了解都能理解

## 工具函数的实现

### alias

```JavaScript
/**
 * Alias a method while keeping the context correct, to allow for overwriting of target method.
 *
 * @param {String} name The name of the target method.
 * @return {Function} The aliased method
 * @api private
 */
function alias(name) {
    return function aliasClosure() {
        return this[name].apply(this, arguments);
    };
}
```

这么调用 `EventEmitter.prototype.on = alise('addListener')`。
可是为什么要多此一举？ 而不直接用 `EventEmitter.prototype.on = EventEmitter.prototype.addListener`

### 其他
isValidListener(listener) 就是判断 listener 是不是合法

indexOfListener(listeners: Function[], listener: Function) 就是在数组中找到对应项的 listener 属性和第二个参数 Function 全等的项并返回索引值

## 核心方法的实现

### _getEvents

```JavaScript
/**
 * Fetches the events object and creates one if required.
 *
 * @return {Object} The events storage object.
 * @api private
 */
proto._getEvents = function _getEvents() {
    return this._events || (this._events = {});
};
```

取 `_event` 属性如果不存在就创建一个空对象赋值给 `_event` 并返回。

### getListeners

```JavaScript
/**
 * Returns the listener array for the specified event.
 * Will initialise the event object and listener arrays if required.
 * Will return an object if you use a regex search. The object contains keys for each matched event. So /ba[rz]/ might return an object containing bar and baz. But only if you have either defined them with defineEvent or added some listeners to them.
 * Each property in the object response is an array of listener functions.
 *
 * @param {String|RegExp} evt Name of the event to return the listeners from.
 * @return {Function[]|Object} All listener functions for the event.
 */
proto.getListeners = function getListeners(evt) {
    var events = this._getEvents();
    var response;
    var key;

    // Return a concatenated array of all matching events if
    // the selector is a regular expression.
    // 支持正则查询， 如果没查询到返回空对象 {}
    if (evt instanceof RegExp) {
        response = {};
        for (key in events) {
            if (events.hasOwnProperty(key) && evt.test(key)) {
                response[key] = events[key];
            }
        }
    }
    else {
        response = events[evt] || (events[evt] = []); // 如果不存在就初始化 _this.event[evt] 为 []
    }

    return response;
};
```

### addListener

```JavaScript
/**
 * Adds a listener function to the specified event.
 * The listener will not be added if it is a duplicate.
 * If the listener returns true then it will be removed after it is called.
 * If you pass a regular expression as the event name then the listener will be added to all events that match it.
 *
 * @param {String|RegExp} evt Name of the event to attach the listener to.
 * @param {Function} listener Method to be called when the event is emitted. If the function returns true then it will be removed after calling.
 * @return {Object} Current instance of EventEmitter for chaining.
 *
 * 1. 先判断要添加的 listener 是否合法
 * 2. 获取当前符合条件的 listener
 * 3. 判断要添加的 listener 是否已经存在
 * 4. 如果不存在 就将 listener 包裹成固定的格式起来放入 listeners 中
 */
proto.addListener = function addListener(evt, listener) {
    if (!isValidListener(listener)) {
        throw new TypeError('listener must be a function');
    }

    var listeners = this.getListenersAsObject(evt);
    var listenerIsWrapped = typeof listener === 'object';
    var key;

    for (key in listeners) {
        if (listeners.hasOwnProperty(key) && indexOfListener(listeners[key], listener) === -1) {
            listeners[key].push(listenerIsWrapped ? listener : {
                listener: listener,
                once: false
            });
        }
    }

    return this;
};
```


### removeListener

```JavaScript
/**
 * Removes a listener function from the specified event.
 * When passed a regular expression as the event name, it will remove the listener from all events that match it.
 *
 * @param {String|RegExp} evt Name of the event to remove the listener from.
 * @param {Function} listener Method to remove from the event.
 * @return {Object} Current instance of EventEmitter for chaining.
 */
// 删除事件订阅
// 1. 找到所有的符合 evt 规则的事件
// 2. 循环找出事件在 events 中的索引并 splice 删除
proto.removeListener = function removeListener(evt, listener) {
    var listeners = this.getListenersAsObject(evt);
    var index;
    var key;

    for (key in listeners) {
        if (listeners.hasOwnProperty(key)) {
            index = indexOfListener(listeners[key], listener);

            if (index !== -1) {
                listeners[key].splice(index, 1);
            }
        }
    }

    return this;
};
```

### removeEvent

```JavaScript
/**
 * Removes all listeners from a specified event.
 * If you do not specify an event then all listeners will be removed.
 * That means every event will be emptied.
 * You can also pass a regex to remove all events that match it.
 *
 * @param {String|RegExp} [evt] Optional name of the event to remove all listeners for. Will remove from every event if not passed.
 * @return {Object} Current instance of EventEmitter for chaining.
 */
/**
 * 删除事件
 * 1. 判断 evt 类型
 * 如果是 字符串 就直接删除 _events 中对应的属性
 * 如果是 正则表达式 就删除符合正则的 _events 中对应的属性
 * 如果都不是 就删除全部事件
 */
proto.removeEvent = function removeEvent(evt) {
    var type = typeof evt;
    var events = this._getEvents();
    var key;

    // Remove different things depending on the state of evt
    if (type === 'string') {
        // Remove all listeners for the specified event
        delete events[evt];
    }
    else if (evt instanceof RegExp) {
        // Remove all events matching the regex.
        for (key in events) {
            if (events.hasOwnProperty(key) && evt.test(key)) {
                delete events[key];
            }
        }
    }
    else {
        // Remove all listeners in all events
        delete this._events;
    }

    return this;
};
```

### emitEvent

```JavaScript
/**
 * Emits an event of your choice.
 * When emitted, every listener attached to that event will be executed.
 * If you pass the optional argument array then those arguments will be passed to every listener upon execution.
 * Because it uses `apply`, your array of arguments will be passed as if you wrote them out separately.
 * So they will not arrive within the array on the other side, they will be separate.
 * You can also pass a regular expression to emit to all events that match it.
 *
 * @param {String|RegExp} evt Name of the event to emit and execute listeners for.
 * @param {Array} [args] Optional array of arguments to be passed to each listener.
 * @return {Object} Current instance of EventEmitter for chaining.
 */
/**
 * 触发事件
 * 1. 找到 符合条件的事件
 * 2. 依次调用
 * 3. 判断是否是一次调用，如果是就删除事件
 */
proto.emitEvent = function emitEvent(evt, args) {
    var listenersMap = this.getListenersAsObject(evt);
    var listeners;
    var listener;
    var i;
    var key;
    var response;

    for (key in listenersMap) {
        if (listenersMap.hasOwnProperty(key)) {
            listeners = listenersMap[key].slice(0);

            for (i = 0; i < listeners.length; i++) {
                // If the listener returns true then it shall be removed from the event
                // The function is executed either with a basic call or an apply if there is an args array
                listener = listeners[i];

                if (listener.once === true) {
                    this.removeListener(evt, listener.listener);
                }

                response = listener.listener.apply(this, args || []);

                if (response === this._getOnceReturnValue()) {
                    this.removeListener(evt, listener.listener);
                }
            }
        }
    }

    return this;
};
```

### 全部方法

* _getEvents
* _getOnceReturnValue
* getListeners
* flattenListeners
* getListenersAsObject
* addListener || on
* addOnceListener || once
* defineEvent
* defineEvents
* removeListener || off
* addListeners
* removeListeners
* manipulateListeners
* removeEvent || removeAllListeners
* emitEvent || trigger
* emit
* setOnceReturnValue

**之后可以自造一个实现该功能的轮子**