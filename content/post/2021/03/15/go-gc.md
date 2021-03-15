---
title: "Go Gc"
date: 2021-03-15T23:11:50+08:00
url: go-gc
tags:
- go
categories: 
- go
---

## GC

版本迭代

- v1.1 STW
- v1.3 Mark STW
- v1.5 三色标记
- v1.8 混合写屏障

三色标记

- 不变性条件，写屏障
- 如何标记，gcmarkerbits，0表示白色对象，1表示灰色或黑色对象。p中wbBuf和gcw以及全局workbuf来表示标记队列
- 对象元信息，查找到对象分配内存的span

gc触发时机

- 分配内存时，当前已分配内存与上次内存比较
- sysmon每2min检查执行一次
- 调用runtime.gc强制执行

### 三色标记

传统的标记-清除需要长时间STW以完成标记和清扫的过程，三色标记用于改进减小STW的时间。

三色标记中将对象分为三种类型：

- 白色：可能存活的对象，在初始阶段所有对象为白色，在标记完成后，所有的白色对象视为垃圾
- 灰色：确认存活的对象，但其引用了白色对象，因此要对灰色对象进行递归扫描
- 黑色：确认存活的对象，扫描完成的对象，根对象可达

三色标记过程：

1. 初始时所有对象为白色
2. 将根对象标记为灰色，根对象包括运行栈中的对象以及全局对象
3. 对所有灰色对象进行扫描，将灰色对象引用的对象标记为灰色，并将该灰色对象标记为黑色
4. 重复上述过程，直到没有灰色对象
5. 标记结束后，程序中只有黑色对象和白色对象，黑色对象为确认存活的对象，白色对象为垃圾

**三色不变性**

当三色标记的标记过程是STW时，可以确保标记过程的正确性，但STW要消耗大量时间。但如果将三色标记的标记过程和用户代码并发执行，则可能出现对象丢失.

三色标记正确性被破坏，如下图，B对象本不该回收的对象，由于引用的改变，导致其被回收了。

图来自https://golang.design/under-the-hood/zh-cn/part2runtime/ch08gc/barrier/

![](http://qiniu.qraffa.cn/gc-mutator.png)



使得三色标记正确性被破坏的两个条件：

- 黑色对象引用了白色对象
- 灰色对象达到白色对象的未访问过的路径被破坏，即黑色对象引用的白色对象，但灰色对象无法找到一条路径到达该白色对象

当上述两个条件同时满足时，三色标记就可能丢失对象。

想要使得三色标记正确，就必须破坏上述条件中的任意一个条件，因此三色标记不变式：

- 强三色不变式：黑色对象不能指向白色对象，只能指向灰色或黑色对象。该不变式破坏了两个条件。
- 弱三色不变式：黑色对象执行的白色对象，必须存在一条从灰色对象经过零个或多个白色对象可达该白色对象的路径。

#### 屏障

**Dijkstra插入写屏障**

当某一对象的引用被插入到已经标记为黑色的对象中，需要将其标记为灰色对象。

将有存活可能的对象标记为灰色，以满足强三色不变式。

```go
// Dijkstra 插入屏障
// slot表示旧指向的对象，ptr表示新指向的对象
func DijkstraWritePointer(slot *unsafe.Pointer, ptr unsafe.Pointer) {
    // 将新指向的对象标记为灰色
    shade(ptr)
    *slot = ptr
}
```

特点：

- 可能产生部分黑色对象的垃圾，需要在下一次GC中回收
- 对于栈上的对象，使用插入写屏障后耗费大量性能，因此不在栈上开启插入写屏障。
- 由于在栈上不开启插入写屏障，因此当栈上的黑色对象指向了白色的对象时，因为没有屏障，因此白色对象会被错误回收。因此插入写屏障在标记结束后，会STW并再次重新扫描栈。

**Yuasa删除写屏障**

起始时，STW将整个栈的可达对象标记为黑色，将所有可达对象在灰色保护下

当被引用的对象被删除时，如果该对象是白色，则将其标记为灰色，以满足弱三色不变式，因为该对象的下游白色对象可以从灰色对象可达。

```go
// Yuasa 屏障
// slot表示旧指向的对象，ptr表示新指向的对象
func YuasaWritePointer(slot *unsafe.Pointer, ptr unsafe.Pointer) {
    // 将被删除的就对象slot标记为灰色
    shade(*slot)
    *slot = ptr
}
```

特点：

- 不需要重新扫描栈
- 由于起始时需要STW，且扫描整个栈，因此对于栈非常多，或栈非常大的情况，耗费性能
- 部分垃圾对象可能存活到下一次GC中才能回收

**混合写屏障**

论文伪代码：

```go
writePointer(slot, ptr):
    shade(*slot)
    if current stack is grey:
        shade(ptr)
    *slot = ptr
```

实际实现伪代码：

```go
writePointer(slot, ptr):
    shade(*slot)
    shade(ptr)
    *slot = ptr
```

源码：`runtime/asm_amd64.s#L1395`，`runtime·gcWriteBarrier`方法

**流程：**

- 新创建的对象标记为黑，因此不再需要重新扫描栈
- 将删除的对象，即旧对象和指向的对象，即新对象都标记为灰

源码实现:

**新创建的对象为黑色**

调用`gcmarknewobject`将分配的对象标记为黑色，主要实现是找到对象所在内存的span位置，修改span的`gcmarkBits`

```go
// newobject -> malloc -> 
if gcphase != _GCoff {
    gcmarknewobject(span, uintptr(x), size, scanSize)
}
```

**批量写缓存标记对象颜色**

混合写屏障中的对对象的标记`shade`并不是立即生效的，而是在混合写屏障中，将需要标记的对象添加到P的`wbBuf`队列中，当该队列满时，调用`wbBufFlush`将队列中的对象写入gcwork中，此时才是真正的标记为灰色

**对象的颜色表示**

- 白色对象：`gcmarkBits`标记为0
- 灰色对象：`gcmarkBits`标记为1，且在扫描队列中
- 黑色对象：`gcmarkBits`标记为1，且不在扫描队列中

### GC流程

#### overview

- 清理终止阶段：
  - STW，将所有P到达安全点
  - 如果GC是强制触发的，则需要清理还未清理的span，只有强制触发会导致该情况发生
- 标记阶段：
  - 将gcphase置为_GCmark，开启写屏障，开启用户进程协助，将根对象入队。当所有P都开启写屏障之后，才可以开始扫描。
  - Start the world，GC标记任务将由标记进程和协助进程完成。写屏障会将被覆盖的旧对象和指向的新对象都标记灰色，并且所有新创建的对象为黑色。
  - GC开始标记根对象，扫描所有栈，全局对象，堆外数据结构。扫描一个栈时，需要暂停该栈，找到所有栈上可达对象，然后恢复。
  - GC遍历标记队列，将队列中灰色对象标记为黑色，将灰色对象执行的对象标记为灰色
  - 使用分布式终止算法，等待所有根对象标记完，没有灰色对象时，进入标记终止阶段
- 标记终止阶段：
  - STW
  - 将gcphase置为_GCmarktermination，关闭标记进程和协助进程
  - 清理mcaches缓存
- 清理阶段：
  - 将gcphase置为_GCoff，初始化清理状态，关闭写屏障
  - Start the world，新创建的对象改为白色，当必要时，分配内存操作会清理span
  - GC在后台并发清理span，当分配内存时触发清理

**并发清理阶段**

清理与用户代码并发执行。当G尝试去分配内存时会触发清理，后台G也会并发清理span。

当用户代码分配内存不足，需要获取新的span时，应该首先尝试去回收那些还未被清理的内存。

当尝试分配小对象时，会尝试去清理那些与申请内存大小相同的span，直到有空闲的内存。

当尝试分配大对象时，会尝试清理span，直到释放足够多大小的pages到堆中。



GC期间会将所有的mcache清理到central中，因此所有mcache都为空。

当G尝试去获取新的span到mcache中时，需要清理该span，当G释放span内存时，需要确保该span是被清理过的，可以是G去清理，也可以是等待并发清理去完成。

当下一次GC开始时，如果还有未清理的span，则需要清理掉。

#### gc触发时机

无论哪种触发方式，都是通过调用`gcStart`并且传递参数`gcTrigger`来检查是否需要GC

- `sysmon`唤醒`forcegchelper`定时触发GC。

  ```go
  gcStart(gcTrigger{kind: gcTriggerTime, now: nanotime()})
  ```

- 分配内存不足或者分配大于32KB的大对象时，触发GC

  ```go
  if t := (gcTrigger{kind: gcTriggerHeap}); t.test() {
      gcStart(t)
  }
  ```

- 手动调用`runtime.GC`方法触发

  ```go
  gcStart(gcTrigger{kind: gcTriggerCycle, n: n + 1})
  ```

Trigger检查方法：

```go
func (t gcTrigger) test() bool {
	if !memstats.enablegc || panicking != 0 || gcphase != _GCoff {
		return false
	}
	switch t.kind {
	// 检查当前堆内存大小是否达到设置的触发堆大小
	case gcTriggerHeap:
		return memstats.heap_live >= memstats.gc_trigger
	// 一定时间内未触发GC，则sysmon会唤醒触发，默认时间2min
	case gcTriggerTime:
		if gcpercent < 0 {
			return false
		}
		lastgc := int64(atomic.Load64(&memstats.last_gc_nanotime))
		return lastgc != 0 && t.now-lastgc > forcegcperiod
	// 如果当前不处于GC，则开启新的GC
	case gcTriggerCycle:
		return int32(t.n-work.cycles) > 0
	}
	return true
}
```

#### gc启动

```go
gcstart func:
// 对每个P启动后台标记进程
gcBgMarkStartWorkers()
// STW
systemstack(stopTheWorldWithSema)
// 根对象入队
gcMarkRootPrepare()
// start the world
startTheWorldWithSema(trace.enabled)
```

### Q

- 删除写屏障的快照标记？全局栈STW，不再re-scan

### 参考

- [https://draveness.me/golang/docs/part3-runtime/ch07-memory/golang-garbage-collector/](https://draveness.me/golang/docs/part3-runtime/ch07-memory/golang-garbage-collector/)
- [https://golang.design/under-the-hood/zh-cn/part2runtime/ch08gc/](https://golang.design/under-the-hood/zh-cn/part2runtime/ch08gc/)
- https://segmentfault.com/a/1190000022030353
- [https://liqingqiya.github.io/](https://liqingqiya.github.io/)
- [http://legendtkl.com/2017/04/28/golang-gc/](http://legendtkl.com/2017/04/28/golang-gc/)
- [https://www.cnblogs.com/zkweb/p/7880099.html](https://www.cnblogs.com/zkweb/p/7880099.html)