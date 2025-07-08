# Syscall
这个实验的主要目的是通过编写两个系统调用函数来加强对系统调用的理解，其中包含了从用户态到内核态的切换过程
实验一共两题，两题在添加系统调用步骤上差不多，区别在各自的具体实现
开始作答前先解释本次实验要用到的头文件的源文件的大概作用
## /user目录下的文件
1. user.h
这个文件列出了用户态下可以调用的系统函数以及其他源文件（如ulib.c）的函数，给出函数的名称、返回类型及传参类型
2. usys.pl
这是一个perl脚本，运行后会生成.s汇编文件，里面定义了每个系统调用函数的跳板函数，会将用户态下的系统调用的id寄存器，然后执行系统调用跳到内核态同一系统调用处理函数syscall
## /kernel目录下的文件
1. syscall.h
这个文件列出了系统调用函数的一系列id，所有系统调用函数都有自己对应的唯一id
2. syscall.c
这个文件是系统调用处理的主要函数，会根据使用系统调用时传入的id，选择对应的系统函数并执行调用，也包含了获取系统函数所需参数的函数
3. sysproc.c
这个文件包含了内核态下，执行的系统函数的具体实现，即对应的系统函数怎么获取用户态传来的参数，然后如何分析并使用系统函数
4. proc.h
这个文件是对于操作系统运行时进程相关的对象的定义，如上下文，trapframe，进程，cpu的状态，由结构体的形式实现
5. proc.c
这个文件是在已有的头文件的定义上，对进程等对象的一些基础操作，如进程的初始化与分配等
6. defs.h
这个头文件定义了内核态下一系列的函数的原型，即函数名，返回值与传参类型
7. kalloc.c
这个文件包含了内核分配相关的函数的具体实现
## 系统调用流程
根据上面的文件列表及其分工我们可以看出在用户态下执行系统调用的流程：
首先，用户申请调用系统函数，这需要要调用的函数包含在系统头文件user.h中，这样用户才可以调用该函数（其实是调用一个跳板函数，因为用户态没有权限直接访问内核态的函数等资源，防止用户越权无意或恶意访问硬件资源造成损坏），然后根据usys.pl脚本生成的汇编文件中的跳板函数，调用跳转到内核态，在syscall.c文件中处理调用的系统函数id，并在syscall.h头文件中找到id对应的系统调用函数，然后syscall.c中根据对应的函数执行在sysproc.c中具体的函数实现，完成系统调用，最后返回用户态

ps：很喜欢用南大蒋老师在他们的os课上对于软件视角和硬件视角下的os的描述，在软件的视角下，系统调用就好比向上天祈祷，而os就是这个老天爷，软件需要系统函数执行操作或申请相关硬件资源，自己又没有权限/能力（用户态），所以告诉os说：我需要xxx，然后闭眼等待，这个时候世界静止了，老天爷（os）去帮你实现愿望（将cpu所有权交给到内核，切换到内核态），然后再睁眼时所有需要的东西就得到了（系统调用完成，回到用户态），而老天爷干了什么，就是内核态的事情了，老天爷（os）怎么在内核中完成用户所需的系统函数，这是我们在实验中要具体实现的。当然，在lab2中，我们除了要完成内核中系统函数的具体实现之外，还需要完成如何顺利的祈祷（在用户态中写好接口）
## p1 trace -ez
要求：完成一个系统调用函数，实现对于所有系统调用函数的跟踪，即用户态请求系统调用函数时，打印出内核态在该过程使用的特定系统函数的信息
实现：
1. 先在用户态提供跳板函数及接口，在user.h中加入定义：
```c
......
int trace(int);
```
因为trace函数需要一个整数掩码作为指定的跟踪的系统函数，所以需要一个int参数，掩码的实现后面说明，同时返回值int，当返回值小于0时说明调用失败
2. 在usys.pl中声明跳板函数：
```perl
......
entry("trace");
```
这个语句会在脚本运行生成的汇编文件中提供调用这个系统函数的跳板函数
3. 在syscall.h中定义trace函数及其id：
```c
......
#define SYS_trace 22
```
这个id是为了在syscall.c文件中处理系统调用时能够找到对应id的系统函数并调用执行
4. 在syscall.c中处理调用：
```c
......
const char *syscall_names[] = {
......
};
......
extern uint64 sys_trace(void);
......
static uint64 (*syscalls[])(void) = {
......
[SYS_trace]  sys_trace,
};
......
void
syscall(void)
{
	...
	if(...){
		...
		if((p -> syscall_trace >> num) & 1){
			printf("%d: syscall %s -> %d\n",p->pid, syscall_names[num], p->trapframe->a0); // syscall_names[num]: 从 syscall 编号到 syscall 名的映射表		}
    }
......
```
这里的extern声明sys_trace表示这是一个外部函数，作用是编译时告诉编译器这个函数的定义位于其他文件（本例中是在sysproc.c中）中，从而实现跨文件共享函数的功能，static一行的语句是声明了一个函数指针数组，成员都是函数的地址，中括号中是对应的id，作为下标使用，这种数组声明方式更加高效，且将函数id与下标绑定，寻找对应函数时更方便，最后在syscall中加入trace的结果，如果掩码有效（trace被调用）则跟踪掩码指定的系统函数，trace的调用会改变进程中syscall_trace的值，据此可以判断是否调用了trace函数及其要跟踪的系统函数，syscall_names是一个辅助字符串数组，用于记录各自id对应的系统函数名称，便于trace时打印对应函数名
5. 在proc.h中的对进程的定义的结构体中加上trace掩码：
```c
......
struct proc {
	......
	uint64 syscall_trace;
}
```
在进程中加上syscall_trace掩码作为标识，当该进程调用了系统函数trace后该标识会被置位，值为对应的掩码
6. 在proc.c中，进程初始化结束后分配时为syscall_trace赋默认值，防止内存中垃圾数据干扰，同时修改fork使子进程继承父进程的syscall_trace标识：
```c
......
int
fork(void)
{
	......
	np -> syscall_trace = p -> syscall_trace;
	......
}
......
static struct proc*
allocproc(void)
{
	......
	p -> syscall_trace = 0;
}
```
7. 在sysproc.c中实现具体的trace代码：
```c
uint64
sys_trace(void)
{
	int mask;

	if(argint(0, &mask) < 0)
		return -1;
	
	myproc() -> syscall_trace = mask;
	return 0;
}
```
将参数中的第一个（trace也仅有一个参数）读取到mask中，然后获取调用该函数的进程，并将其进程结构体中的syscall_trace的值赋为对应的掩码，掩码的解释为右移要跟踪的系统调用编号，如果与1按位与的结果为true，则跟踪，比如read的id为5，调用trace时输入的参数为32，则掩码被赋值为32，syscall.c文件中执行syscall函数时执行了read系统函数，随后判断掩码右移read对应id，即右移5位，值为1，再与1按位与结果为true，则说明跟踪到了要求的系统函数，随后打印出系统函数调用的信息
## p2 sysinfo -moderate
要求：完成一个系统调用函数，作用为打印出当前剩余的内存以及进程数量
实现：
1. 用户态下的文件处理与p1类似，不再赘述
因为我们要统计剩余的内存以及当前进程的数量，故我们要两个辅助函数，一个为mem_count，用于计算剩余内存，一个为proc_count，用于统计当前进程数
2. 在defs.h中定义两个辅助函数：
```c
......
// kalloc.c
......
uint64 mem_count(void);
......
// proc.c
......
uint64 proc_count(void);
......
```
因为mem_count是计算内存相关的函数，故声明在kalloc.c这个系列中，proc_count是进程相关的函数，故声明在proc.c这个系列中
3. 在kalloc.c中实现mem_count：
```c
uint64
mem_count(void)
{
	acquire(&kmem.lock);

	uint64 mem_bytes = 0;
	struct run *r = kmem.freelist;
	while(r){
		mem_bytes += PGSIZE;
		r = r -> next;
	}

		release(&kmem.lock);
		return mem_bytes;
}
```
在这个函数中，我们要先取得锁，防止竞态条件出现干扰，kmem是内核中管理内存的结构体，xv6中的内存空闲形式为空闲链表法，其他常见的方法还有空闲表法、位示图法、成组链接法，通过空闲
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTU5MDc4NjI4MSwtODM1NDU1Nzg0LC0xMD
AxMjg2NzIyLDE4OTAxODc3MTUsMjQ3MDY3MzMzLC0zNzY1MjQ2
OSwxNzEwODA1NywtNDU5OTg2MjkxLDEwMDQ1NTcyNjUsMTAzOD
MxMDQ3Niw5ODA2MjY4NjQsNzIxNDExODc2LDE1NTI2NDA5MTMs
LTEwMzU2MzQzNzJdfQ==
-->