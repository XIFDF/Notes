## ARP相关笔记
### 原文地址<a>https://www.cnblogs.com/csguo/p/7527303.html</a>
* ARP（Address Resolution Protocol）即地址解析协议。<br>用于实现从 IP 地址到 MAC 地址的映射，即询问目标IP对应的MAC地址。
* 在网络通信中，主机和主机通信的数据包需要依据OSI模型从上到下进行数据封装，当数据封装完成后，在向外发出。
所以在局域网的通信中，不仅需要源目IP地址的封装，也需要源目MAC的封装。
* 一般情况下，上层应用程序更多关心IP地址而不关心MAC地址，所以需要通过ARP协议来获知目的主机的MAC地址，完成数据封装。
