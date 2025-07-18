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

A发送B数据报过程中，在某个路由器上找不到到B的下一跳，于是返回A错误信息，icmp截取ip报头和ip data的前8byte，然后加上错误类型type和代码code，作为ip data，然后加上ip报头，src为当前路由器，dst为A发送回去
![输入图片说明](/imgs/2025-07-16/xRskIsuJ9UX1t80U.png)常见错误类型与代码

ping指令常用于检测目标主机是否存活，是否连接
![输入图片说明](/imgs/2025-07-16/dDE6nRmrSaMUJYbe.png)ping的工作原理

traceroute可用于跟踪数据报经过的路径和每个路由器往返延迟，通过发送udp消息实现，要跟踪下一跳的路由器将ttl改为2即可，直到到达主机B，udp会在udp头特意设置一个B不知道的端口号，这样B就会发送回一条icmp消息，表示该端口不可达，A收到此消息时就会直到traceroute已完成
![输入图片说明](/imgs/2025-07-16/Pb3syEBywMfxTp3k.png)
traceroute的工作原理，要跟踪下一跳的路由器将ttl改为2即可
## 端到端原则
![输入图片说明](/imgs/2025-07-16/LBFErXlUWSO5kcGS.png)论文原文

简单来说，端到端原则就是要求特定的功能应该在通信的端点处实现，而不是在网络的中间节点，例如安全特性等，但是网络可以包含一个不完整的功能作为性能增强

![输入图片说明](/imgs/2025-07-16/V2WMSp3qAMalpTrQ.png)强端到端原则

强端到端原则表述网络的任务是尽可能高效、灵活的传输数据报，其他所有的事情应在端点完成，这也是分层思想的重要体现，核心思想是对于网络层的零信任以及端点的绝对责任
## Error Detection
网络通常使用三种方法进行错误检测：检验和（checksum），循环冗余验证（CRC，常用于以太网），消息认证码（MAC，用于传输层）
![输入图片说明](/imgs/2025-07-16/oMmzP3Zo74XRNfeg.png)三种方式的特点

### 校验和
![输入图片说明](/imgs/2025-07-16/fb0bC8vofPyAgGxt.png)ip校验和特点

ip校验和不保护数据部分，只校验头部，计算时忽略校验和本身字段，接收方验证时对整个头部执行相同计算，结果应为0xffff，ip校验和只有检测错误能力，但无法检测多比特错误，没有纠错能力
计算过程：
1. 先将校验和部分字段置零，将ip头部以16位2byte进行划分，若报头长度不是偶数则补零，补的零不参与传输，仅计算
2. 对所有分割后的十六位字进行反码加法，若结果中有溢出，则保留低16位后再加一
3. 对上一步的结果按位取反，结果放入校验和字段

检测时同样的方法进行计算，若结果不为0xffff，则说明报头损坏
### 循环冗余校验
![输入图片说明](/imgs/2025-07-16/X2ddEw6PXdrhuHQr.png)crc特点

crc对奇数位错误，连续的小于crc位数的错误，长错误有良好的检测能力，并且计算速度快，对硬件良好，因此十分适用于链路层的检错应用

![输入图片说明](/imgs/2025-07-16/bi6IQa0TuW9SR8Eg.png)crc长除法示意

crc通过将数据视为一个多项式，通过模2除法除以一个生成的预设多项式，得到的余数即为crc值校验码，生成多项式对应的实际使用值不存储最高次的系数作为位，即多项式的最高次幂永远为1，且最高次数为对应位数
计算过程：
1. 先生成一个多项式用于计算
2. 对数据进行预处理，在后面补上多项式对应位数的0
3. 进行模2除法运算，每一位异或不进位，得到余数长度为生成多项式的位数-1，余数即为crc值

接收方验证时用同样的多项式对整个数据（含crc值）进行模2除法，若余数为0则无错误，反之有错误，crc的强度取决于多项式
### 消息认证码
![输入图片说明](/imgs/2025-07-16/v3i8DmbgSMXeg5Fo.png)mac特点

mac在检错能力的基础上还增加了安全检测能力，mac本质上是一种信息加密的方法，事实上mac不保证能检测到任何错误

实现方法：
1. 发送方输入消息M和密钥S
2. 通过mac算法得到标签C
3. 发送M与C
4. 接收方通过接收到的M，结合密钥S再次计算C，对比接收到的C，若一致则消息真实且完整，否则可能被篡改或消息不完整
### 总结
![输入图片说明](/imgs/2025-07-16/bE7MdCfM33OPGC5c.png)三种检错方法比对

## 有限状态机
![输入图片说明](/imgs/2025-07-16/vH10rsDdfocNGUWb.png)有限状态机的示意图

![输入图片说明](/imgs/2025-07-16/YZsE5LGm8RkmGIfd.png)http请求的三态状态机

![输入图片说明](/imgs/2025-07-16/WG62BzN5nAPB9uNN.png)tcp连接的状态机示意图

上面四个状态表示了tcp连接的建立过程，中间的established是已建立连接的状态，在建立连接之后，关闭连接之前，下面六个状态表示连接如何关闭，底部状态closed表示连接已关闭，节点可以忽略它
## 流量控制
流量控制试图解决的问题是发送方发送数据速度比接收方快，通常通过简单的停等协议实现
![输入图片说明](/imgs/2025-07-18/1uPEdkKjlpCcGdOE.png)基本概念与方法

停等协议也是一种有限状态机
![输入图片说明](/imgs/2025-07-18/jFBcprHuNXR4A3kG.png)停等协议方法

![输入图片说明](/imgs/2025-07-18/LlUM7iMAcVtvYOu4.png)停等协议状态机

![输入图片说明](/imgs/2025-07-18/PDD0j10NYJkNt6ch.png)停等协议的四个例子

上述四个例子中的第四个是回复延迟的情况，这是所有可靠协议都会遇到的一个问题，这种情况下，第二次发送方收到的ack无法确认是哪一数据报对应的，停等协议下通过


<!--stackedit_data:
eyJoaXN0b3J5IjpbLTY1NDQyMjI1NiwtNjU0OTg3MjI4LC02MD
c1NDU5NTIsLTE1NTQ3MzcwMzYsLTE3NTk5OTMzNSwtNzM2Mjcz
NDc4LDE5MDM4MzczODAsOTk1NDE1MTQwLC0xOTkzNzcyMjQyLC
0yNzU0NTExMDksMTU2MDQzNjE4NiwtMTg1NDM1ODk5MCwxNDU2
MTA1MjM5LDEwNTk4ODMyNzcsLTI4MzMzNzE0OCw5MzM2MTYwOD
IsLTgxMDAwNjI3OSwtMTU5MzQ1MTI0LDg5NTk2MzQzLC0xODky
NTIzNTc1XX0=
-->