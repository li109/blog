# Vue2.0源码阅读笔记（四）：nextTick
&emsp;&emsp;Vue.nextTick 与 vm.nextTick 是两个比较常用的 API，作用是在下次 DOM 更新循环结束之后执行延迟回调。在修改数据之后立即使用该方法，获取更新后的 DOM。<br/>
&emsp;&emsp;这两个 API 在源码中的实现是调用同一个方法 nextTick()，Vue2.5 版本将该方法单独写在一个 js 文件中，并更改了其实现方式。在具体了解 nextTick() 实现之前有必要阐述一下 javaScript 执行环境的运行机制。<br/>
## 一、javaScript 执行环境运行机制
### 1、浏览器环境下的js
&emsp;&emsp;以浏览器为例，浏览器中的每个页面都是一个独立的进程，在该进程中拥有多个线程，比如：GUI 渲染线程、JS 引擎线程、定时触发器线程等。<br/>
&emsp;&emsp;GUI 渲染线程解析 html 生成 DOM 树，解析 css 生成 CSSOM 树，然后将两棵树合并成渲染树，最后根据渲染树画出界面。当 DOM 的修改导致了样式非几何属性的变化时，渲染线程重新绘制新的样式，称为“重绘”；当 DOM 的修改导致了样式几何属性的变化，渲染线程会重新计算元素的集合属性，然后将结果绘制出来，称为“回流”。<br/>
&emsp;&emsp;JS 引擎线程负责处理Javascript脚本程序，且与GUI 渲染线程是互斥的，因为 js 是可以操控 DOM 的，如果这两个线程并行会导致错误。JS 引擎线程与其他可以并行的线程配合来实现称为**Event Loop**的 javaScript 执行环境运行机制。<br/>
### 2、Event Loop
&emsp;&emsp;Javascript任务分为**同步任务**和**异步任务**，同步任务是指调用之后立刻得到结果的任务；异步任务是指调用之后无法立刻得到结果，需要进行额外操作的任务。<br/>
1、JS 引擎线程会顺序执行**执行栈**中任务，当执行栈中的有异步任务时，会将该任务交给对应的线程处理。比如：通过 ajax 发送网络请求，会通过**异步http请求线程**来完成；设置一个定时器，会使用**定时触发器线程**来完成。当这些线程处理完任务之后，会在各自的任务队列中放置对应事件。<br/>
2、当JS 引擎线程**执行栈**中的任务执行完毕，开始检查渲染，然后 GUI 渲染线程接管渲染。<br/>
3、渲染完毕后，JS 引擎线程继续接管，读取各任务队列的任务，添加到执行栈中开始执行。<br/>
&emsp;&emsp;主线程按照1、2、3的顺序不断重复执行，一个这样的循环过程称为一个 tick。这就是 js 执行环境运行机制：事件循环（Event Loop）。<br/>
## 二、macrotask 与 microtask
&emsp;&emsp;<br/>
&emsp;&emsp;<br/>
&emsp;&emsp;<br/>
## 三、使用 MutationObserver 实现的版本
&emsp;&emsp;<br/>
&emsp;&emsp;<br/>
&emsp;&emsp;<br/>
## 四、使用 MessageChannel 实现的版本
&emsp;&emsp;<br/>
&emsp;&emsp;<br/>
&emsp;&emsp;<br/>
## 五、异步更新队列
&emsp;&emsp;<br/>
&emsp;&emsp;<br/>
&emsp;&emsp;<br/>
## 六、总结
&emsp;&emsp;<br/>
&emsp;&emsp;<br/>
&emsp;&emsp;<br/>
