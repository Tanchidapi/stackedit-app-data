## what is packet switching
### circuit switching
分组交换的前身是电路交换，用于早期的电话系统，可以通过三步解析它，首先是拨号，这告诉了交换机我们想要连接的端点，从而建立专用的电路，然后是使用，在我们说话的过程中，声信号被转换成电信号通过交换机建立的电路传递，最后是挂断，要移除该专用电路和路径上交换机的任意状态![输入图片说明](/imgs/2025-08-15/bjHCJUR65UbWrACX.png)

示意图

![输入图片说明](/imgs/2025-08-15/6C6cE8LGdb6Mf3ij.png)电路交换的性质

电路交换在电话系统中运行的很好，但是用于计算机通信时会有一些问题：
1. 效率不高，计算机通信中常见猝发通信，并且可能在建立连接后进入长时间的沉默，如浏览网页时，电路交换在专用电路建立时是独占的，不允许在结束断开连接时被别的通信使用该线路，因此易造成效率低下
2. 多样化的速率，计算机通信中不同应用不同时间的传输速率一般是不同的，电路交换的单一线路提供的速率不符合实际使用情况
3. 状态维护，在电路交换中，我们需要维护所有的状态，如果出现故障，我们需要进入并更改这些状态，来重新路由电话
### packet switching
分组交换网络由端主机、链路和分组交换机组成，路由器也是分组交换机的一种，它处理互联网地址![输入图片说明](/imgs/2025-08-15/mJ4gWgnQSEnA2F66.png)分组交换示意图

分组交换机有缓冲区（buffer），假设有两个数据包以输出链路的全速率同时到达一个分组交换机时，数据包交换机必须保留其中一个，一次只能发送一个，因一个交换机可能有多个输入链路，因此要有一个缓冲区来保存多个数据包
![输入图片说明](/imgs/2025-08-17/oIDhbixS2vnV8tr1.png)
分组交换的性质

### why use it
![输入图片说明](/imgs/2025-08-18/ZP8H1ACvwOhl9sdD.png)使用分组交换的原因

第三个重要原因是，互联网最初被设计成已有网络的互连，当时所有的通信网络/计算机网络都是分组交换的，互联网要连接这些网络就也要使用分组交换

![输入图片说明](/imgs/2025-08-21/MIxlXESMa6CeEBWx.png)分组交换与电路交换的区别

![输入图片说明](/imgs/2025-08-21/YzhCvDTpst9xigU5.png)分组交换的特点

## 传播延迟、分组延迟与排队延迟
### useful definition
![输入图片说明](/imgs/2025-08-18/5ECYxlTHARUFFHjq.png)传播延迟的定义
和链路大小无关，只和链路长度和传播速度（一般是光速）有关

![输入图片说明](/imgs/2025-08-19/eTc1S2qTnYxMG0rD.png)
分组延迟的定义
是从分组的第一个bit被放到链路上开始，到最后一个bit被放到链路上所用的时间，本质上取决于我们可以打包（packet）的有多紧
### end-to-end delay
端到端延迟是从在第一个链路上发送的第一个bit开始，直到数据包最后一个bit到达目的地所用的时间，是分组延迟和传播延迟的和
![输入图片说明](/imgs/2025-08-19/c5a7bk72okhgAU3f.png)端到端延迟
![输入图片说明](/imgs/2025-08-19/MPpCiS46oHxtvSFq.png)example

链路在所用用户的数据包之间是共享的，多个数据包在交换机处可能需要在缓冲区等待，称有大量数据包在队列等待通过的行为的下一条链路为拥塞
在缓冲区存在的情况下，端到端的延迟还需要多考虑在各个交换机的等待时间，即排队延迟
![输入图片说明](/imgs/2025-08-19/9vgMw51pV4ICtCZs.png)考虑排队延迟后的公式

### queuing delay
一些应用，如视频播放器/网站、语音通话、音乐软件等，会更看重数据的实时性，因此其对于排队延迟更敏感，为解决排队延迟带来的问题，一般会采用数据缓冲区，以防一些数据包延迟或没有及时到达，出现短暂的故障，设计缓冲区时一般要考虑提前多长时间
![输入图片说明](/imgs/2025-08-20/OPoY2SFJlcqbmg3i.png)示意图

![输入图片说明](/imgs/2025-08-20/5evYBZY1WEE23iFI.png)上图是发送、接收、播放与缓冲的字节的关系图，其中，纵坐标是累积的字节数，横坐标是时间，曲线间水平方向的差分别表示了变化的延迟时长（因为排队延迟）和缓冲所用的时间，竖直方向上的差分别表示了缓冲路径上缓冲的数据量有多大和（客户端）缓冲区的字节数，接收的字节的曲线的斜率体现了最后一个链路的速率（上限）

![输入图片说明](/imgs/2025-08-21/fplG2uQufPLhV8iS.png)客户端视角

![输入图片说明](/imgs/2025-08-21/wL6D9KIiCK5MimfO.png)缓冲区不够的情况
一般客户端会通过冻结屏幕等待缓冲来解决

![输入图片说明](/imgs/2025-08-21/3QPXRramZMBKMHfj.png)playback buffer summary

![输入图片说明](/imgs/2025-08-21/aSUVzJrgAPmAT5Qo.png)end-to-end delay summary

## queue models
### simple deterministic queue model
![输入图片说明](/imgs/2025-08-21/LghPru4OW7BsRy2A.png)示意图

Q是排队中的字节数，A是累积到达的字节数，D是输出离开的字节数，均为t的函数，R是输出链路的速率
![输入图片说明](/imgs/2025-08-21/wbexRll5THJIBDN6.png)坐标下的示意图
绿色曲线为A，黄色是D，标注的红色即为Q，是绿色和黄色线的竖直距离，水平差表示了一个字节经过了多久才离开路由器，用d表示其延迟时长

![输入图片说明](/imgs/2025-08-21/FA5Yh0VqjnMIwva2.png)计算题例题，计算平均队列占用率

### small packets reduce end to end delay
![输入图片说明](/imgs/2025-08-21/7mli5TVMNRLTyW5Z.png)用更小的数据包的原因
有点类似于计算机组成原理中的流水线，分成更小的包发送可以极大的提高发送效率

### statistical multiplexing
![输入图片说明](/imgs/2025-08-21/JH61GFGrvoDBd8VR.png)统计多路复用
用于处理多输入单输出时，输出链路跟不上输入速率的问题

![输入图片说明](/imgs/2025-08-21/RAJYhNlFJuDJCyIt.png)example for statistical multiplex
AB两个流量相加预期会得到统计复用增益

![输入图片说明](/imgs/2025-08-21/JMEUxaGVM7GLThIb.png)统计复用增益的计算
两种不同的情况，一种是未使用缓冲区，一种使用了缓冲区，使用了缓冲区的情况下值会更大，总之，统计复用可以让我们在一个链路上高效的承载多个流，这也是我们使用分组交换的主要原因之一

### workd example
![输入图片说明](/imgs/2025-08-21/6Y7P9I9cDxueVUrX.png)questionA

![输入图片说明](/imgs/2025-08-21/EUTrqMbAG5d0P33Y.png)questionB，计算了平均每个bit的延迟

![输入图片说明](/imgs/2025-08-22/0DMWEPwqPbAPAoVg.png)questionC，如果随机到达的情况下，保持平均每秒一个100bit的包到达，问占有率如何变化，答案是变大，下图两个例子解释了QC的答案

![输入图片说明](/imgs/2025-08-22/GakiJmB6FrFO1ZFo.png)因为输出速率不变，在case2中会导致需要两倍的时间去输出存储的bit

![输入图片说明](/imgs/2025-08-22/AgrL6d6V3bukTrGN.png)questionD，问再套娃一个队列的情况下，第二个队列的占用率，答案是零，因为不缓存数据

## useful queue properties

<!--stackedit_data:
eyJoaXN0b3J5IjpbLTI0MTkzNTAyMCw2NTUxNzE1MTUsLTEzMD
E2MjYzNDUsLTIwODc1OTIyODYsLTIxMTg3NDUyNDIsLTE0OTMw
NTc1NDcsMTE1MDQ3MDMzOSwxMTYzNzM5OTAsNDAxMDQxODIyLC
0xOTIwNDYzMDA1LDY5NzM5OTY2Niw1NjU3NTk0NDgsLTk3NTMx
MjkyMiw4MDUxOTE4NTUsMjQ5MTQwNDUwLDY1ODcxNzU0NiwtOD
MyOTMyNDU5LC0yMTQyNzk2MDU1LDMyNjg4MTYwMSwxMzk3MTQ0
MjUxXX0=
-->