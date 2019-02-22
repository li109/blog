# 网络学习笔记（一）：TCP连接的建立与关闭
&emsp;&emsp;五层网络模型分为：物理层、数据链路层、网络层、传输层、应用层。其中，传输层有两种主要协议：面向连接的TCP（Transmission Control Protocol 传输控制协议）、无连接的UDP（User Datagram Protocol 用户数据报协议）。<br/>
&emsp;&emsp;TCP是面向连接的传输层协议，提供点对点的可靠交付服务。TCP是面向字节流的，提供全双工通信，允许连接双方任何时候都能发送数据。<br/>
### 一、TCP数据段
&emsp;&emsp;TCP传送的数据单元是**报文段**，TCP报文段分为首部与数据两部分，首部的各字段能体现TCP的全部功能。TCP数据报被封在IP数据报里。<br/>
&emsp;&emsp;TCP数据段首部格式如下图所示：<br/>
&emsp;&emsp;
### 二、建立连接
### 三、关闭连接
### 四、总结