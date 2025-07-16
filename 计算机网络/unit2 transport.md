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

<!--stackedit_data:
eyJoaXN0b3J5IjpbODk1OTYzNDMsLTE4OTI1MjM1NzUsLTQ3MT
c1Mjk1LC0yMDg4NzQ2NjEyXX0=
-->