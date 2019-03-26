# 网络学习笔记（三）：HTTP缓存
&emsp;&emsp;HTTP缓存是一种保存资源副本并在下次请求时直接使用该副本的技术，合理的使用缓存可以有效的提升web性能。<br/>
&emsp;&emsp;浏览器将js文件、css文件、图片等资源缓存，当下次请求这些资源时，可以不发送网络请求直接从缓存中取出，称为**缓存命中**；或者发送网络请求验证缓存而不是重新接收该资源，称为**再验证命中**。这两种情况都能够减少冗余数据传输、降低对服务器的要求、提高web性能。在缓存中没有找到副本，直接发送请求给服务器称为**缓存未命中**。<br/>
&emsp;&emsp;缓存空间有限，不可能将全部资源全部缓存，因此选择请求频率高的资源进行缓存很有必要。另外，当服务器上的资源发生可以忽略的改变时，希望能够继续使用缓存而不是重新请求资源；当资源发生需要客户端知晓的改变时，能够准确的更新缓存内容。这些都需要合理的使用缓存机制。<br/>
## 一、与缓存有关的首部字段
&emsp;&emsp;HTTP报文由**起始行**、**首部**和**实体**组成，缓存机制是由首部中的字段控制，下面分别介绍与缓存有关的首部字段。<br/>
### 1、Pragma与Expires
&emsp;&emsp;Pragma 与 Expires 字段是HTTP/1.0规定的首部，用来向后兼容只支持 HTTP/1.0 协议的缓存服务器。<br/>
&emsp;&emsp;Pragma 是通用首部，只有一个值 no-cache ，与 HTTP/1.1 的 Cache-Control: no-cache 效果一致，强制要求缓存在返回缓存的版本之前将请求提交到源头服务器进行验证。<br/>
&emsp;&emsp;Expires 是响应首部，其值是一个GMT（格林尼治时间），表示资源在该时刻之后过期。这个值是相对服务器上的日期而言的，如果浏览器和服务器上的日期不通，则通过这种方式缓存会产生预期之外的情况。<br/>
&emsp;&emsp;需要注意的是： Pragma 的优先级很高。当 Pragma 与 Expires 同时存在时，不管资源有没有过期，都会发起验证请求。当 Pragma 与 Cache-Control 同时存在时，也会发起验证请求。<br/>
### 2、Last-Modified、If-Modified-Since与If-Unmodified-Since
&emsp;&emsp;Last-Modified 是响应首部，其值是一个GMT，表示服务器认定的资源做出修改的时间。包含有 If-Modified-Since 或 If-Unmodified-Since 首部的条件请求会使用该字段。Last-Modified 的时间精确到秒，因此无法识别一秒内进行多次修改的情况。<br/>
&emsp;&emsp;If-Modified-Since 是条件式请求首部，其值是上次响应中 Last-Modified 的值。如果服务器在该时间之后修改了资源，则会返回该资源，状态码为200 ；若在该时间之后没有修改资源，则会返回不带主体的响应，状态码是 304 。该请求首部只能用于 GET 或 HEAD 请求中。当与 If-None-Match 同时出现时，只要服务器支持 If-None-Match ，If-Modified-Since 的值就会被忽略。<br/>
&emsp;&emsp;If-Unmodified-Since 是条件式请求首部，如果所请求的资源在指定的时间之后发生了修改，那么会返回 412 错误；当资源在指定的时间之后没有进行过修改的情况下，服务器会返回请求的资源。<br/>
### 3、ETag、If-None-Match与If-Match
&emsp;&emsp;ETag 是响应首部，其值是资源的特定版本的标识符，可以在标识符之前添加弱验证器标识 W/ ，HTTP 协议默认使用强验证类型。示例如下：<br/>
> ETag: "33a64df551425fcc55e4d42a148795d9f25f89d4"
> ETag: W/"0815"

&emsp;&emsp;强验证类型应用于需要逐个字节相对应的情况，很难有效地生成。弱验证类型应用于用户代理只需要确认资源内容相同即可。即便是有细微差别也可以接受，比如显示的广告不同，或者是页脚的时间不同。弱验证器很容易生成，但不利于比较。<br/>
&emsp;&emsp;If-None-Match 是条件式请求首部，<br/>
&emsp;&emsp;If-Match 是条件式请求首部，<br/>
> If-Match: "bfc13a64729c4290ef5b2c2730249c88ca92d82d"
> If-Match: W/"67ab43", "54ed21", "7892dd"
> If-Match: *

&emsp;&emsp;“空中碰撞”<br/>
### 4、Cache-Control
&emsp;&emsp;<br/>
## 二、缓存三要素
&emsp;&emsp;<br/>
## 三、面试题
&emsp;&emsp;<br/>
## 四、缓存在实践中的应用
&emsp;&emsp;<br/>
## 五、浏览器刷新
&emsp;&emsp;<br/>
## 六、总结
&emsp;&emsp;<br/>

## 参考资料
***
[彻底弄懂 Http 缓存机制 - 基于缓存策略三要素分解法](https://mp.weixin.qq.com/s/qOMO0LIdA47j3RjhbCWUEQ)
[MDN-HTTP缓存](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Caching_FAQ)
[HTTP缓存控制小结](https://imweb.io/topic/5795dcb6fb312541492eda8c)
[Browser: F5 vs Ctrl+F5](https://mocheng.wordpress.com/2007/11/30/browser-f5-vs-ctrlf5/)

Cache-Control: public private no-cache no-store
1、该不该发出网络请求
2、发出网络请求后，服务器是重传报文还是发送304

第一步：服务器响应首部定义缓存策略（是否使用缓存，缓存对比策略是什么样的
（ETag还是Last-Modified））
第二步：需要再次请求资源时，客户端根据响应首部来决定是发送请求还是从浏览器缓存中读取。
第三步：如果客户端发送请求，服务器决定是发送全部资源并且状态码200，还是仅仅发送首部，状态码304