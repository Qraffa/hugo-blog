---
title: "Go Stack"
date: 2021-01-26T00:35:51+08:00
tags:
- go
categories: 
- go
---

## go 栈

### 操作系统进程内存

#### 虚拟内存空间大小

Linux为每个进程维护了一个单独的虚拟地址空间。在32位系统上，该虚拟地址空间大小为2^32Bytes = 4G，其中内核空间1G，在高地址处，用户空间3G，在低地址处；在64位系统上，该虚拟地址空间大小为2^48Bytes = 256TB，其中用户和内核各128TB，用户空间0x0000000000000000~0x00007fffffffffff，内核空间0xffff800000000000~0xffffffffffffffff，中间16M TB暂未用到。

> linux查看cpu可访问地址空间位数
>
> ```bash
> cat /proc/cpuinfo | grep address
> address sizes	: 43 bits physical, 48 bits virtual
> ```
>
> `48 bits virtual`表示可访问地址空间位数为48，即256TB。

#### 进程的虚拟内存空间分布

```bash
高地址+---------> +------------------+-----------+
                  |                  |           |
                  | 进程相关数据结构 |           |
                  |                  |           |
                  +------------------+           v
                  |                  |
                  |     物理内存     |         内核区
                  |                  |
                  +------------------+           ^
                  |                  |           |
                  |  内核代码和数据  |           |
                  |                  |           |
                  +------------------------------+
                  |XXXXXXXXXXXXXXXXXX|
                  +------------------------------+
                  |                  |           |
                  |      用户栈      |           |
                  |                  |           |
 %rsp  +--------> +--------+---------+           |
                  |        |         |           |
                  +--------v---------+           |
                  |                  |           |
                  |共享区内存映射区域|           |
                  |                  |           |
                  +--------^---------+           |
                  |        |         |           |
 brk   +--------> +--------+---------+           v
                  |                  |
                  |     运行时堆     |         用户区
                  |                  |
                  +------------------+           ^
                  |                  |           |
                  |   未初始化数据   |           |
                  |                  |           |
                  +------------------+           |
                  |                  |           |
                  |   已初始化数据   |           |
                  |                  |           |
                  +------------------+           |
                  |                  |           |
                  |       代码       |           |
                  |                  |           |
                  +------------------+           |
                  |XXXXXXXXXXXXXXXXXX|           |
低地址+---------> +------------------------------>

```

64位代码段总是从地址0x400000开始的。

#### 用户态和内核态

处理器通常用某个控制寄存器中的一个模式位，来描述进程当前享有的特权。

当设置了该模式位后，进程就运行在内核模式中，在内核模式中的进程可以执行指令集中的任何指令，并且可以访问到系统中任何内存的位置。

在用户模式中的进程，不允许执行特权执行，比如发起IO操作，也不能直接引用地址空间中内核区的代码和数据。用户模式中的进程必须通过系统调用接口切换到内核模式，才能执行特权指令和访问内核代码和数据。

程序默认是在用户模式下进行的，想要切换到内核模式，只能通过`中断`、`故障`、`陷入系统调用`异常。这些异常发生时，控制传递到异常处理程序，处理器将切换到内核模式，当返回到用户程序代码后，切换回用户模式。

对于系统调用，执行syscall指令，相当于对程序有意的触发一次异常，为了处理该异常，就需要切换到内核模式，调用异常处理程序，然后调用对应的内核程序，完成系统调用后返回到用户模式。

### 线程栈

Linux2.6以后，对于线程的实现是NPTL。内核将所有的线程都当做进程来看待，因此进程和线程的描述结构体都是`task_struct`，线程与其他进程的其他在于，线程被视为一个与其他进程共享某些资源的进程。

对于创建一个进程`pthread_create`，内部使用`clone`，与父进程共享地址空间、文件等资源，并且在`task_struct`中每个线程的`pid`都是唯一的，但对于一个线程组，他们的`tgid`都是相同的。比如现在一个进程id为123，那么该进程的`task_struct`中pid与tgid都是123，因为该进程可以理解为该线程组的leader；现在该进程创建了一个线程，那么该线程的`task_struct`中pid为124，但tgid为123，表示他是线程组中的一个。

线程之间的地址空间是共享的，因此线程组中的线程的task_strcut的mm字段都会指向同一块内存区域。

每个线程都应该有自己独立的用户栈空间和内核栈空间，线程的内核栈空间是通过`mmap`在堆上分配的，默认大小为8M。

内核栈，当进程陷入系统调用时，内核代码使用的栈空间并不是进程的用户栈空间，需要一个单独的内核栈空间。

> x86_64 has a kernel stack for every active thread
>
> 线程栈通过mmap分配在堆区域上，
>
> > and the pthread library uses anonymous mapped regions as stacks for new threads.

### Go函数调用

`CALL`指令：将IP入栈(PUSH)，然后修改IP，跳转被调函数的位置

​	我们知道程序计数器IP，是指向下一条指令的地址的。因此在调用CALL时，此时的IP就是指向了CALL的下一条指令。

`RET`指令：将IP出栈(POP)，此值作为IP

​	此时栈顶存储的是IP，即CALL指令的下一条指令，因此将此值POP出来作为IP即可，继续回到之前的流程继续执行

以下为例，Go函数调用过程

```go
package main

func add(a, b int) (int, int) {
	c := 3
	c = c + 1
	return a + b, b - a
}

func main() {
	add(1, 2)
}
```

编译汇编代码

```bash
GOOS=linux GOARCH=amd64 go tool compile -S -N -l a.go
```

去掉一些无关信息.

```assembly
"".add STEXT nosplit size=88 args=0x20 locals=0x10
	0x0000 00000 (a.go:3)	TEXT	"".add(SB), NOSPLIT|ABIInternal, $16-32
	0x0000 00000 (a.go:3)	SUBQ	$16, SP	// SP指针下移16字节，相当于分配16字节的栈空间
	0x0004 00004 (a.go:3)	MOVQ	BP, 8(SP)	// 将BP寄存器中的值保存到SP+8地址上
	0x0009 00009 (a.go:3)	LEAQ	8(SP), BP	// 将BP寄存器中的值改为SP+8的地址
	...
	0x000e 00014 (a.go:3)	MOVQ	$0, "".~r2+40(SP)	// 将SP+40地址的值清空
	0x0017 00023 (a.go:3)	MOVQ	$0, "".~r3+48(SP)	// 将SP+48地址的值清空
	0x0020 00032 (a.go:4)	MOVQ	$3, "".c(SP)	// 该函数的本地变量入栈，放在SP+0地址上
	0x0028 00040 (a.go:5)	MOVQ	$4, "".c(SP)	// c=c+1
	0x0030 00048 (a.go:6)	MOVQ	"".a+24(SP), AX	// 将函数参数a=1，写入AX寄存器中
	0x0035 00053 (a.go:6)	ADDQ	"".b+32(SP), AX	// 将函数参数b=2，与AX寄存器值做加法，AX=AX+b，写入AX中
	0x003a 00058 (a.go:6)	MOVQ	AX, "".~r2+40(SP)	// AX寄存器的值为3，写入SP+40地址上
	0x003f 00063 (a.go:6)	MOVQ	"".b+32(SP), AX	// 将函数参数b=2，写入AX寄存器中
	0x0044 00068 (a.go:6)	SUBQ	"".a+24(SP), AX	// 将函数参数a=1，与AX寄存器值做减法，AX=AX-a，写入AX中
	0x0049 00073 (a.go:6)	MOVQ	AX, "".~r3+48(SP)	// AX寄存器的值为1，写入SP+48地址上
	0x004e 00078 (a.go:6)	MOVQ	8(SP), BP	// 恢复SP+8地址上保存的上一个函数栈帧的BP地址到BP寄存器中
	0x0053 00083 (a.go:6)	ADDQ	$16, SP	// SP上移16字节，销毁栈空间
	0x0057 00087 (a.go:6)	RET	// 函数返回，POP出IP，继续执行上一个函数
"".main STEXT size=71 args=0x0 locals=0x28
	0x0000 00000 (a.go:9)	TEXT	"".main(SB), ABIInternal, $40-0
	...
	0x000f 00015 (a.go:9)	SUBQ	$40, SP // SP指针下移40字节，相当于分配40字节的栈空间
	0x0013 00019 (a.go:9)	MOVQ	BP, 32(SP)	// 将BP寄存器中的值保存到SP+32栈地址上
	0x0018 00024 (a.go:9)	LEAQ	32(SP), BP	// 将BP寄存器的值改为SP+32的地址
	...
	0x001d 00029 (a.go:10)	MOVQ	$1, (SP)	// 参数a=1入栈，放在SP+0地址上
	0x0025 00037 (a.go:10)	MOVQ	$2, 8(SP)	// 参数b=2入栈，放在SP+8地址上
	...
	0x002e 00046 (a.go:10)	CALL	"".add(SB)	// CALL调用add函数，PUSH进IP，JMP跳转到目标位置
	0x0033 00051 (a.go:11)	MOVQ	32(SP), BP	// 恢复SP+32地址上保存的上一个函数栈帧的BP地址到BP寄存器中
	0x0038 00056 (a.go:11)	ADDQ	$40, SP	// SP上移40字节，销毁栈空间
	0x003c 00060 (a.go:11)	RET	// 函数返回，POP出IP，继续执行上一个函数
```

### Go栈管理

#### goroutine栈分配

```go
go func()
```

当我们使用go创建goroutine时，通过`newproc`->`newproc1`函数调用，在`newproc1`函数中，如果获取不到闲置的G，就会通过`malg`函数来创建一个新的G，该函数中，通过`stackalloc`函数来为新创建的G分配栈空间。

- 在`stackalloc`函数前，会对需要申请的栈空间向上取2^x整
- 如果申请的栈空间小于32K，则属于小栈空间分配；如果申请的栈空间大于32K，则是大栈空间分配。
- 分配完成后，设置G的`stackguard0 = newg.stack.lo + _StackGuard`，用于栈溢出检测

**小栈空间分配**

1. 计算当前栈空间大小对应的位置order
2. 通过`stackpoolalloc`函数，尝试从全局栈缓存`stackpool`链表中分配
   1. 如果此时`stackpool`中对应大小的缓存空间不足，则需要通过`mheap_.allocManual`来分配新内存
3. 如果当前M被标记为抢占，则需要先尝试从当前P的mcache中获取对应大小的栈空间
   1. 如果当前P的mcache的`stackcache`中获取不到，则调用`stackcacherefill`
   2. `stackcacherefill`函数会调用`stackpoolalloc`，相当于从全局的栈缓存`stackpool`中获取`_StackCacheSize/2 = 16K`的缓存放到当前P的mcache的`stackcache`中

**大栈空间分配**

1. 尝试从全局大栈缓存`stackLarge`获取内存
2. 获取失败则需要通过`mheap_.allocManual`来分配新内存

**堆上内存分配**

小栈空间参数为`_StackCacheSize>>_PageShift = 4`，大栈空间参数为`申请内存大小右移13位，则以2的底对数`

- `allocManual`调用`allocSpan`

待做Go内存分配再写......

#### 栈扩容

**扩容发生时机**

栈扩容发生在`morestack `函数，调用`newstack`函数

**栈溢出检测**

```assembly
0x0000 00000 (a.go:9)	MOVQ	(TLS), CX
0x0009 00009 (a.go:9)	CMPQ	SP, 16(CX)
0x000d 00013 (a.go:9)	JLS	64
...
0x0040 00064 (a.go:9)	CALL	runtime.morestack_noctxt(SB)
```

- 获取当前G

> `TLS`也是一个伪寄存器，表示的是`thread-local storage`，它存放了`g`结构体。并且只能被载入到另一个寄存器中。

- 通过SP指针与G的`stackguard0`比较来检测是否需要扩容

```go
type g struct {
   stack       stack   // 
   stackguard0 uintptr // stackguard0 = g.stack.lo + _StackGuard
   ...
}
type stack struct {
	lo uintptr
	hi uintptr
}
```

因此G结构体的指针后移16恰好的stackguard0的地址。

**栈扩容**

- 栈扩容时，会扩容为原空间的两倍大小

> 这里有个优化点是会在计算新大小时，算出当前需要的合适大小的newsize，避免了recheck。
>
> 例如当前栈大小为2K，需要的栈大小为10K，则newsize会计算成4K->8K->16K

- 如果申请的栈空间溢出，则直接异常
- 调用`copystack`进行栈复制
- copy结束后，`gogo`恢复当前G继续执行

**栈复制**

- 调用`stackalloc`为新栈空间分配内存
- 调整`sudog`结构体的指针（sudog与G的阻塞相关，比如channel阻塞的读写）
- `memmove`将旧stack内存移动到新stack地址上
- `adjustctxt`，`adjustdefers`，`adjustpanics`调整G的其他指针到对应位置
- 调整当前G的stack为新的stack
- `gentraceback`调整指针（没懂）
- 释放旧stack

#### 栈缩容

**栈缩容发生时机**

栈的收缩发生在 GC 时对栈进行扫描的阶段，调用`shrinkstack`函数

**栈缩容函数**

- 栈缩容时，新空间为原空间的一半
- 如果新空间小于stack最小大小，则不缩容
- 计算栈已使用空间`gp.stack.hi - gp.sched.sp + _StackLimit`，如果使用空间小于栈空间的1/4，则缩容


### 参考

- [https://www.kernel.org/doc/html/latest/x86/x86_64/mm.html](https://www.kernel.org/doc/html/latest/x86/x86_64/mm.html)
- [线程栈](https://gist.github.com/CMCDragonkai/10ab53654b2aa6ce55c11cfc5b2432a4)
- [线程栈](https://sploitfun.wordpress.com/2015/02/10/understanding-glibc-malloc/)
- [CALL和RET](https://mhy12345.xyz/technology/assembly-ret-call/)
- [CALL和RET](https://rj45mp.github.io/ret%E6%8C%87%E4%BB%A4%E4%B8%8Ecall%E6%8C%87%E4%BB%A4%E7%9A%84%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3/)
- [深入研究goroutine栈](http://www.huamo.online/2019/06/25/%E6%B7%B1%E5%85%A5%E7%A0%94%E7%A9%B6goroutine%E6%A0%88/)
- [函数调用](https://draveness.me/golang/docs/part2-foundation/ch04-basic/golang-function-call/)
- [Go 函数调用 ━ 栈和寄存器视角](https://segmentfault.com/a/1190000019753885)