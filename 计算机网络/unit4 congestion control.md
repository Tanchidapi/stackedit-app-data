## 拥塞基本定义
### 拥塞的情况
![输入图片说明](/imgs/2025-09-18/BBudinJn2s2Lz8Eg.png)总结
拥塞是不可避免的，流量控制是解决拥塞的关键
### 公平性有利于设计拥塞控制
![输入图片说明](/imgs/2025-09-18/Dphl7bM1AsV9ma65.png)示意图

方案一保证了吞吐量，但是方案二在第二条链路上保证了公平性，设计拥塞时要提前达成公平性

![输入图片说明](/imgs/2025-09-18/7uSqvVPRANRCjw1L.png)常用的公平方法——最大最小原则

简单来说就是将最大流最小化，即不能通过减小一个小流的速率来增大一个比他大的流的速率，本质上为共享一个链路的瓶颈，上述情况的方案二就是在最大最小原则下的速率分配情况

![输入图片说明](/imgs/2025-09-18/oyHfz87EVB0Hbnda.png)以单链路为例的一个分配例子

先根据输入链路数量与输出lian'lu
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE3NDg3NzQzODYsLTE3NTI2ODM2MjgsLT
ExMTU3MTI3MTcsLTE0MzE3MzQ2NTksMTMzNzgxODg4OF19
-->