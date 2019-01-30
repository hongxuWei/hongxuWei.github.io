---
title: Lodash.debounce 源码阅读
date: 2019-01-25 09:14:40
tags: [JavaScript, Lodash, debounce]
categories: JavaScript
thumbnail: /img/lodash.jpeg
---
# Lodash 源码阅读（一）

## 前言小絮叨

距离上次写博客已经过了大概 4 个月了，自从到了贝壳业务越来越多，一到周末就累得什么也不想干。最近刚忙完 H5 的项目（😝后续可以来一篇企微浏览器使用蓝牙的小总结感觉很有意思）又接近年关 OP 封板停止发布。虽然需求也在排着，但是想写写自己的东西。

## 常见用法

`debounce` 就是函数防抖，当函数一直被触发时等到 waiting 时间到了之后再执行，常见用法就是防止用户多次点击或者当一个函数会频繁触发而又没有必要调用那么多次时使用该函数（或函数节流）以优化性能。

## debounce

```JavaScript
// lodash debounce
function debounce(func, wait, options) {
  let lastArgs,
    lastThis,
    maxWait,
    result,
    timerId,
    lastCallTime

  let lastInvokeTime = 0 // 上一次执行时间
  let leading = false    // 开始是否执行
  let maxing = false     // 是否有最大等待时间
  let trailing = true    // 结束是否执行

  // Bypass `requestAnimationFrame` by explicitly setting `wait=0`.
  // 是否使用 requestAnimationFrame
  // 当忽略 wait 时会使用 requestAnimationFrame 下一帧时就调用函数。（16ms）
  const useRAF = (!wait && wait !== 0 && typeof root.requestAnimationFrame === 'function')
  // 如果 func 不是函数直接抛出错误
  if (typeof func != 'function') {
    throw new TypeError('Expected a function')
  }
  // 获取配置项
  // +arg 类似一个简单的 toNumber(arg)
  wait = +wait || 0
  if (isObject(options)) {
    leading = !!options.leading
    maxing = 'maxWait' in options
    maxWait = maxing ? Math.max(+options.maxWait || 0, wait) : maxWait
    trailing = 'trailing' in options ? !!options.trailing : trailing
  }

  // 调用 func
  function invokeFunc(time) {          // time 调用时的时间戳
    const args = lastArgs              // 调用的参数
    const thisArg = lastThis           // 调用的 this

    lastArgs = lastThis = undefined    // 清除 lastArgs 和 lastThis
    lastInvokeTime = time              // 记录调用时间
    result = func.apply(thisArg, args) // 调用 func 并将结果返回
    return result
  }
  // 开始计时
  function startTimer(pendingFunc, wait) {
    if (useRAF) {
      return root.requestAnimationFrame(pendingFunc)
    }
    return setTimeout(pendingFunc, wait)
  }
  // 取消计时
  function cancelTimer(id) {
    if (useRAF) {
      return root.cancelAnimationFrame(id)
    }
    clearTimeout(id)
  }
  // 延时前调用
  function leadingEdge(time) {
    // Reset any `maxWait` timer.
    lastInvokeTime = time
    // Start the timer for the trailing edge.
    timerId = startTimer(timerExpired, wait)
    // Invoke the leading edge.
    return leading ? invokeFunc(time) : result
  }
  // 设置需要等待时间
  function remainingWait(time) {
    const timeSinceLastCall = time - lastCallTime     // 距上次触发的时间
    const timeSinceLastInvoke = time - lastInvokeTime // 距上次调用 func 的时间
    const timeWaiting = wait - timeSinceLastCall      // 还需等待的时间

    return maxing
      ? Math.min(timeWaiting, maxWait - timeSinceLastInvoke)
      : timeWaiting
  }
  // 判断是否应该被调用
  function shouldInvoke(time) {
    const timeSinceLastCall = time - lastCallTime
    const timeSinceLastInvoke = time - lastInvokeTime

    // Either this is the first call, activity has stopped and we're at the
    // trailing edge, the system time has gone backwards and we're treating
    // it as the trailing edge, or we've hit the `maxWait` limit.
    return (lastCallTime === undefined || (timeSinceLastCall >= wait) ||
      (timeSinceLastCall < 0) || (maxing && timeSinceLastInvoke >= maxWait))
  }
  // 刷新 timer
  function timerExpired() {
    const time = Date.now()
    if (shouldInvoke(time)) {
      return trailingEdge(time)
    }
    // Restart the timer.
    timerId = startTimer(timerExpired, remainingWait(time))
  }
  // 延时后调用
  function trailingEdge(time) {
    timerId = undefined

    // Only invoke if we have `lastArgs` which means `func` has been
    // debounced at least once.
    // 配置中 trailing 为 true 且有 lastArgs（func 被执行过一次）
    if (trailing && lastArgs) {
      return invokeFunc(time)
    }
    lastArgs = lastThis = undefined
    return result
  }
  // 取消执行
  function cancel() {
    if (timerId !== undefined) {
      cancelTimer(timerId)
    }
    lastInvokeTime = 0
    lastArgs = lastCallTime = lastThis = timerId = undefined
  }
  // 直接执行
  function flush() {
    return timerId === undefined ? result : trailingEdge(Date.now())
  }
  // 等待
  function pending() {
    return timerId !== undefined
  }

  function debounced(...args) {
    const time = Date.now()
    const isInvoking = shouldInvoke(time)        // 判断是否可调用

    lastArgs = args                              // 获取参数
    lastThis = this                              // 获取调用时的 this
    lastCallTime = time                          // 触发时间

    if (isInvoking) {
      if (timerId === undefined) {               // 首次触发调用 leadingEdge
        return leadingEdge(lastCallTime)
      }
      if (maxing) {
        // Handle invocations in a tight loop.
        // 处理频繁的多次调用
        timerId = startTimer(timerExpired, wait) // 设置定时器
        return invokeFunc(lastCallTime)
      }
    }
    if (timerId === undefined) {                 // 如果没有 timer 设置定时器
      timerId = startTimer(timerExpired, wait)
    }
    return result
  }
  debounced.cancel = cancel
  debounced.flush = flush
  debounced.pending = pending
  return debounced
}
```
