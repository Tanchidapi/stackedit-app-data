## TCP的工作原理与构建
![输入图片说明](/imgs/2025-07-15/kNpSmTWeCrWrEuvK.png)示意图

tcp连接由三次握手建立：
第一次为A发送syn到B，同时发送用于识别的字节流基数，标识发送的数据从多少开始
第二次由B发送syn+ack到A，ack表示确认请求并同意建立连接，syn表示B想与A的tcp层建立连接，发送一个数字标识反向字节流的起始
最后A回复一个ack，表示同意反方向通信的请求

tcp的关闭：
首先A向B发送fin消息表示finish
然后B确认A没有数据要发送，并停止接收来自A的新数据，B发送（data+）ack，其中data是B中可能还未发送完的数据
当B要关闭向A的tcp连接时同理，两边都关闭后tcp连接完全关闭，相关状态机可以被安全移除
![输入图片说明](/imgs/2025-07-15/UrmOlWODh1XLsmj3.png)
tcp提供的服务

tcp通过四种机制保证字节流可靠：
1. tcp层收到消息后向发送方发送确认消息
2. 通过校验和检测损坏的数据
3. 通过序列号检测丢失的数据
4. 流量控制防止接收方超载
![输入图片说明](/imgs/2025-07-16/qjoF2pV4jYaxAlD4.png)tcp的报头示例图

几个重要的位置含义：
 1. source port：表示发送方端口
 2. destination port：表示接收方端口
 3. sequence：表示第一个开始的字序列号
 4. acknowledge sequence：期待的下一个字序列号
 5. flag：包括一系列符号为，标识是否确认通信、是否关闭通信、是否立即传递数据、是否同步等
 6. checksum：校验和
 7. windowsize等：报头大小

tcp连接通过tcp和ip首部的五部分信息进行唯一标识，ip唯一标识端点，tcp的ip协议id告诉所用传输协议为tcp，端口号标识了端主机上的应用程序进程
为避免源端口冲突，主机为每个新连接递增源端口号，该字段16位，故需要64k的连接才会出现重复
为减少字节流混淆，tcp连接时使用随机的初始字序列号来标识字节流中的字节
## UDP的工作原理与构建
udp一般是在对于数据的实时性更看重的情况下所使用的一个传输层协议，因此报头也更加简单，因为不需要构建可靠的双向字节流
![输入图片说明](/imgs/2025-07-16/1SuMi0eLpzRmN8Nj.png)udp提供的服务

![输入图片说明](/imgs/2025-07-16/a4wEIGMzRW9WF9wL.png)udp报头示意图

1. source port：源端口号
2. destination port：目的端口号
3. checksum：校验和，可选，如不设置则为全0，设置可以包含IP v4的一部分信息，用于检验是否发送到错误地址
4. length：udp数据大小
## ICMP的工作原理与构建
icmp（internet control message protocol）能够帮助在终端主机和路由器之间传递有关网络层的消息，通常用于报告错误条件，诊断问题，找出数据报路径等，是保证网络层工作的一部分
![输入图片说明](/imgs/2025-07-16/EShPyqVa2aJkxAfj.png)网络层工作的三大依赖

icmp运行在网络层之上，严格来说是传输层协议
![输入图片说明](/imgs/2025-07-16/NnsIKnFgkdxi0twl.png)icmp的服务模型

![输入图片说明](/imgs/2025-07-16/q4jemG4JGAJJnqAq.png)工作例子

A发送B数据报过程中，在某个路由器上找不到到B的下一跳，于是返回A错误信息，
<!--stackedit_data:
eyJoaXN0b3J5IjpbMjEyNDczNDE0OCwtODEwMDA2Mjc5LC0xNT
kzNDUxMjQsODk1OTYzNDMsLTE4OTI1MjM1NzUsLTQ3MTc1Mjk1
LC0yMDg4NzQ2NjEyXX0=
-->