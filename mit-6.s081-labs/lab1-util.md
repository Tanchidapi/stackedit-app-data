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
## p2 pingpong -ez
要求：完成一个c程序pingpong，实现创建一对父子进程并相互通信，打印通信信息
实现：通过fork函数实现创建子进程，getpid获得进程序列号，用pipe函数创建管道实现父子进程的通信。pipe接收一个长度为2的整型数组，并为其创建两个文件描述符，0为读端，1为写端，逻辑上是一个环形队列，通过write函数向管道写端写入数据，read函数从管道读端写入数据，在进行对应操作时需关闭另一端，读写函数返回值为字符串长度，但printf打印字符串需要终止符，故读取5字节
```c
#include "kernel/types.h"
#include "kernel/stat.h"
#include "user/user.h"
#define PINGPONGBUF 5

int
main(int argc, char *argv[]){
    if(argc != 1){
        printf("error: wrong numbers of arg!\n");
        exit(1);
    }
    int fd[2], pid, cur_pid;
    char inbuf[PINGPONGBUF];
    char *par = "ping";
    char *chi = "pong";
    if(pipe(fd) < 0)
        exit(1);
    if((pid = fork()) > 0){
        cur_pid = getpid();
        if(write(fd[1], par, PINGPONGBUF) != 5){
            printf("Can't write to child!\n");
            exit(1);
        }
        close(fd[1]);
        wait(0);
        if(read(fd[0], inbuf, PINGPONGBUF) != 5){
            printf("Can't read from child!\n");
            exit(1);
        }
        printf("%d: received %s\n", cur_pid, inbuf);
        close(fd[0]);
        exit(0);
    }
    else{
        cur_pid = getpid();
        if(read(fd[0], inbuf, PINGPONGBUF) != 5){
            printf("Can't read from parent!\n");
            exit(1);
        }
        printf("%d: received %s\n", cur_pid, inbuf);
        close(fd[0]);
        if(write(fd[1], chi, PINGPONGBUF) != 5){
            printf("Can't write to parent!\n");
            exit(1);
        }
        close(fd[1]);
        exit(0);
    }
    exit(0);
}
```
## p3 primes -moderate
要求：完成一个c程序primes，实现素数筛，打印出2-35的素数
实现：关于素数筛的算法可以网上搜索/问ai，本程序中通过fork创建子进程递归和管道pipe实现，父进程从2开始向管道中写入不能被当前数整除的数，子进程从管道中
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE2NjU3MDIzMDcsMTMxMjExMTUyMCwtMz
A4MjM2Mzk3LDE5MjM5ODUzNTEsLTYwNTIzOTg4Ml19
-->