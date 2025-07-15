## TCP的工作原理与构建
![输入图片说明](/imgs/2025-07-15/kNpSmTWeCrWrEuvK.png)示意图

tcp连接由三次握手建立：
第一次为A发送syn到B，同时发送用于识别的字节流基数，标识发送的数据从多少开始
第二次由B发送syn+ack到A，ack表示确认请求并同意建立连接，syn表示B想与A的tcp层建立连接，发送一个数字标识反向字节流的起始
最后A回复一个ack，表示同意反方向通信的lian'jie
<!--stackedit_data:
eyJoaXN0b3J5IjpbMjI1NDA0MzgzLC0yMDg4NzQ2NjEyXX0=
-->