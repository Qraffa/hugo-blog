---
title: "Golang基本数据类型"
date: 2020-12-03T21:00:10+08:00
tags:
- go
categories: 
- go
---

## golang 基本数据类型

### 数组、字符串、切片

#### 数组

数组的特点是**长度固定**，拥有多个相同数据类型的数据元素。

**长度也是数组类型的一部分**。因此不同长度的数组是不同的数组，不同长度的数组也不能互相赋值。

<!--more-->

```go
// 定义数组的方式
a1 := [3]int{1,2,3} // 以固定长度定义。
a2 := [...]int{1,2} // 根据后面初始化的元素个数来动态决定数组长度。
```

关于数组作为函数参数的问题：

```go
func main() {
	arr := [3]int{1, 2, 3}
	charr(arr)
	fmt.Println(arr) // 结果为1 2 3 
	charrp(&arr)
	fmt.Println(arr) // 结果为-1 2 3
}
func charr(arr [3]int) {
	arr[0] = -1
}
func charrp(arr *[3]int) {
	arr[0] = -1
}
```

在go中，所有的函数参数传参都是传值的方式。[https://golang.org/doc/faq#pass_by_value](https://golang.org/doc/faq#pass_by_value)

因此对于第一个函数，修改的只是副本，因此原数组没有变化；对于第二个函数，传递的是指针，参数copy了一份指针，指向同一个底层数组，因此原数组被修改了。

#### 字符串

字符串的特点是**底层字节数组只读，不可修改**。

```go
type StringHeader struct {
	Data uintptr
	Len  int
}
```

由于字符串只读的特点，因此对于字符串的拼接，实际上是创建了新的底层字节数组，重新指向。

```go
func main() {
	s := "first"
	t := s
	s += "-second"
	fmt.Println(t) // first 底层仍然指向原来的first
	fmt.Println(s) // first-second 分配新的字符串，s指向新的字符串
}
```

由于字符串只读的特点，因此字符串原串和子串，**共享同一段底层字节序列**，因此生成子串的开销较低。

#### 切片

切片类似动态数组，长度不固定。

```go
type SliceHeader struct {
	Data uintptr
	Len  int
	Cap  int
}
```

SliceHeader表示一个切片，因此在切片的复制操作中，并不会真正复制底层数据，只是复制了一个SliceHeader指向同一个底层数组。因此将slice作为函数参数传递时，尽管是传值的设计，但由于指向了同一个底层数组，因此在函数内对slice的修改，在函数外是可以共享的。同理对于slice的分割操作，与字符串的子串类似。

**切片的截取**：可以有三个参数low,high,max需要满足low≤high≤max。low表示被截取切片的起始下标，high表示被截取下标的结束下标，low和high满足左闭右开。对于第三个参数max，新切片cap=max-low，表示能截取到原切片最大的下标，开区间；max需要满足≤原切片cap，否则运行错误；当max缺省时，max=原切片cap

```go
arr := []int{1, 2, 3, 4, 5, 6}
fmt.Println(len(arr), cap(arr)) // 6 6 
as1 := arr[0:3]
fmt.Println(len(as1), cap(as1)) // 3 6
as2 := arr[1:3:5]
fmt.Println(len(as2), cap(as2)) // 2 4
```

##### 切片的扩容

扩容策略：

1. 如果申请容量大于原容量的两倍，则扩容容量为申请容量
2. 如果原容量小于1024，则每次扩容为原来的2倍
3. 如果原容量大于1024，则每次增长原容量的1/4，直到满足申请容量或者溢出
4. 如果溢出，则扩容容量为申请容量

```go
newcap := old.cap
doublecap := newcap + newcap
if cap > doublecap {
    newcap = cap
} else {
    if old.len < 1024 {
        newcap = doublecap
    } else {
        // Check 0 < newcap to detect overflow
        // and prevent an infinite loop.
        for 0 < newcap && newcap < cap {
            newcap += newcap / 4
        }
        // Set newcap to the requested cap when
        // the newcap calculation overflowed.
        if newcap <= 0 {
            newcap = cap
        }
    }
}
```

除此之外，在计算了新容量大小后，会在内存分配上，对新容量的大小做一个调整，保证内存对齐。

```go
switch {
	case et.size == 1:
		lenmem = uintptr(old.len)
		newlenmem = uintptr(cap)
		capmem = roundupsize(uintptr(newcap))
		overflow = uintptr(newcap) > maxAlloc
		newcap = int(capmem)
	case et.size == sys.PtrSize:
		lenmem = uintptr(old.len) * sys.PtrSize
		newlenmem = uintptr(cap) * sys.PtrSize
		capmem = roundupsize(uintptr(newcap) * sys.PtrSize)
		overflow = uintptr(newcap) > maxAlloc/sys.PtrSize
		newcap = int(capmem / sys.PtrSize)
```



```go
// Returns size of the memory block that mallocgc will allocate if you ask for the size.
func roundupsize(size uintptr) uintptr {
	if size < _MaxSmallSize {
		if size <= smallSizeMax-8 {
			return uintptr(class_to_size[size_to_class8[divRoundUp(size, smallSizeDiv)]])
		} else {
			return uintptr(class_to_size[size_to_class128[divRoundUp(size-smallSizeMax, largeSizeDiv)]])
		}
	}
	if size+_PageSize < size {
		return size
	}
	return alignUp(size, _PageSize)
}
// divRoundUp returns ceil(n / a).
func divRoundUp(n, a uintptr) uintptr {
	// a is generally a power of two. This will get inlined and
	// the compiler will optimize the division.
	return (n + a - 1) / a
}
```

底层会根据数据类型大小和请求的内存大小，在预定义的内存对齐表中查找到对应的值，才是最终扩容后新的容量