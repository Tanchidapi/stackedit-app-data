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
实现：关于素数筛的算法可以网上搜索/问ai，本程序中通过fork创建子进程递归和管道pipe实现，父进程从2开始向管道中写入所有的数，子进程从管道中读出数并打印出第一个数，创建子进程，然后读出剩下的数，如果剩下的数不能被第一个数整除则再写入管道中，然后递归调用子进程即可
```c
#include "kernel/types.h"
#include "kernel/stat.h"
#include "user/user.h"

void prime(int *fd){
    int p, d;
    close(fd[1]);
    if(read(fd[0], &p, 4) != 4){
        printf("error: Can't write!\n");
        exit(1);
    }
    printf("prime %d\n", p);
    if(read(fd[0], &d, 4)){
        int fd_of_child[2];
        if(pipe(fd_of_child) < 0){
            printf("error: fail to create the pipe!\n");
            exit(1);
        }
        if(fork() == 0){
            prime(fd_of_child);
        }
        else{
            close(fd_of_child[0]);
            do{
                if(d % p != 0){
                    if(write(fd_of_child[1], &d, 4) != 4){
                        printf("error: Can't write!");
                        exit(1);
                    }
                }
            }while(read(fd[0], &d, 4));
            close(fd[0]);
            close(fd_of_child[1]);
            wait(0);
        }
    }
    exit(0);
}

int
main(int argc, char *argv[]){
    if(argc != 1){
        printf("fault: wrong number of args!\n");
        exit(1);
    }
    int fd[2];
    if(pipe(fd) < 0){
        printf("error: fail to create the pipe!\n");
        exit(1);
    }
    int start = 2, end = 35;
    if(fork() == 0){
        prime(fd);
    }
    else{
        close(fd[0]);
        for(int i = start; i <= end; i++){
            if(write(fd[1], &i, 4) != 4){
                printf("error: Can't write!\n");
                exit(1);
            }
        }
        close(fd[1]);
        wait(0);
    }
    exit(0);
}
```
## p4 find -moderate
要求：完成c程序find，通过ls函数实现对当前目录及其所有子目录的特殊名称的文件查找
实现：通过递归遍历当前目录文件及其子目录，如果当前路径指向文件且文件名匹配则打印当前文件，如果路径指向目录则记录当前路径，如果当前路径为当前目录或者父目录则不进入递归，否则扩展路径到子目录，对子目录中所有文件遍历递归
```c
#include "kernel/types.h"
#include "kernel/stat.h"
#include "user/user.h"
#include "kernel/fs.h"


void
find(char *path, char *name){
    int fd;
    char *p, buf[512];
    struct dirent de;
    struct stat st;
    if((fd = open(path, 0)) < 0){
        fprintf(2, "error: fail to open %s\n", path);
        exit(1);
    }
    if(fstat(fd, &st) < 0){
        fprintf(2, "error: Can't get stat %s\n", path);
        close(fd);
        exit(1);
    }
    switch(st.type){
        case T_FILE:
            p = path + strlen(path);
            while(p >= path && *p != '/') p--;
            p++;
            if(strcmp(p, name) == 0){
                printf("%s\n", path);
            }
            break;
        case T_DIR:
            if(strlen(path) + 1 + DIRSIZ + 1 > sizeof(buf)){
                fprintf(2, "find: path too long!\n");
                exit(1);
            }
            strcpy(buf, path);
            p = buf + strlen(buf);
            *p++ = '/';
            while(read(fd, &de, sizeof(de)) == sizeof(de)){
                if(de.inum == 0 || 
                strcmp(de.name, ".") == 0 || 
                strcmp(de.name, "..") == 0) continue;
                memmove(p, de.name, DIRSIZ);
                p[DIRSIZ] = 0;
                if(stat(buf, &st) < 0){
                    fprintf(2, "error: Can't get stat %s\n", buf);
                    continue;
                }
                if(st.type == T_FILE){
                    if(strcmp(de.name, name) == 0){
                        printf("%s\n", buf);
                    }
                }
                else if(st.type == T_DIR){
                    find(buf, name);
                }
            }
            break;
    }
    close(fd);
}


int
main(int argc, char *argv[]){
    if(argc != 3){
        fprintf(2, "error: wrong number of arg!\n");
        exit(1);
    }
    char *path = argv[1];
    char *name = argv[2];
    find(path, name);
    exit(0);
}
```

<!--stackedit_data:
eyJoaXN0b3J5IjpbLTQxMTM4NjEyNSw1MTM4MDM1MjUsODA0Nz
U2OTYwLDUzMTk5MjE4OCwxMTQyNzkxODksMTMxMjExMTUyMCwt
MzA4MjM2Mzk3LDE5MjM5ODUzNTEsLTYwNTIzOTg4Ml19
-->