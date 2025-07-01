# Util
这个实验主要加强对系统调用的应用，编写c程序，通过系统调用函数完成题目所给题目要求，在qemu虚拟机的xv6环境下实现程序运行

qemu的搭建根据mit官网的实验指导书运行到make这一步会出问题，解决方案见
https://www.bilibili.com/video/BV11K4y127Qk?t=5.0， 评论区有精简版
所有实验过程中的调试（用qemu-gdb）、本地评测等问题见https://www.bilibili.com/video/BV1Nkxre2Ewe?t=576.0
## p1 sleep -ez
要求：完成一个c程序sleep，实现暂停（sleep）用户从终端键入的时长
实现：在程序中调用系统函数sleep即可，该系统函数接收一个整型参数，暂停所给的时长。编写时先判断参数数量然后直接进行调用即可，使用atoi函数将参数字符串转为对应整数
```c
#include "kernel/types.h"
#include "kernel/stat.h"
#include "user/user.h"

int main(int argc, char *argv[]){
    if (argc != 2){
        fprintf(2, "Please enter a number!\n");
        exit(1);
    }else{
        sleep(atoi(argv[1]));
        exit(0);
    }
}
```
## p2  
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTM0NTE5NDUyMCwtNjA1MjM5ODgyXX0=
-->