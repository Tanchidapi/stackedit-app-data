# Pgtbl
本实验目的是要增强对虚拟内存中页表的管理机制，通过阅读并修改内核中与页表相关的函数实现，现有网上常见的题解是该系列22年的，本题解的题目是21年的
开始前先简单说明xv6的页表管理机制，虚拟地址va的用于页表寻址的位有27位，9位一组，也就是一共是三级页表，后十位是相关的属性位，机制如下图所示![输入图片说明](/imgs/2025-08-13/UZqIRcUFSskBQSV4.png)![地址转换](https://i-blog.csdnimg.cn/blog_migrate/3d49e116164758cd91e905c366296286.png)示意图
## p1 Speed up system calls -ez
要求：当要执行一些系统调用时，如getpid()，获取当前进程号，需要通过用户态与内核态的交叉，某些操作系统通过在用户与内核间共享只读的数据来实现加速系统调用，实现上述操作
实现：
在lab2中，我们已经知道了几个头文件的作用，这边不再复述。根据提示，我们应在kernel的proc文件中完成我们要共享的数据的映射，并记得在这之前要在allocproc文件中完成对应页表的分配与初始化，其实这块共享的数据就是在一个物理内存，然后将它的pa映射到用户态的进程页表中，这样用户在执行系统调用要使用的数据时就不用再来回切换获取，本题特指进程号pid
1. 根据学到的进程虚拟内存的分布图，如下![输入图片说明](/imgs/2025-08-14/uw4sKHFW0Mjb4Dhp.png)虚拟内存分布图
我们可以知道，我们要分配的这个页面应该在trapframe下方，heap的上方（关于trampoline和trapframe的作用会在后续课程学到，并在下一实验中使用），因此，在kernel中的memlayout.h添加该页面与相关结构体
```c
······
#ifdef LAB_PGTBL
#define USYSCALL (TRAPFRAME - PGSIZE)
struct usyscall{
	int pid;
};
#endif
```
2. 为该页面添加映射
因为这是一个进程的页面，所以我们要在进程的相关文件中去增加这个页面的映射代码
在proc.c中添加：
```c
pagetable_t
proc_pagetable(struct proc *p)
{
	······
	if(mappages(pagetable, USYSCALL, PGSIZE,
	(uint64)(p -> usyscall), PTE_R | PTE_U) < 0){
		uvmunmap(pagetable, TRAMPOLINE, 1, 0);
		uvmunmap(pagetable, TRAPFRAME, 1, 0);
		uvmfree(pagetable, 0);
		return 0;
	}

	return pagetable;
}
```
3. 为该页面分配内存
参考trapframe即可
在proc.h中添加：
```c
struct proc {
	······
	struct usyscall *usyscall;
}
```
在proc.c中添加：
```c
static struct proc*
allocproc(void)
{
	······
	if((p -> usyscall = (struct usyscall *)kalloc()) == 0){

freeproc(p);

release(&p -> lock);

}

p -> usyscall -> pid = p -> pid;

// An empty user page table.

p->pagetable = proc_pagetable(p);

	if(p->pagetable == 0){
	freeproc(p);
	release(&p->lock);
	return 0;
	}
	······
}
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTYyOTE3NTI3MywtMTQzODkxNjcyMywxMT
YxMDM4MTIyLC0xMTgzNDQsLTIxNDMxODIzMTgsMjYwOTcxNzMs
LTE5MjQwOTE1MDIsLTg2Nzg0MTcxMSw0MDUyMzY4MThdfQ==
-->