---
title: "Go Func Stack"
date: 2021-01-26T00:35:44+08:00
tags:
- go
- gdb
categories: 
- go
---

## go函数调用过程分析

对go的简单函数调用过程通过GDB调试的方式分析一下，查看一下函数的栈帧变化情况。

### 源代码

源代码：

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

// GOOS=linux GOARCH=amd64 go build -gcflags=all="-N -l" a.go
```

### gdb调试开始

1. 进入gdb调试

通过`gdb a`进入gdb调试

```bash
gdb a

(gdb) info files
Symbols from "/home/qraffa/gopkg/a".
Local exec file:
	`/home/qraffa/gopkg/a', file type elf64-x86-64.
	Entry point: 0x465860
	0x0000000000401000 - 0x0000000000467907 is .text
	0x0000000000468000 - 0x00000000004884a6 is .rodata
	0x0000000000488680 - 0x0000000000488b38 is .typelink
	0x0000000000488b38 - 0x0000000000488b40 is .itablink
	0x0000000000488b40 - 0x0000000000488b40 is .gosymtab
	0x0000000000488b40 - 0x00000000004c26bc is .gopclntab
	0x00000000004c3000 - 0x00000000004c3020 is .go.buildinfo
	0x00000000004c3020 - 0x00000000004c4a00 is .noptrdata
	0x00000000004c4a00 - 0x00000000004c6b90 is .data
	0x00000000004c6ba0 - 0x00000000004f6090 is .bss
	0x00000000004f60a0 - 0x00000000004f8ee8 is .noptrbss
	0x0000000000400f9c - 0x0000000000401000 is .note.go.buildid
```

2. main函数处设置断点，然后运行

源代码中，main函数在第9行，因此在第9行设置一个断点。

```bash
(gdb) break a.go:9
Breakpoint 1 at 0x4678c0: file /home/qraffa/gopkg/a.go, line 9.

(gdb) run 
Starting program: /home/qraffa/gopkg/a 
[New LWP 4639]
[New LWP 4640]
[New LWP 4641]

Thread 1 "a" hit Breakpoint 1, main.main () at /home/qraffa/gopkg/a.go:9
9	func main() {
```

3. 查看寄存器，BP和SP

```bash
(gdb) i register
...
rbp            0xc0000307d0	0xc0000307d0
rsp            0xc000030780	0xc000030780
...

或者

(gdb) p $rbp
$3 = (void *) 0xc0000307d0
(gdb) p $rsp
$4 = (void *) 0xc000030780
```

4. 查看寄存器，PC，可以查看到接下来会运行的指令

```bash
(gdb) x/10i $pc
=> 0x4678c0 <main.main>:	mov    %fs:0xfffffffffffffff8,%rcx
   0x4678c9 <main.main+9>:	cmp    0x10(%rcx),%rsp
   0x4678cd <main.main+13>:	jbe    0x467900 <main.main+64>
   0x4678cf <main.main+15>:	sub    $0x28,%rsp
   0x4678d3 <main.main+19>:	mov    %rbp,0x20(%rsp)
   0x4678d8 <main.main+24>:	lea    0x20(%rsp),%rbp
   0x4678dd <main.main+29>:	movq   $0x1,(%rsp)
   0x4678e5 <main.main+37>:	movq   $0x2,0x8(%rsp)
   0x4678ee <main.main+46>:	callq  0x467860 <main.add>
   0x4678f3 <main.main+51>:	mov    0x20(%rsp),%rbp
```

5. 逐条执行

```bash
(gdb) nexti
0x00000000004678c9	9	func main() {
(gdb) nexti
0x00000000004678cd	9	func main() {
```

6. 执行到0x4678d3处，再次查看SP和BP

```bash
(gdb) p $rbp
$5 = (void *) 0xc0000307d0
(gdb) p $rsp
$6 = (void *) 0xc000030758
```

可以看到SP指针已经下移了（0xc000030780-0xc000030758=0x28）

7. 继续执行到0x4678dd处，查看SP和BP

```bash
(gdb) p $rsp
$8 = (void *) 0xc000030758
(gdb) p $rbp
$9 = (void *) 0xc000030778
```

可以看到BP指针现在已经指向了0xc000030778地址，即SP+0x20

8. 继续向下执行两条，查看SP上的两个地址

```bash
(gdb) p *0xc000030758
$13 = 1
(gdb) p *0xc000030760
$14 = 2
```

可以看到两个参数入栈了

9. 继续执行，进入add函数，查看SP和BP

```bash
(gdb) p $rsp
$18 = (void *) 0xc000030750
(gdb) p $rbp
$19 = (void *) 0xc000030778
```

可以看到SP又向下移动了8个字节，这是CALL指令需要将IP入栈

10. 继续执行，查看SP和BP

```bash
(gdb) p $rsp
$22 = (void *) 0xc000030740
(gdb) p $rbp
$23 = (void *) 0xc000030778
```

可以看到SP向下移动了0x10

11. 继续执行

```bash
(gdb) p *0xc000030748
$27 = 198520
```

12. 继续执行，查看SP和BP

```bash
(gdb) p $rsp
$30 = (void *) 0xc000030740
(gdb) p $rbp
$31 = (void *) 0xc000030748
```

可以看到BP已经指向了SP+8

13. 接下来两条是返回值地址清空，执行到0x467880处

```bash
(gdb) p *0xc000030740
$35 = 3
```

本地变量c=3入栈，然后是该变量+1

14. 接下来是计算返回值a+b和b-a的

```bash
0x467890 <main.add+48>:	mov    0x18(%rsp),%rax
```

查看rax寄存器

```bash
(gdb) p $rax
$40 = 1
```

后边同理

最后到0x4678a9处，查看两个返回值地址的值

```bash
(gdb) p *0xc000030770
$50 = 1
(gdb) p *0xc000030768
$51 = 3
```

看到返回值已经计算完，放入main栈帧对应的位置上了

15. add函数收尾处理，查看一下当前状态

```bash
(gdb) p $rsp
$54 = (void *) 0xc000030740
(gdb) p $rbp
$55 = (void *) 0xc000030748
(gdb) x/5i $pc
=> 0x4678ae <main.add+78>:	mov    0x8(%rsp),%rbp
   0x4678b3 <main.add+83>:	add    $0x10,%rsp
   0x4678b7 <main.add+87>:	retq   
   0x4678b8:	int3   
   0x4678b9:	int3   
```

16. 继续执行，查看BP指针

```bash
(gdb) p $rbp
$58 = (void *) 0xc000030778
```

可以看到BP指针已经恢复到main栈帧的状态了

17. 继续执行，查看SP指针

```bash
(gdb) p $rsp
$59 = (void *) 0xc000030750
```

SP指针也恢复到了call指令执行后的状态了

18. 接下来是retq指令的执行，执行后查看SP和BP指针

```bash
(gdb) p $rsp
$64 = (void *) 0xc000030758
(gdb) p $rbp
$65 = (void *) 0xc000030778
```

可以看到SP上移了8字节，这是RET指令将之前CALL指令入栈的IP出栈了

19. 接下来是main函数的收尾处理

```bash
(gdb) p $rsp
$69 = (void *) 0xc000030780
(gdb) p $rbp
$70 = (void *) 0xc0000307d0
```

已经恢复到了初始状态了

接下来SP指针同样需要上移8字节，但由于我们是在main函数入口处打的断点，因此，在断点时查看的SP指针已经是由于call指令下移了8个字节

20. 到这里后面都是go其他处理了，暂不用关心，main函数和里面的add函数整个调用过程已经结束了。

### 作图分析

![stack](http://qiniu.qraffa.cn/stack.svg)

- 1~4对应图1的初始状态
- 5~8对应图2的main栈帧在调用add函数之前的一些操作，比如BP，SP指针修改，参数入栈等
- 9对应图3的CALL指令，会将IP入栈，因此SP会再次下移8个字节
- 10~13对应图4，add栈帧在计算函数返回值之前的移动指针，本地变量入栈，清零返回值的操作
- 14对应图5，add函数计算函数返回值，并返回main栈帧中的对应位置
- 15~17对应图6，add函数完成，将BP，SP指针置于函数调用前的状态
- 18对应图7的RET指令，将IP处栈，因此SP会上移8个字节
- 19~20对应图8，main函数完成，将BP，SP指针置于函数调用前的状态