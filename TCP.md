# TCP相关的读书笔记
## 摘自《TCP.IP协议详解》
## TCP的首部
TCP数据被封装在一个IP数据报中。
TCP首部中有6个标志比特。它们中的多个可同时被设置为1。
* URG     紧急指针(urgent pointer)
* ACK     确认序号有效
* PSH     接收方应该尽快将这个报文段交给应用层
* RST     重建连接
* SYN     同步序号用来发起一个连接
* FIN     发端完成发送任务

![TCP报文格式](http://blog.chinaunix.net/attachment/201304/7/22312037_1365321234nnNc.png)
## TCP连接的建立
### 三次握手(three-way handshake)
1) 请求端(通常称为客户)发送一个SYN段指明客户打算连接的服务器的端口，以及初始序号(ISN)。这个SYN段为报文段1。
2) 服务端发回包含服务器的初始序号的SYN报文段(报文段2)作为应答。同时，将确认序号设置为客户的ISN加1，已对客户的SYN报文段进行确认。一个SYN将占用一个序号。
3) 客户必须将确认序号设置为服务器的ISN加1，已对服务器的SYN报文段进行确认(报文段3)。

![TCP三次握手](http://blog.chinaunix.net/attachment/201304/8/22312037_1365405910EROI.png)
