---
title: "Go Bootstrap"
date: 2021-01-02T23:33:16+08:00
tags:
- go
categories: 
- go
---

## go启动过程

### 找到程序入口

使用gdb调试程序找到入口

1. `gdb main`调试
2. 在gdb中使用`info files`找到程序入口`Entry point: 0x464820`
3. `b *0x464820`，在入口处设置断点，找到文件位置

<!--more-->

这里我是在windows下go build，再到linux平台运行的，因此文件位置显示的是windows下go的安装位置。根据该文件找到对应位置即可。

```bash
gdb main
(gdb) info files
Symbols from "/home/rj/main".
Local exec file:
	`/home/rj/main', file type elf64-x86-64.
	Entry point: 0x464820
	0x0000000000401000 - 0x000000000049910a is .text
	0x000000000049a000 - 0x00000000004ddd06 is .rodata
	0x00000000004ddee0 - 0x00000000004de614 is .typelink
	0x00000000004de618 - 0x00000000004de668 is .itablink
	0x00000000004de668 - 0x00000000004de668 is .gosymtab
	0x00000000004de680 - 0x000000000053da83 is .gopclntab
	0x000000000053e000 - 0x000000000053e020 is .go.buildinfo
	0x000000000053e020 - 0x000000000054c4c0 is .noptrdata
	0x000000000054c4c0 - 0x0000000000553930 is .data
	0x0000000000553940 - 0x0000000000583250 is .bss
	0x0000000000583260 - 0x00000000005860e8 is .noptrbss
	0x0000000000400f9c - 0x0000000000401000 is .note.go.buildid
(gdb) b *0x464820
Breakpoint 1 at 0x464820: file F:/Go/src/runtime/rt0_linux_amd64.s, line 8.
```

### 查看启动步骤

根据上面找到的入口文件位置

**runtime/rt0_linux_amd64.s#L7**

```assembly
TEXT _rt0_amd64_linux(SB),NOSPLIT,$-8
   JMP    _rt0_amd64(SB)
```

跳转到_rt0_amd64

**runtime/asm_amd64.s#L10**

```assembly
// _rt0_amd64 is common startup code for most amd64 systems when using
// internal linking. This is the entry point for the program from the
// kernel for an ordinary -buildmode=exe program. The stack holds the
// number of arguments and the C-style argv.
TEXT _rt0_amd64(SB),NOSPLIT,$-8
	MOVQ	0(SP), DI	// argc
	LEAQ	8(SP), SI	// argv
	JMP	runtime·rt0_go(SB)
```

将程序的启动参数放到寄存器，跳转到rt0_go

**runtime/asm_amd64.s#L87**

省略其他系统相关处理，

```assembly
LEAQ	runtime·g0(SB), CX
MOVQ	CX, g(BX)
LEAQ	runtime·m0(SB), AX
// save m->g0 = g0
MOVQ	CX, m_g0(AX)
// save m0 to g0->m
MOVQ	AX, g_m(CX)
```

对go中的g0和m0做初始化操作，并将g0和m0互相绑定起来

```assembly
CALL	runtime·args(SB)
CALL	runtime·osinit(SB)
CALL	runtime·schedinit(SB)
// create a new goroutine to start program
MOVQ	$runtime·mainPC(SB), AX		// entry
PUSHQ	AX
PUSHQ	$0			// arg size
CALL	runtime·newproc(SB)
POPQ	AX
POPQ	AX
// start this M
CALL	runtime·mstart(SB)
CALL	runtime·abort(SB)	// mstart should never return
RET
```

- 调用参数处理
- 调用`osinit`初始化，获取cpu核心数
- 调用`schedinit`初始化，设置M的最大数量，创建P
- `$runtime·mainPC`设置runtime.main入口，即main需要运行的函数
- 调用`newproc`创建一个G
- 调用`mstart`启动M

**runtime/proc.go#L114**

```go
// The main goroutine.
func main() {
	g := getg()
	// Allow newproc to start new Ms.
	mainStarted = true

	if GOARCH != "wasm" { // no threads on wasm yet, so no sysmon
		systemstack(func() {
			newm(sysmon, nil, -1)
		})
	}

	// Lock the main goroutine onto this, the main OS thread,
	// during initialization. Most programs won't care, but a few
	// do require certain calls to be made by the main thread.
	// Those can arrange for main.main to run in the main thread
	// by calling runtime.LockOSThread during initialization
	// to preserve the lock.
	lockOSThread()

	if g.m != &m0 {
		throw("runtime.main not on m0")
	}

	doInit(&runtime_inittask) // must be before defer
	if nanotime() == 0 {
		throw("nanotime returning zero")
	}

	// Defer unlock so that runtime.Goexit during init does the unlock too.
	needUnlock := true
	defer func() {
		if needUnlock {
			unlockOSThread()
		}
	}()

	// Record when the world started.
	runtimeInitTime = nanotime()

	gcenable()

	main_init_done = make(chan bool)
	if iscgo {
		if _cgo_thread_start == nil {
			throw("_cgo_thread_start missing")
		}
		if GOOS != "windows" {
			if _cgo_setenv == nil {
				throw("_cgo_setenv missing")
			}
			if _cgo_unsetenv == nil {
				throw("_cgo_unsetenv missing")
			}
		}
		if _cgo_notify_runtime_init_done == nil {
			throw("_cgo_notify_runtime_init_done missing")
		}
		// Start the template thread in case we enter Go from
		// a C-created thread and need to create a new thread.
		startTemplateThread()
		cgocall(_cgo_notify_runtime_init_done, nil)
	}

	doInit(&main_inittask)

	close(main_init_done)

	needUnlock = false
	unlockOSThread()

	if isarchive || islibrary {
		// A program compiled with -buildmode=c-archive or c-shared
		// has a main, but it is not executed.
		return
	}
	fn := main_main // make an indirect call, as the linker doesn't know the address of the main package when laying down the runtime
	fn()
	if raceenabled {
		racefini()
	}

	// Make racy client program work: if panicking on
	// another goroutine at the same time as main returns,
	// let the other goroutine finish printing the panic trace.
	// Once it does, it will exit. See issues 3934 and 20018.
	if atomic.Load(&runningPanicDefers) != 0 {
		// Running deferred functions should not take long.
		for c := 0; c < 1000; c++ {
			if atomic.Load(&runningPanicDefers) == 0 {
				break
			}
			Gosched()
		}
	}
	if atomic.Load(&panicking) != 0 {
		gopark(nil, nil, waitReasonPanicWait, traceEvGoStop, 1)
	}

	exit(0)
	for {
		var x *int32
		*x = 0
	}
}
```

**runtime/proc.go#L113**

```go
// The main goroutine.
func main() {
	g := getg()
	// 设置main为已启动，以此来运行启动新的M
	mainStarted = true
	// 启动sysmon后台线程，该线程不需要绑定P
	if GOARCH != "wasm" { // no threads on wasm yet, so no sysmon
		systemstack(func() {
			newm(sysmon, nil, -1)
		})
	}
	// main goroutine必须运行在m0上
	if g.m != &m0 {
		throw("runtime.main not on m0")
	}
	// 执行runtime init
	doInit(&runtime_inittask) // must be before defer
	// gc启动
	gcenable()
    // 执行用户的main.init(即运行main函数之前的那些init操作)
	doInit(&main_inittask)
    // 执行用户的main.main(运行main函数)
	fn := main_main // make an indirect call, as the linker doesn't know the address of the main package when laying down the runtime
	fn()
	// 退出程序
	exit(0)
	// 访问非法地址，保证程序一定能够退出(被系统杀死)
	for {
		var x *int32
		*x = 0
	}
}
```

### 参考

[cch123-Bootstrap](https://github.com/cch123/golang-notes/blob/master/bootstrap.md)