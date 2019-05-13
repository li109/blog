# Vue2.0源码阅读笔记（四）：nextTick
&emsp;&emsp;在阅读 nextTick 的源码之前，要先弄明白 JS 执行环境运行机制，介绍 JS 执行环境的**事件循环**机制的文章很多，大部分都阐述的比较笼统，甚至有些文章说的是错误的，以下为个人理解，如有错误，欢迎指正。<br/>
## 一、浏览器中的进程与线程
&emsp;&emsp;以 chorme 浏览器为例，浏览器中的每个页面都是一个独立的进程，在该进程中拥有多个线程，通常有以下几个常驻线程：<br/>
> 1、GUI 渲染线程<br/>
> 2、JavaScript引擎线程<br/>
> 3、定时触发器线程<br/>
> 4、事件触发线程<br/>
> 5、异步http请求线程<br/>

&emsp;&emsp;**GUI 渲染线程**解析 html 生成 DOM 树，解析 css 生成 CSSOM 树，然后将两棵树合并成渲染树，最后根据渲染树画出界面。当 DOM 的修改导致了样式非几何属性的变化时，渲染线程重新绘制新的样式，称为“重绘”；当 DOM 的修改导致了样式几何属性的变化，渲染线程会重新计算元素的集合属性，然后将结果绘制出来，称为“回流”。<br/>
&emsp;&emsp;**JS 引擎线程**负责处理Javascript脚本程序，且与GUI 渲染线程是互斥的，因为 js 是可以操控 DOM 的，如果这两个线程并行会导致错误。JS 引擎线程与其他可以并行的线程配合来实现称为**Event Loop**的 javaScript 执行环境运行机制。<br/>
&emsp;&emsp;JS 的运行环境是单线程的，在代码中如果调用形如 setTimeout() 这样的计时功能的 API ，JS 引擎线程会将该任务交给**定时触发器线程**。定时触发器线程在定时结束之后会将任务放入任务队列中，等待 JS 引擎线程读取。<br/>
&emsp;&emsp;JS 与 HTML 之间的交互是通过**事件**来实现的。在 JS 代码中使用**侦听器**来预定事件，以便事件发生时执行相应的代码，该代码称为**事件处理程序**或者**事件侦听器**。例如点击事件的事件侦听器是 onclick 。**JS 引擎线程**在执行侦听 DOM 元素的代码时，会将该任务交给**事件触发线程**处理，当事件被触发时，**事件触发线程**会将任务放入任务队列中，等待 JS 引擎线程读取。<br/>
&emsp;&emsp;JS 代码中通过 XMLHttpRequest 发起 ajax 请求时，会使用**异步http请求线程**来管理，在状态改变时，该线程会将对应的回调放入任务队列中，等待 JS 引擎线程读取。<br/>
## 二、Event Loop
&emsp;&emsp;Javascript 任务分为**同步任务**和**异步任务**，同步任务是指调用之后立刻得到结果的任务；异步任务是指调用之后无法立刻得到结果，需要进行额外操作的任务。<br/>
&emsp;&emsp;JS 引擎线程顺序执行**执行栈**中的任务，**执行栈中只有同步任务**，遇到异步任务就交给相应的线程处理。例如在代码块中有 setTimeout() 方法的调用，则将其交由**定时触发器线程**处理，定时结束之后**定时触发器线程**将方法的回调放入自身的任务队列中，当执行栈中的任务处理完之后会读取各线程中任务队列中的事件。<br/>
&emsp;&emsp;前面是从同步异步的角度来划分任务的，从执行顺序来说，任务也分为两种：macrotask（宏任务）、microtask（微任务）。异步的 macrotask 执行完之后返回的事件会放在各线程的任务队列中，microtask 执行完之后返回的事件会放在微任务队列中。<br/>
> macrotask包括：script（JS文件）、MessageChannel、setTimeout、setInterval、setImmediate、I/O、ajax、eventListener、UI rendering。<br/>
> microtask包括：Promise、MutationObserver、已废弃的Object.observe()、Node中的process.nextTick<br/>

&emsp;&emsp;其中需要注意的是**GUI 渲染线程**去渲染页面也是以 macrotask 的形式进行的，这个之后详谈。<br/>
![Event Loop](../image/vue/nextTick_1.jpg)
&emsp;&emsp;JS 执行环境运行机制——Event Loop（事件循环）的过程如上图所示：<br/>
1、**JS 引擎线程**顺序执行**执行栈**中的任务，以一个 macrotask 为单位，在单个宏任务没有处理完之前，**JS 引擎线程**不会将程序交由**GUI 渲染线程**接管。也就是说耗时的任务会阻塞渲染，导致页面卡顿的情况发生。典型浏览器一般1秒钟插入60个渲染帧，也就是说16ms进行一次渲染，单个任务超过16ms，如果渲染树发生改变将得不到及时更新渲染。<br/>
&emsp;&emsp;流畅的页面中一般任务执行情况如下所示：<br/>
![](../image/vue/nextTick_2.jpg)
&emsp;&emsp;单个任务耗时较多，则会发生丢帧的情况：<br/>
![](../image/vue/nextTick_3.jpg)
2、**JS 引擎线程**在执行 macrotask 时，会将遇到的异步任务交给指定的线程处理。当异步任务为 macrotask 时，对应线程处理完毕之后**放入线程自身的任务队列中**；若异步任务为 microtask 时，对应线程处理完毕之后**放入微任务队列中**。macrotask 执行完之后会遍历微任务队列中的任务加以执行，清空微任务队列。<br/>
3、当**执行栈**中的任务执行完毕后，会读取各个线程中的任务队列，将各任务队列中的事件添加到**执行栈**中开始执行。从读取各任务队列中的事件放入**执行栈**中到清空微任务队列的过程称为一个“tick”。JS引擎线程会循环不断地读取任务、处理任务，这个就称为**Event Loop**（事件循环）机制。<br/>
## 三、nextTick的实现
&emsp;&emsp;Vue的数据更新采用的是异步更新的方式，这样的好处是数据属性多次求值只不用重复调用渲染函数，能够大幅提高性能。其中，异步更新队列是通过调用 nextTick 方法完成的。<br/>
&emsp;&emsp;Vue是数据驱动的框架，最好的情况是在页面重新渲染前完成数据的更新。从前面的讲述中可以知道，浏览器的运行机制是首先执行 macrotask，然后执行 microtask ，清空微任务队列后，再从各线程的任务队列中读取新的事件之前，GUI 渲染线程有可能接管程序，完成页面重新渲染。<br/>
&emsp;&emsp;nextTick() 在2.5版本之后被单独提取到一个 js 文件中，并且改变了其实现方式。下面分别介绍两种具体实现情况：<br/>
### 1、Vue2.5+ 版本实现方式
&emsp;&emsp;Vue2.5.22 版本的 nextTick() 实现如下所示：<br/>
```js
export function nextTick (cb?: Function, ctx?: Object) {
  let _resolve
  callbacks.push(() => {
    if (cb) {
      try {
        cb.call(ctx)
      } catch (e) {
        handleError(e, ctx, 'nextTick')
      }
    } else if (_resolve) {
      _resolve(ctx)
    }
  })
  if (!pending) {
    pending = true
    if (useMacroTask) {
      macroTimerFunc()
    } else {
      microTimerFunc()
    }
  }
  if (!cb && typeof Promise !== 'undefined') {
    return new Promise(resolve => {
      _resolve = resolve
    })
  }
}
```
&emsp;&emsp;首先说明其中三个变量，callbacks 是存储异步更新回调的任务队列、pending 标识任务队列是否正在刷新、useMacroTask 变量表明是否强制使用 macrotask 方式执行回调。<br/>
&emsp;&emsp;nextTick() 注册一个执行传入回调的函数放入到 callbacks 数组中，如果没有传入回调则返回 Promise 对象。如果队列没有开始刷新，则将等待刷新标识设为 true，开始刷新任务。如果没有强制指明需要使用 macrotask 的方式刷新，则默认调用 microTimerFunc 方法来执行。<br/>
&emsp;&emsp;microTimerFunc 方法的实现如下代码所示：<br/>
```js
if (typeof setImmediate !== 'undefined' && isNative(setImmediate)) {
  macroTimerFunc = () => { setImmediate(flushCallbacks) }
} else if (typeof MessageChannel !== 'undefined' && (
  isNative(MessageChannel) ||
  MessageChannel.toString() === '[object MessageChannelConstructor]'
)) {
  const channel = new MessageChannel()
  const port = channel.port2
  channel.port1.onmessage = flushCallbacks
  macroTimerFunc = () => { port.postMessage(1) }
} else {
  macroTimerFunc = () => { setTimeout(flushCallbacks, 0) }
}

if (typeof Promise !== 'undefined' && isNative(Promise)) {
  const p = Promise.resolve()
  microTimerFunc = () => {
    p.then(flushCallbacks)
    if (isIOS) setTimeout(noop)
  }
} else {
  microTimerFunc = macroTimerFunc
}
```
&emsp;&emsp;microTimerFunc 方法实质就是将 flushCallbacks 方法注册成异步任务加以执行。<br/>
&emsp;&emsp;优先使用 Promise 的方式将 flushCallbacks() 的执行注册成 microtask；其中需要注意的是在有的ios环境下，即使将任务推到微任务队列中，队列也不会马上刷新，直到浏览器需要做一些其它的工作，因此在此处添加一个空的计时器来使微任务队列刷新。<br/>
&emsp;&emsp;如果环境不兼容 Promise，则将 flushCallbacks() 的执行注册成 macrotask。优先使用 setImmediate 注册任务，setImmediate() 性能好、优先级高，但是兼容性很差，目前只有 IE 浏览器支持。其次使用 MessageChannel 实现，如果都不支持，则调用 setTimeout() 实现。<br/>
&emsp;&emsp;flushCallbacks() 的实现方式如下所示：<br/>
```js
function flushCallbacks () {
  pending = false
  const copies = callbacks.slice(0)
  callbacks.length = 0
  for (let i = 0; i < copies.length; i++) {
    copies[i]()
  }
}
```
&emsp;&emsp;首先将是否刷新的标识设为 false ，然后复制 callbacks 数组到 copies ，再清空 callbacks 数组，遍历 copies 执行每一个回调。这里将 callbacks 清空、遍历复制数组 copies 的原因是为了防止在遍历执行回调的过程中，不断有新的回调添加到 callbacks 数组中的情况发生。<br/>
### 2、老版本实现方式
&emsp;&emsp;Vue2.4.4 版本的 nextTick() 实现与2.5+ 版本的差异主要是下面这段代码：<br/>
```js
  if (typeof Promise !== 'undefined' && isNative(Promise)) {
    var p = Promise.resolve()
    var logError = err => { console.error(err) }
    timerFunc = () => {
      p.then(nextTickHandler).catch(logError)
      if (isIOS) setTimeout(noop)
    }
  } else if (!isIE && typeof MutationObserver !== 'undefined' && (
    isNative(MutationObserver) ||
    MutationObserver.toString() === '[object MutationObserverConstructor]'
  )) {
    var counter = 1
    var observer = new MutationObserver(nextTickHandler)
    var textNode = document.createTextNode(String(counter))
    observer.observe(textNode, { characterData: true })
    timerFunc = () => {
      counter = (counter + 1) % 2
      textNode.data = String(counter)
    }
  } else {
    timerFunc = () => {setTimeout(nextTickHandler, 0)}
  }
```
&emsp;&emsp;老版本的 nextTick() 与2.5+ 版本的最主要区别是将任务注册成异步队列的方式不同。优先使用 Promise 将任务注册成 microtask，其次使用 MutationObserver 将任务注册成 microtask。如果环境不允许将任务注册成 microtask，则直接使用 setTimeout() 将任务注册成 macrotask。<br/>
&emsp;&emsp;可以看出老版本的 nextTick() 对性能的追求特别高，基本上都是采用 microtask 来实现异步更新的，macrotask 没有区分层级，直接使用 setTimeout() 来最后兜底。<br/>
&emsp;&emsp;MutationObserver 的优先级特别高，在某些场景下它甚至要比事件冒泡还要快，会导致很多问题。如果全部使用 macrotask 则对一些有重绘和动画的场景也会有性能影响。所以 Vue2.5+ 版本删除了对 MutationObserver 的使用，增强了 macrotask 的使用。<br/>
