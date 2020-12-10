---
title: "Go for Range遍历"
date: 2020-11-21T21:05:49+08:00
tags:
- go
categories: 
- go
---

### go for range遍历

#### 问题

在代码中使用for-range遍历切片，然后在循环体中append，无法遍历完整引发的疑问。

<!--more-->

```go
func main() {
	arr := []int{1,2,3}
	for k, v := range arr {
		if k == 0 {
			arr = append(arr, 4)
		}
		fmt.Printf("%d ", v)
	}
}
// 输出结果为 1 2 3
```

可以看到for-range遍历的结果没有4，只有起始的1 2 3

#### 解析

首先弄清楚for-range的遍历底层是如何实现的。

[https://garbagecollected.org/2017/02/22/go-range-loop-internals/](https://garbagecollected.org/2017/02/22/go-range-loop-internals/)

[https://golang.org/ref/spec#For_statements](https://golang.org/ref/spec#For_statements)

其中提到了range就是一个语法糖，在range底层是传统的遍历方式，只不过还有其他一些操作。

对于切片的实现代码如下：

```go
//   for_temp := range
//   len_temp := len(for_temp)
//   for index_temp = 0; index_temp < len_temp; index_temp++ {
//           value_temp = for_temp[index_temp]
//           index = index_temp
//           value = value_temp
//           original body
//   }
```

1. for_temp：指向原来的切片arr，我们知道切片实际上就是一个结构体，里面包含了指向底层数组的指针+长度+容量，因此这里只是复制了一份结构体，指向了同一个底层数组。由于是指向了同一个底层数组，因此对数组的改变，比如`arr[0]=1`,依然是可以感觉到的。但假如切片发生了扩容就失效了。
2. len_temp：原切片的原长度
3. 接下来是传统的遍历方式，由于` index_temp < len_temp`。因此循环只会执行原长度的次数
4. index，value：for-range左侧的短变量

**因此，如果我们在循环体中继续append，由于之前已经算出了len_temp因此并不会增加循环执行次数。**

#### 其他细节

- range变量

上面的for-range代码中还涉及到range左侧的变量问题

```go
// for index,value := range arr
```

在range的短变量声明中，我们知道可以获取到下标和值，但这里的index和value都是副本，每次for-range迭代中，都会重用，因此在for-range迭代中修改value的值是不会对原数组有影响的。

```go
arr := []int{1, 2, 3}
for _, v := range arr {
    fmt.Println(v)
    v = 2
}
fmt.Println(arr)
// 输出结果仍为1 2 3
```

- 底层数组

上面提到for_temp和原切片arr底层指向的是同一个数组，对原切片的修改，for-range循环中是可以感觉到的。

```go
arr := []int{1, 2, 3}
for k, v := range arr {
    if k == 0 {
        arr[2] = 4
    }
    fmt.Printf("%d ", v)
}
// 输出结果为1 2 4
```

但假如在for-range循环体中对原切片append导致了扩容，那么就无法感觉到了。因为这里扩容之后使得arr指向的底层数组已经改变，`arr[2] = 4`是在对新底层数组修改，原来复制体指向的底层数并没有改变。

```go
arr := []int{1, 2, 3}
for k, v := range arr {
    if k == 0 {
        arr = append(arr, 5)
        arr[2] = 4
    }
    fmt.Printf("%d ", v)
}
fmt.Println(arr)
// 循环提中输出结果为1 2 3
// arr为1 2 4 5
```

