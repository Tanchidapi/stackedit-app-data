# Pgtbl
本实验目的是要增强对虚拟内存中页表的管理机制，通过阅读并修改内核中与页表相关的函数实现，现有网上常见的题解是该系列22年的，本题解的题目是21年的
开始前先简单说明xv6的页表管理机制，虚拟地址va的用于页表寻址的位有27位，9位一组，也就是一共是三级页表，后十位是相关的属性位，机制如下图所示![输入图片说明](/imgs/2025-08-13/UZqIRcUFSskBQSV4.png)![地址转换](https://i-blog.csdnimg.cn/blog_migrate/3d49e116164758cd91e905c366296286.png)示意图
## p1 Speed up system calls-ez

<!--stackedit_data:
eyJoaXN0b3J5IjpbMTE5OTMxNDgzMSw0MDUyMzY4MThdfQ==
-->