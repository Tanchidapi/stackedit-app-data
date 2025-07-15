## TCP的工作原理与构建
![输入图片说明](/imgs/2025-07-15/kNpSmTWeCrWrEuvK.png)示意图

tcp连接由三次握手建立：
第一次为A发送syn到B，同时发送用于识别的字节流基数，标识发送的数据从多少开始
第二次由B发送syn+ack到A，ack表示确认请求并同意建立连接
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTI5NTA4Nzc3LC0yMDg4NzQ2NjEyXX0=
-->