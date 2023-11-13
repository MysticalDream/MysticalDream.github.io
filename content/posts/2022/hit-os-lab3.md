---
title: "哈工大操作系统 实验楼实验三 系统调用"
date: 2022-12-11T00:00:00+08:00
draft: false
# summary: "博客的概述" # 文章简单描述，会展示在主页
categories:
- 操作系统

tags:
- linux
- 操作系统

---

# 实验楼实验三 系统调用

{{<sitesrc>}}

## 操作系统实现系统调用的基本过程

- 应用程序调用库函数（API）；
- API 将系统调用号存入 EAX，然后通过中断调用使系统进入内核态；
- 内核中的中断处理函数根据EAX中的系统调用号，调用对应的内核函数（系统调用）；
>函数（API）展开含有`int指令`的代码,此时CPL=3,0x80处的中断描述符（系统门）的DPL也为3，改变cs:ip(cp)[CPL变成0]进入内核的系统调用代码处
- 系统调用完成相应功能，将返回值存入 EAX，返回到- 中断处理函数；
- 中断处理函数返回到 API 中；
- API 将 EAX 返回给应用程序。

linux0.11的系统调用（3个参数）的源码

![](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets/202311122306844.png)
>这里还有一个点需要注意一下，实验要求添加的两个系统调用在满足条件时返回`拷贝的字符数`，不满足条件时`返回 “-1”，并置 errno 为 EINVAL`，从上面这个系统调用（3个参数）源码中可以看出来，在实现`sys_xxxx`时只需要返回`-EINVAL`即可，这里会对返回值进行判断，若`__res>=0`则返回`__res`，反之则置`errno=-__res`，并返回-1。

记录过程:
>lib/xxx.c->include/unistd.h->kernel/system_call.s->include/linux/sys.h->(sys_xxx的实现)

### int 0x80的系统调用实现

先是`init/main.c`下，内核初始化时调用了`sched_init()`

```c
void main(void){
//...
tty_init();
ime_init();
sched_init();
buffer_init(buffer_memory_end);
//...
}
```
在`kernel/sched.c`中,调用了`set_system_gate`
```c
void sched_init(void){
//...
outb(inb_p(0x21)&~0x01,0x21);
set_system_gate(0x80,&system_call);
//...
}
```
`set_system_gate` 是个宏，在 `include/asm/system.h` 中定义为：

```c
//...
#define set_system_gate(n,addr) \
        _set_gate(&idt[n],15,3,addr)
//...
```
同时该头文件下`_set_gate` 的定义是：

```c
//...
#define _set_gate(gate_addr,type,dpl,addr) \
__asm__ ("movw %%dx,%%ax\n\t" \
        "movw %0,%%dx\n\t" \
        "movl %%eax,%1\n\t" \
        "movl %%edx,%2" \
        : \
        : "i" ((short) (0x8000+(dpl<<13)+(type<<8))), \
        "o" (*((char *) (gate_addr))), \
        "o" (*(4+(char *) (gate_addr))), \
        "d" ((char *) (addr)),"a" (0x00080000))
//...
```
`_set_gate`主要就是填充IDT（中断描述符表），将 `system_call` 函数`地址`写到 `0x80` 对应的中断描述符中，也就是在中断 `0x80` 发生后，自动调用函数 `system_call`。

 `system_call`定义在`kernel/system_call.s`

 ```unix-assembly

 # offsets within sigaction
sa_handler = 0
sa_mask = 4
sa_flags = 8
sa_restorer = 12

!这个系统调用的总数。
!添加删除时需要对这个数字进行相应更改
!比如这次实验中添加的两个系统调用就需要在此处修改，
!不然会导致程序没有达到预期效果（可能编译运行一切正常，却没有调用对应函数）
nr_system_calls = 72

/*
 * Ok, I get parallel printer interrupts while using the floppy for some
 * strange reason. Urgel. Now I just ignore them.
 */

.globl system_call,sys_fork,timer_interrupt,sys_execve
.globl hd_interrupt,floppy_interrupt,parallel_interrupt
.globl device_not_available, coprocessor_error

.align 2
bad_sys_call:
        movl $-1,%eax
        iret
.align 2
reschedule:
        pushl $ret_from_sys_call
        jmp schedule
.align 2

 system_call:
        cmpl $nr_system_calls-1,%eax
        ja bad_sys_call
        push %ds
        push %es
        push %fs
        pushl %edx
        pushl %ecx              # push %ebx,%ecx,%edx as parameters
        pushl %ebx              # to the system call
        movl $0x10,%edx         # set up ds,es to kernel space
        mov %dx,%ds
        mov %dx,%es
        movl $0x17,%edx         # fs points to local data space
        mov %dx,%fs
        call sys_call_table(,%eax,4)
        pushl %eax
        movl current,%eax
        cmpl $0,state(%eax)             # state
        jne reschedule
        cmpl $0,counter(%eax)           # counter
        je reschedule
 ```

 此处我们只需要关注`call sys_call_table(,%eax,4)`这一句即可，前面的压栈和后面的一些调度相关的先不关心。

`call sys_call_table(,%eax,4)`实际上就是`call sys_call_table + 4 * %eax`，其中 `eax`就是之前设置的系统调用号，即 `__NR_xxxxxx`(定义在`include/unistd.h`)。

其中`sys_call_table` 是一个函数指针数组的起始地址，它定义在 `include/linux/sys.h` 中

```c
// include/linux/sys.h
//...
fn_ptr sys_call_table[] = { sys_setup, sys_exit, sys_fork, sys_read,
sys_write, sys_open, sys_close, sys_waitpid, sys_creat, sys_link,
sys_unlink, sys_execve, sys_chdir, sys_time, sys_mknod, sys_chmod,
sys_chown, sys_break, sys_stat, sys_lseek, sys_getpid, sys_mount,
sys_umount, sys_setuid, sys_getuid, sys_stime, sys_ptrace, sys_alarm,
sys_fstat, sys_pause, sys_utime, sys_stty, sys_gtty, sys_access,
sys_nice, sys_ftime, sys_sync, sys_kill, sys_rename, sys_mkdir,
sys_rmdir, sys_dup, sys_pipe, sys_times, sys_prof, sys_brk, sys_setgid,
sys_getgid, sys_signal, sys_geteuid, sys_getegid, sys_acct, sys_phys,
sys_lock, sys_ioctl, sys_fcntl, sys_mpx, sys_setpgid, sys_ulimit,
sys_uname, sys_umask, sys_chroot, sys_ustat, sys_dup2, sys_getppid,
sys_getpgrp, sys_setsid, sys_sigaction, sys_sgetmask, sys_ssetmask,
sys_setreuid,sys_setregid };

// include/linux/sched.h
typedef int (*fn_ptr)();
```
## 添加iam和whoami两个系统调用

### 在 `include/linux/sys.h`文件下作如下修改
```c
//新增这两个函数的声明
extern int sys_iam();
extern int sys_whoami();

//在函数指针数组末尾加上新增加的两个函数
fn_ptr sys_call_table[] = { sys_setup, sys_exit, sys_fork, sys_read,
sys_write, sys_open, sys_close, sys_waitpid, sys_creat, sys_link,
sys_unlink, sys_execve, sys_chdir, sys_time, sys_mknod, sys_chmod,
sys_chown, sys_break, sys_stat, sys_lseek, sys_getpid, sys_mount,
sys_umount, sys_setuid, sys_getuid, sys_stime, sys_ptrace, sys_alarm,
sys_fstat, sys_pause, sys_utime, sys_stty, sys_gtty, sys_access,
sys_nice, sys_ftime, sys_sync, sys_kill, sys_rename, sys_mkdir,
sys_rmdir, sys_dup, sys_pipe, sys_times, sys_prof, sys_brk, sys_setgid,
sys_getgid, sys_signal, sys_geteuid, sys_getegid, sys_acct, sys_phys,
sys_lock, sys_ioctl, sys_fcntl, sys_mpx, sys_setpgid, sys_ulimit,
sys_uname, sys_umask, sys_chroot, sys_ustat, sys_dup2, sys_getppid,
sys_getpgrp, sys_setsid, sys_sigaction, sys_sgetmask, sys_ssetmask,
sys_setreuid,sys_setregid,sys_iam,sys_whoami };

```
>注意添加了这两个系统调用后需要到`kernel/system_call.s`下修改`nr_system_calls=72`为`nr_system_calls=74`

### 内核中实现函数 sys_iam() 和 sys_whoami()
我们在`kernel`目录下新增`who.c`文件，实现这两个函数即可
,具体代码如下:

```c
#include <errno.h>
#include <string.h>
#include <asm/segment.h>//用户态和内核态数据传递
#include <linux/kernel.h>//'kernel.h' 包含一些常用的函数原型等

#ifndef NULL
#define NULL ((void*)0)
#endif

#define LEN 24

char inner[LEN]={0};
int inner_len=0;

int sys_iam(const char *name){
	int i;
	char temp[LEN];
	if(name!=NULL){
		for(i=0;i<LEN;i++){
			temp[i]=get_fs_byte(name+i);

			if(temp[i]=='\0'){
				break;
			}

		}


		if(i<LEN)
		{
			strcpy(inner,temp);
			inner_len=i;
			return inner_len;
		}
	}
	return -EINVAL;
}


int sys_whoami(char *name,unsigned int size){
	int i=0;
	if(size<inner_len+1){
		return -EINVAL;
	
	}

	for(i=0;i<=inner_len;i++){
		put_fs_byte(inner[i],name+i);
	}

	return inner_len;
}
```
### 修改 Makefile

我们还需要修改`Makefile`文件使得我们新增加的文件可以被编译链接

我们要修改`kernel/Makefile`文件的两处地方：

1. 
```makefile
OBJS  = sched.o system_call.o traps.o asm.o fork.o \
        panic.o printk.o vsprintf.o sys.o exit.o \
        signal.o mktime.o
```
改成:
```makefile
OBJS  = sched.o system_call.o traps.o asm.o fork.o \
        panic.o printk.o vsprintf.o sys.o exit.o \
        signal.o mktime.o who.o
```

2.

```makefile
exit.s exit.o: exit.c ../include/errno.h ../include/signal.h \
  ../include/sys/types.h ../include/sys/wait.h ../include/linux/sched.h \
  ../include/linux/head.h ../include/linux/fs.h ../include/linux/mm.h \
  ../include/linux/kernel.h ../include/linux/tty.h ../include/termios.h \
  ../include/asm/segment.h

```
改成
```makefile
who.s who.o: who.c ../include/linux/kernel.h ../include/unistd.h
exit.s exit.o: exit.c ../include/errno.h ../include/signal.h \
  ../include/sys/types.h ../include/sys/wait.h ../include/linux/sched.h \
  ../include/linux/head.h ../include/linux/fs.h ../include/linux/mm.h \
  ../include/linux/kernel.h ../include/linux/tty.h ../include/termios.h \
  ../include/asm/segment.h

```

### 编译linux源码

修改后需要进行编译
切换到`linux0.11`目录下执行以下命令

```bash
# make clean
# make all
```
一切顺利的话，会在当前目录下生成`Image`镜像文件


![](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets/202311122306900.png)
 


### 挂载文件系统镜像进行修改

在 `oslab`目录下，运行

```bash
# sudo ./mount-hdc 
```

之后切换到`oslab`目录下的`hdc`目录下，该目录下就是挂载的文件系统，也就是`linux0.11`使用的文件系统。

切换到`hdc`后

#### 修改unistd.h
我们需要在`usr/include/unistd.h`头文件中添加

```c
//...
#define __NR_setsid     66
#define __NR_sigaction  67
#define __NR_sgetmask   68
#define __NR_ssetmask   69
#define __NR_setreuid   70
#define __NR_setregid   71
//添加如下两个宏
#define __NR_iam        72
#define __NR_whoami     73
```

之后我们切换到`usr/root`目录下

#### 添加程序验证系统调用
新增两个文件`iam.c`和`whoami.c`

`iam.c`内容如下：
```c
#define __LIBRARY__

#include<stdio.h>
#include<unistd.h>


_syscall1(int,iam,const char*,name);

int main(int argc,char* argv[])
{
        int l;
        l=iam(argv[1]);
        if(l!=-1){
                printf("saved!\n");
        }else{
                printf("Failed to save!\nPlease check if the length of the input name is less than or equal to 23.\n");
        }
        return 0;
}

```

`whoami.c`内容如下：
```c
#define __LIBRARY__

#include<unistd.h>
#include<stdio.h>

_syscall2(int,whoami,char*,name ,unsigned int,size);


int main(int argc,char *argv){
        char name[24];
        whoami(name,24);
        printf("%s\n",name);
        return 0;
}
```

* 以上两个文件开头为什么要添加宏`#define __LIBRARY__`？
>我们可以打开`unistd.h`文件，其中的定义系统调用的位置的地方有`ifdef __LIBRARY__`，所以我们需要在使用这个`unistd.h`中的这部分宏定义的地方前添加`#define __LIBRARY`宏，以使得宏定义生效

![](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets/202311122307528.png)

弄完后记得卸载这个文件系统

```bash
# sudo umount hdc
```

### 运行bochs验证系统调用

首先在`oslab`目录下，执行以下命令

```bash
# ./run
```
运行截图

![](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets/202311122307022.png)



![](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets/202311122307934.png)


在`/usr/root`目录下，执行以下命令
```bash
# gcc iam.c -o iam

# gcc whoami.c -o whoami

# iam mysticaldream
saved!

# whoami
mysticaldeam

```
 

![](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets/202311122307759.png)

## 问题回答

- 从 Linux 0.11 现在的机制看，它的系统调用最多能传递几个参数？你能想出办法来扩大这个限制吗？
>答:最多传递三个参数。`linux 0.11`通过`bx`、`cx`、`dx`寄存器传递（`ax`作为系统调用号），这种方式受限于通用寄存器的数量。
解决办法：通过使用一个寄存器保存指向进程的用户态栈中的一块内存区域的地址，该内存中保存参数的值。

- 用文字简要描述向 Linux 0.11 添加一个系统调用 foo() 的步骤。
>答:
>- 需在`hdc/usr/include/unistd.h`中添加`__NR_foo`对应的索引下标。
>- 首先可以在`kernel`目录下编写`sys_foo.c`，里面包含系统调用的实现。
>- 还需要在`include/linux/sys.h`文件中添加`sys_foo`函数声明和添加到系统调用指针数组
>- 并修改`kernel/system_call.s`中的`nr_system_calls`（系统调用总数），即原来的值加1
>- 然后需要修改`kernel/Makefile`
>- 编译源码生成系统镜像文件
>- 之后可以在系统类库中编写`foo.c`，调用系统调用`sys_foo`来实现系统调用库函数

## 文章参考:
[操作系统原理与实践](https://www.lanqiao.cn/courses/115)


