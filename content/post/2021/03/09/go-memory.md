---
title: "Go Memory"
date: 2021-03-09T01:07:35+08:00
url: post/2021/03/09/go-memory
tags:
- go
categories: 
- go
---

## TCMalloc

给每个线程都分配一个局部缓存，采用线程局部数据技术。小块内存的分配从线程局部缓存分配即可满足。同时周期性地将线程缓存内存回收到中心缓存中，线程局部缓存不足时从中心缓存中申请。

#### 小对象分配(<=256K)

对于256K以内的内存大小，划分了约88个种类，每个size class对应一个种类，分配时是将申请内存大小向上取整到size class类别的大小，比如申请5bytes会分配8bytes，由于分配的内存会大于申请的内存，则会造成内存浪费。

- 每个线程分配单独的缓存ThreadCache，ThreadCache中对于每个size class都分配一个FreeList用于分配和回收该大小的内存空间。小对象的分配直接在ThreadCache中对应大小的FreeList中获取空闲对象
- 全局共享缓存CentralCache，对于每个size class都分配一个CentralFreeList，当ThreadCache尝试分配发现没有空闲对象时，就从CentralCache的对应大小的CentralFreeList中获取一部分空闲对象，这里需要加锁。
- 当CentralCache中的CentralFreeList没有空闲对象时，向PageHeap申请内存，将其划分成对应大小的空闲对象，加入到空闲链表中。
- 内存回收时，将空闲对象插入对应的ThreadCache的FreeList中，当满足一定条件将FreeList中的内存归还给CentralCache，当满足一定条件将CentralCache的CentralFreeList中的内存归还给PageHeap

#### 中对象分配(256K<size<=1M)

对于中对象的分配，从PageHeap中的FreeLsit中获取，PageHeap划分为128个span，每个span由1,2...128个page组成。

- 分配内存时首先向上取整到对应K个page
- 从PageHeap的对应大小的FreeList中尝试获取span
- 如果该大小没有空闲的span，则增大page往下寻找
- 找个n个page的span后，将该span划分为两部分
  - 将k个page的span作为结果返回
  - 将剩余的n-k个page的span，插入到PageHeap中
- 如果128个spanlist都没有空闲对象，则当做大对象来分配

#### 大对象分配(>1M)

对于大于1m的对象或者小于1m但PageHeap的spanList无法满足的对象，则采用大对象分配。

- 首先向上取整到对应K个page
- 大于128K的span都是使用红黑树来保存的，然后使用best-fit首次适应算法，找到合适的span
- 找个n个page的span后，将该span划分为两部分
  - 将k个page的span作为结果返回
  - 将剩余的n-k个page的span，如果n-k>128则插入红黑树中继续用于大对象的分配，n-k<=128则插入到PageHeap中
- 如果无法找到足够大小的空闲对象，则需要向系统申请新的内存

## 内存分配

### 分级分配

golang根据对象内存大小划分为3类，

- 微对象：0~16B
- 小对象：16B~32KB，以及<16B的指针对象
- 大对象：32KB以上

三级内存管理

- 线程缓存：每个线程有自己的内存缓存，如果内存大小能够满足，则直接在线程缓存上分配，不需要考虑全局锁的问题
- 中心缓存：当线程缓存无法满足时，就需要在全局的中心缓存来分配
- 页堆：当需要分配32KB以上的大对象时，使用页堆

### 内存管理

#### 内存管理单元mspan

- 每个mspan都管理`npages`个8K的页

  比如最小的分配8B的mspan，则需要一个8K的页，因此最后可以分配8K/8B=1024个对象

  比如最大的分配32K的mspan，则需要4个8K的页，因此最多可以分配32K/32K=1个对象

- mspan中使用`allocBits`来标记内存的占用情况，1表示已分配，0表示未使用

- mspan分为无指针的noscan和有指针的两种，与gc有关

#### 线程缓存mcache

- 每个P都有自己的mcache，用于分配小对象

- 每个mcache都有`alloc [numSpanClasses]*mspan`，67*2=134个mspan，用来分配不同大小的对象

- 每个P在初始化时会调用`allocmcache`来初始化mcache

  初始化时mcache中每个mspan都是空的`emptymspan`，并没有内存空间

- 当mcache中的mspan满或者为`emptymspan`时，调用`refill`从中心缓存中获取至少包含一个空闲对象空间的mspan

#### 中心缓存mcentral

- 每个spanclass对应一个mcentral，共134个

- mcentral中主要维护两个spanlist

  `nonempty  mSpanList`：表示存在空闲对象空间的mspan

  `empty     mSpanList`：表示没有空闲对象空间的mspan或者缓存在mcache中

- 主要方法`cacheSpan`该方法来返回span给mcache

```go
// Allocate a span to use in an mcache.
func (c *mcentral) cacheSpan() *mspan {
	sg := mheap_.sweepgen
retry:
	var s *mspan
	// 先从有空闲的span列表中查找
	for s = c.nonempty.first; s != nil; s = s.next {
		// sg-2表示等待被回收，可以拿来再次使用，从空闲中删除，添加到非空闲中
		if s.sweepgen == sg-2 && atomic.Cas(&s.sweepgen, sg-2, sg-1) {
			c.nonempty.remove(s)
			c.empty.insertBack(s)
			unlock(&c.lock)
			s.sweep(true)
			goto havespan
		}
		if s.sweepgen == sg-1 {
			// 该span正在被回收，跳过
			continue
		}
		// 找到一个空闲的不需要回收的span
		c.nonempty.remove(s)
		c.empty.insertBack(s)
		unlock(&c.lock)
		goto havespan
	}
	// 再从有非空闲的span列表中查找
	for s = c.empty.first; s != nil; s = s.next {
		if s.sweepgen == sg-2 && atomic.Cas(&s.sweepgen, sg-2, sg-1) {
			// 找到一个非空闲，需要清理的span，就尝试去清理内存，看能否使用
			c.empty.remove(s)
			c.empty.insertBack(s)
			unlock(&c.lock)
			s.sweep(true)
			// 在该span中可以找到空闲的对象空间，则使用
			freeIndex := s.nextFreeIndex()
			if freeIndex != s.nelems {
				s.freeindex = freeIndex
				goto havespan
			}
			lock(&c.lock)
			// 无法清理出空闲的对象空间，则重试
			goto retry
		}
		if s.sweepgen == sg-1 {
			// 该span正在被回收，跳过
			continue
		}
		// already swept empty span,
		// all subsequent ones must also be either swept or in process of sweeping
		break
	}
	// 无法找到可使用的span，需要扩容
	s = c.grow()
	c.empty.insertBack(s)
havespan:
	// 找到可以用的span，更新allocCache等字段
	freeByteBase := s.freeindex &^ (64 - 1)
	whichByte := freeByteBase / 8
	// Init alloc bits cache.
	s.refillAllocCache(whichByte)
	s.allocCache >>= s.freeindex % 64
	return s
}
```

- 扩容span

```go
// grow allocates a new empty span from the heap and initializes it for c's size class.
func (c *mcentral) grow() *mspan {
	// 获取该span需要的8K页的数量
	npages := uintptr(class_to_allocnpages[c.spanclass.sizeclass()])
	// 获取该span的内存大小
	size := uintptr(class_to_size[c.spanclass.sizeclass()])
	// 从mheap中分配span
	s := mheap_.alloc(npages, c.spanclass, true)
	// 清理位图信息
	n := (npages << _PageShift) >> s.divShift * uintptr(s.divMul) >> s.divShift2
	s.limit = s.base() + size*n
	heapBitsForAddr(s.base()).initSpan(s)
	return s
}
```

#### 页堆mheap

mheap管理整个虚拟内存空间，管理全局的mspan，并且包含了134种类型的mcentral

主要方法`alloc`，用于从堆中分配含有若干个page的mspan

```go
func (h *mheap) alloc(npages uintptr, spanclass spanClass, needzero bool) *mspan {
	var s *mspan
	systemstack(func() {
		if h.sweepdone == 0 {
			h.reclaim(npages)
		}
        // 切换到系统栈上运行
		s = h.allocSpan(npages, false, spanclass, &memstats.heap_inuse)
	})
	if s != nil {
		if needzero && s.needzero != 0 {
			memclrNoHeapPointers(unsafe.Pointer(s.base()), s.npages<<_PageShift)
		}
		s.needzero = 0
	}
	return s
}
```

// TODO

### 内存分配总结

所有堆上的对象，分配内存时都会调用`newobject`函数，该函数调用`mallocgc`进行内存分配

```go
c := gomcache()
var x unsafe.Pointer
noscan := typ == nil || typ.ptrdata == 0
```

首先获取当前mcache，并判断是否为指针类型

#### 微对象分配

对于<=16B且非指针类型的内存，使用微对象分配

微对象分配主要用在较小的字符串以及逃逸变量上

微对象分配是将多个微对象组合在一个内存块中，因此只有当该内存块中所有的对象都被回收时，该内存块才会被回收

- 首先获取微对象内存块的偏移量，如果该内存还有足够空闲空间分配，则分配完成
- 当没有足够空闲空间时，则调用`nextFreeFast`找到微对象对应大小的mspan，然后用于分配微对象
- 当没有空闲mspan时，则调用`nextFree`从mcentral和mheap中获取内存

#### 小对象分配

对于16B<size<=32KB的对象，以及<16B的指针对象，使用小对象分配

- 首先确定申请内存对应的mspan类型
- 然后先从mcache中尝试查找空闲空间`nextFreeFast`，如果存在足够空闲空间则分配
- 如果没有空闲空间，则调用`nextFree`，调用`refill`从mcentral中替换没有空闲空间的mspan
- 当从mcentral中没有找到空闲的mspan则需要扩容，从mheap中获取内存

#### 大对象分配

对于>32KB的对象，使用大对象分配

直接通过系统栈调用`largeAlloc`函数从堆上分配

- 首先计算所需要的page数量
- 调用`mheap_.alloc`从堆上分配内存，对于大对象使用的是类型为0的span

### 参考

- [https://draveness.me/golang/docs/part3-runtime/ch07-memory/golang-memory-allocator/](https://draveness.me/golang/docs/part3-runtime/ch07-memory/golang-memory-allocator/)
- [https://golang.design/under-the-hood/zh-cn/part2runtime/ch07alloc/](https://golang.design/under-the-hood/zh-cn/part2runtime/ch07alloc/)
- [https://lessisbetter.site/2019/07/06/go-memory-allocation/](https://lessisbetter.site/2019/07/06/go-memory-allocation/)
- [https://i6448038.github.io/2019/05/18/golang-mem/](https://i6448038.github.io/2019/05/18/golang-mem/)
- [https://gperftools.github.io/gperftools/tcmalloc.html](https://gperftools.github.io/gperftools/tcmalloc.html)
- [https://wallenwang.com/2018/11/tcmalloc/](https://wallenwang.com/2018/11/tcmalloc/)

### Q

1. 隔离适应（Segregated-Fit）— 将内存分割成多个链表，每个链表中的内存块大小相同，申请内存时先找到满足条件的链表，再从链表中选择合适的内存块；
2. **madvise**