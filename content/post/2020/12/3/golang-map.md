---
title: "Golang Map"
date: 2020-12-03T21:00:44+08:00
tags:
- go
categories: 
- go
---

## golang map分析

### map

- 底层结构
- get
- set
- del
- 扩容

<!--more-->

#### 底层结构

```go
// A header for a Go map.
type hmap struct {
	// Note: the format of the hmap is also encoded in cmd/compile/internal/gc/reflect.go.
	// Make sure this stays in sync with the compiler's definition.
	count     int // # live cells == size of map.  Must be first (used by len() builtin)
	flags     uint8
	B         uint8  // log_2 of # of buckets (can hold up to loadFactor * 2^B items)
	noverflow uint16 // approximate number of overflow buckets; see incrnoverflow for details
	hash0     uint32 // hash seed

	buckets    unsafe.Pointer // array of 2^B Buckets. may be nil if count==0.
	oldbuckets unsafe.Pointer // previous bucket array of half the size, non-nil only when growing
	nevacuate  uintptr        // progress counter for evacuation (buckets less than this have been evacuated)

	extra *mapextra // optional fields
}
```

- count：map集合中元素的个数，len函数的返回值
- B：buckets数组大小的对数
- buckets：底层数组，大小为2^B
- oldbuckets：扩容时使用的数组，用于保存扩容前的buckets数组，大小为新buckets的一半

```go
type bmap struct {
    topbits  [8]uint8
    keys     [8]keytype
    values   [8]valuetype
    pad      uintptr
    overflow uintptr
}
```

buckets数组内部存的就是bmap的数组。

- topbits：存储内部8个k-v键值对的key的高8位
- keys：保存所有key的数组
- values：保存所有value的数组，将keys和values分开数组存放是为了内存对齐
- overflow：指向溢出bmap，即当放入当前bmap的k-v超出8个时，就会创建新的bmap，然后overflow指向新的bmap

#### get操作

1. 对key做hash运算
2. 对结果取后B位，用于定位buckets数组中该k-v所在的bmap的位置
3. 遍历当前bmap中的tophash数组，比较hash结果的高8位于tophash的结果
4. 如果tophash与hash结果高8位相同，再比较key和bmap中keys对应位置的key，这里可以看出tophash是不能直接进行定位的，只是起到一个比较的结果，当然tophash还有其他特殊值
5. 如果key也相等，则定位到values数组中对应位置的value
6. 如果在当前bmap中遍历完了所有的tophash和keys都没有对应值，则遍历bmap中的溢出链，寻找下一个bmap，回到步骤3
7. 如果bmap及其所有溢出bmap都遍历完了都没有，则返回value的零值，而不是返回nil

```go
// mapaccess1 returns a pointer to h[key].  Never returns nil, instead
// it will return a reference to the zero object for the elem type if
// the key is not in the map.
// NOTE: The returned pointer may keep the whole map live, so don't
// hold onto it for very long.
func mapaccess1(t *maptype, h *hmap, key unsafe.Pointer) unsafe.Pointer {
	// 如果map为nil或者没有元素，直接返回value零值
	if h == nil || h.count == 0 {
		if t.hashMightPanic() {
			t.hasher(key, 0) // see issue 23734
		}
		return unsafe.Pointer(&zeroVal[0])
	}
	// 读写冲突检查
	if h.flags&hashWriting != 0 {
		throw("concurrent map read and map write")
	}
	// 计算hash值
	hash := t.hasher(key, uintptr(h.hash0))
	// 计算2^B-1
	m := bucketMask(h.B)
	// hash&m 计算出后B为，定位bucket位置
	// 这里的运算应该是对 hash%buckets的长度 来定位，这里使用&运算来代替%运算加速。
	//    hash % len(buckets)
	// => hash % 2^B
	// => hash & (2^B-1)
	// => hash & m
	b := (*bmap)(add(h.buckets, (hash&m)*uintptr(t.bucketsize)))
	// old不为空，说明在扩容，因此需要判断是否去oldbucket中查找
	if c := h.oldbuckets; c != nil {
		// 如果不是等量扩容则需要将m/2，因为扩容时新buckets的容量是old的两倍
		if !h.sameSizeGrow() {
			// There used to be half as many buckets; mask down one more power of two.
			m >>= 1
		}
		oldb := (*bmap)(add(c, (hash&m)*uintptr(t.bucketsize)))
      // oldb未被扩容搬运，表示需要去oldbucket中查找，则将b指向old
		if !evacuated(oldb) {
			b = oldb
		}
	}
	// 取hash值的高8位
	top := tophash(hash)
bucketloop:
	// 遍历bmap及其溢出链
	for ; b != nil; b = b.overflow(t) {
		// bucketCntBits = 3
		// bucketCnt     = 1 << bucketCntBits
		// 一个bmap最多存8个k-v
		for i := uintptr(0); i < bucketCnt; i++ {
			if b.tophash[i] != top {
				// emptyRest表示当前tophash对应的k-v为空，并且其后都为空，因此不用继续找下去了
				if b.tophash[i] == emptyRest {
					break bucketloop
				}
				continue
			}
			// 找到tophash对应值的key，定位到key的位置
			k := add(unsafe.Pointer(b), dataOffset+i*uintptr(t.keysize))
			if t.indirectkey() {
				k = *((*unsafe.Pointer)(k))
			}
			// 比较key，如果相等则表示找到了对应的k-v
			if t.key.equal(key, k) {
				e := add(unsafe.Pointer(b), dataOffset+bucketCnt*uintptr(t.keysize)+i*uintptr(t.elemsize))
				if t.indirectelem() {
					e = *((*unsafe.Pointer)(e))
				}
				return e
			}
		}
	}
	// 所有的bmap都找过了，都没有，则返回value的零值
	return unsafe.Pointer(&zeroVal[0])
}
```

用到的其他函数代码

```go
// 取后h.B位
// m := uintptr(1) << (h.B & (sys.PtrSize*8 - 1)) - 1
m := bucketMask(h.B)
func bucketMask(b uint8) uintptr {
	return bucketShift(b) - 1
}
func bucketShift(b uint8) uintptr {
	// Masking the shift amount allows overflow checks to be elided.
	return uintptr(1) << (b & (sys.PtrSize*8 - 1))
}

// 计算hash的高8位
func tophash(hash uintptr) uint8 {
	top := uint8(hash >> (sys.PtrSize*8 - 8))
	// minTopHash及之前的一些值，是map预定义用来表示状态的一些值
	// 如果计算的高8位小于需要加上minTopHash则需要加上minTopHash，以区别正常hash值和状态值
	if top < minTopHash {
		top += minTopHash
	}
	return top
}
```

get操作还有返回bool值表示k-v是否存在的操作。实际由两个函数来实现。

实现细节相同，mapaccess2在返回值上多添加一个bool

```go
func mapaccess1(t *maptype, h *hmap, key unsafe.Pointer) unsafe.Pointer
func mapaccess2(t *maptype, h *hmap, key unsafe.Pointer) (unsafe.Pointer, bool)
```

#### set操作

与get操作类似，底层使用`mapassign`函数实现

```go
// Like mapaccess, but allocates a slot for the key if it is not present in the map.
func mapassign(t *maptype, h *hmap, key unsafe.Pointer) unsafe.Pointer {
	if h.flags&hashWriting != 0 {
		throw("concurrent map writes")
	}
	hash := t.hasher(key, uintptr(h.hash0))
	h.flags ^= hashWriting

	// bucket为空，分配第一个新的bucket
	if h.buckets == nil {
		h.buckets = newobject(t.bucket) // newarray(t.bucket, 1)
	}

again:
	bucket := hash & bucketMask(h.B)
   // 如果是扩容状态，则优先扩容当前bucket
	if h.growing() {
		growWork(t, h, bucket)
	}
	b := (*bmap)(unsafe.Pointer(uintptr(h.buckets) + bucket*uintptr(t.bucketsize)))
	top := tophash(hash)

	// inserti -> 表示新k-v在tophash数组中的位置
	// insertk -> 表示新k-v的key在keys中的位置
	// elem    -> 表示新k-v的value的地址
	var inserti *uint8
	var insertk unsafe.Pointer
	var elem unsafe.Pointer
bucketloop:
	// 与mapaccess类似。遍历bmap及其溢出链
	for {
		for i := uintptr(0); i < bucketCnt; i++ {
			// 判断新k-v是否在bucket中已经存在
			if b.tophash[i] != top {
				// 如果当前tophash位置不是新k-v的位置，则判断他是否为空位
				// 在插入新空位时，应该尽可能的在前面的空位插入，因此需要做inserti == nil判断，
				// inserti != nil表示已经找到了更前的空位，不使用新位置。
				if isEmpty(b.tophash[i]) && inserti == nil {
					inserti = &b.tophash[i]
					insertk = add(unsafe.Pointer(b), dataOffset+i*uintptr(t.keysize))
					elem = add(unsafe.Pointer(b), dataOffset+bucketCnt*uintptr(t.keysize)+i*uintptr(t.elemsize))
				}
				if b.tophash[i] == emptyRest {
					break bucketloop
				}
				continue
			}
			// tophash值与新k-v的高8位相同，则判断key是否相同
			k := add(unsafe.Pointer(b), dataOffset+i*uintptr(t.keysize))
			if t.indirectkey() {
				k = *((*unsafe.Pointer)(k))
			}
			// key不相同则，则跳过，寻找下一个
			if !t.key.equal(key, k) {
				continue
			}
			// key相同，则更新即可
			if t.needkeyupdate() {
				typedmemmove(t.key, k, key)
			}
			elem = add(unsafe.Pointer(b), dataOffset+bucketCnt*uintptr(t.keysize)+i*uintptr(t.elemsize))
			goto done
		}
		// 遍历溢出链
		ovf := b.overflow(t)
		if ovf == nil {
			break
		}
		b = ovf
	}

	// 没有在map找到key，需要分配新的位置
	// 如果map中键值对数量触发了扩容，或者bmap溢出过多，则需要扩容
	if !h.growing() && (overLoadFactor(h.count+1, h.B) || tooManyOverflowBuckets(h.noverflow, h.B)) {
		hashGrow(t, h)
		// 触发了扩容，调整了map结构，因此需要重新查找合适的位置
		goto again // Growing the table invalidates everything, so try again
	}

	// 表示在bmap及其溢出链都没有合适的位置，即都满了，则需要创建新的溢出bmap，将新k-v分配到新的bmap中
	if inserti == nil {
		newb := h.newoverflow(t, b)
		inserti = &newb.tophash[0]
		insertk = add(unsafe.Pointer(newb), dataOffset)
		elem = add(insertk, bucketCnt*uintptr(t.keysize))
	}

	// 在对应位置插入key和value
	if t.indirectkey() {
		kmem := newobject(t.key)
		*(*unsafe.Pointer)(insertk) = kmem
		insertk = kmem
	}
	if t.indirectelem() {
		vmem := newobject(t.elem)
		*(*unsafe.Pointer)(elem) = vmem
	}
	typedmemmove(t.key, insertk, key)
	*inserti = top
	h.count++

done:
	if h.flags&hashWriting == 0 {
		throw("concurrent map writes")
	}
	h.flags &^= hashWriting
	if t.indirectelem() {
		elem = *((*unsafe.Pointer)(elem))
	}
	return elem
}
```

`mapassign`函数最后的返回值为value的地址，需要通过汇编将实际的value值赋值到value的地址上。

[https://github.com/cch123/golang-notes/blob/master/map.md](https://github.com/cch123/golang-notes/blob/master/map.md)

>这里有件比较奇怪的事情，mapassign 并没有把用户 `m[k] = v` 时的 v 写入到 map 的 value 区域，而是直接返回了这个值所应该在的内存地址。那么把 v 拷贝到该内存区域的操作是在哪里做的呢？
>
>```
>var a = make(map[int]int, 7)
>for i := 0; i < 1000; i++ {
>   a[i] = 99999
>}
>```
>
>看看生成的汇编部分:
>
>```
>0x003f 00063 (m.go:9)    MOVQ    DX, (SP) // 第一个参数
>0x0043 00067 (m.go:9)    MOVQ    AX, 8(SP) // 第二个参数
>0x0048 00072 (m.go:9)    MOVQ    CX, 16(SP) // 第三个参数
>0x004d 00077 (m.go:9)    PCDATA    $0, $1 // GC 相关
>0x004d 00077 (m.go:9)    CALL    runtime.mapassign_fast64(SB) // 调用函数
>0x0052 00082 (m.go:9)    MOVQ    24(SP), AX // 返回值，即 value 应该存放的内存地址
>0x0057 00087 (m.go:9)    MOVQ    $99999, (AX) // 把 99999 放入该地址中
>```
>
>赋值的最后一步实际上是编译器额外生成的汇编指令来完成的，可见靠 runtime 有些工作是没有做完的。这里和 go 在函数调用时插入 prologue 和 epilogue 是类似的。编译器和 runtime 配合，才能完成一些复杂的工作。

#### del操作

底层使用`mapdelete`函数实现，首先查找到cell对应的位置，再将key和value内存清空，查找过程与get操作类似。

在清空内存后，还需要调整bmap及其溢出链的tophash的状态值，将一些emptyOne调整为emptyReset

```go
func mapdelete(t *maptype, h *hmap, key unsafe.Pointer) {
	// map为空，或者没有元素，则直接返回
	if h == nil || h.count == 0 {
		if t.hashMightPanic() {
			t.hasher(key, 0) // see issue 23734
		}
		return
	}
	if h.flags&hashWriting != 0 {
		throw("concurrent map writes")
	}

	hash := t.hasher(key, uintptr(h.hash0))

	// Set hashWriting after calling t.hasher, since t.hasher may panic,
	// in which case we have not actually done a write (delete).
	h.flags ^= hashWriting

	bucket := hash & bucketMask(h.B)
	// 如果正常扩容，则优先做当前bucket的扩容处理
	if h.growing() {
		growWork(t, h, bucket)
	}
	// 定位到对应的bucket上
	b := (*bmap)(add(h.buckets, bucket*uintptr(t.bucketsize)))
	// 记录对应bucket的第一个bmap，后续的reset需要用到
	bOrig := b
	top := tophash(hash)
search:
	// 与赋值类似，遍历bmap及其溢出链
	for ; b != nil; b = b.overflow(t) {
		for i := uintptr(0); i < bucketCnt; i++ {
			if b.tophash[i] != top {
				if b.tophash[i] == emptyRest {
					break search
				}
				continue
			}
			// 定位到对应的key的位置
			k := add(unsafe.Pointer(b), dataOffset+i*uintptr(t.keysize))
			k2 := k
			if t.indirectkey() {
				k2 = *((*unsafe.Pointer)(k2))
			}
			if !t.key.equal(key, k2) {
				continue
			}
			// 清空key的内存
			if t.indirectkey() {
				*(*unsafe.Pointer)(k) = nil
			} else if t.key.ptrdata != 0 {
				memclrHasPointers(k, t.key.size)
			}
			// 定位value的位置，清空value的内存
			e := add(unsafe.Pointer(b), dataOffset+bucketCnt*uintptr(t.keysize)+i*uintptr(t.elemsize))
			if t.indirectelem() {
				*(*unsafe.Pointer)(e) = nil
			} else if t.elem.ptrdata != 0 {
				memclrHasPointers(e, t.elem.size)
			} else {
				memclrNoHeapPointers(e, t.elem.size)
			}
			// 将当前位置对应的tophash设置为emptyOne状态，表示当前cell为空
			b.tophash[i] = emptyOne

			// 调整当前bucket中cell的状态，将一些emptyOne设置为emptyRest
			// emptyOne表示当前cell为空，emptyRest表示当前cell及其后都为空，因此emptyRest可以避免多余的遍历操作

			// 如果当前删除的k-v为bmap中的最后一个，即第7位
			if i == bucketCnt-1 {
				// 当前bmap后面还有溢出链，并且溢出链中有内容，则跳到收尾处理
				if b.overflow(t) != nil && b.overflow(t).tophash[0] != emptyRest {
					goto notLast
				}
			} else {
				// 当前删除的k-v不是最后一个，则需要判断他的下一位是否为emptyRest，
				// 如果下一位状态不emptyRest，则表示其后还有内存，则跳到收尾处理
				if b.tophash[i+1] != emptyRest {
					goto notLast
				}
			}
			// 如果能到这里，表示当前删除的k-v的后面所有cell状态都为emptyRest，
			// 因此需要调整当前cell为emptyRest，并且继续向前调整
			for {
				b.tophash[i] = emptyRest
				// 如果当前cell为bmap中的首位，则需要找溢出链的前一个bmap
				if i == 0 {
					// 表示当前已经是首个bmap，因此调整完了。
					if b == bOrig {
						break
					}
					// 类似单向链表查找前一项，找到前一个bmap
					c := b
					for b = bOrig; b.overflow(t) != c; b = b.overflow(t) {
					}
					// 此时b指向了前一个的bmap。将i调整为bmap的最后一位，即7
					i = bucketCnt - 1
				} else { // 如果当前cell不是首位，则直接i--往前找cell
					i--
				}
				// 如果往前找到的cell位置状态都为emptyOne，则需要将他们置为emptyRest，
				// 直到找到一个cell位置不为空
				if b.tophash[i] != emptyOne {
					break
				}
			}
		notLast:
			// 最后的收尾处理，将map元素数量减1
			h.count--
			break search
		}
	}

	if h.flags&hashWriting == 0 {
		throw("concurrent map writes")
	}
	h.flags &^= hashWriting
}
```

#### 扩容

扩容触发时机，在`mapassign`函数中，当需要分配新bucket时会检查是否需要扩容

```go
if !h.growing() && (overLoadFactor(h.count+1, h.B) || tooManyOverflowBuckets(h.noverflow, h.B)) {
    // 扩容函数
    hashGrow(t, h)
    goto again // Growing the table invalidates everything, so try again
}
```

扩容的两个判断条件：

1. 元素数量超过扩容界限，即元素数量>6.5*2^B，此时map查询效率降低，需要增量扩容来提高查找效率
2. bucket溢出链过长，当某些元素落入同一bucket时，频繁的插入和删除，可能导致许多bucket中并没有填满8个，因此需要等量扩容，来紧缩这些内存空间

```go
// overLoadFactor reports whether count items placed in 1<<B buckets is over loadFactor.
func overLoadFactor(count int, B uint8) bool {
	return count > bucketCnt && uintptr(count) > loadFactorNum*(bucketShift(B)/loadFactorDen)
}
// tooManyOverflowBuckets reports whether noverflow buckets is too many for a map with 1<<B buckets.
// Note that most of these overflow buckets must be in sparse use;
// if use was dense, then we'd have already triggered regular map growth.
func tooManyOverflowBuckets(noverflow uint16, B uint8) bool {
	// If the threshold is too low, we do extraneous work.
	// If the threshold is too high, maps that grow and shrink can hold on to lots of unused memory.
	// "too many" means (approximately) as many overflow buckets as regular buckets.
	// See incrnoverflow for more details.
	if B > 15 {
		B = 15
	}
	// The compiler doesn't see here that B < 16; mask B to generate shorter shift code.
	return noverflow >= uint16(1)<<(B&15)
}
```

扩容函数，扩容前的设置在`hashGrow`函数中

```go
func hashGrow(t *maptype, h *hmap) {
	// 判断是否为超过扩容因子触发的扩容
	bigger := uint8(1)
	if !overLoadFactor(h.count+1, h.B) {
		bigger = 0
		h.flags |= sameSizeGrow
	}
	// 将buckets挂载到old上
	oldbuckets := h.buckets
	// 创建大小为2^(h.B+bigger)的新buckets数组
	newbuckets, nextOverflow := makeBucketArray(t, h.B+bigger, nil)

	flags := h.flags &^ (iterator | oldIterator)
	if h.flags&iterator != 0 {
		flags |= oldIterator
	}
	// commit the grow (atomic wrt gc)
	h.B += bigger
	h.flags = flags
	h.oldbuckets = oldbuckets
	h.buckets = newbuckets
	h.nevacuate = 0
	h.noverflow = 0

	if h.extra != nil && h.extra.overflow != nil {
		// Promote current overflow buckets to the old generation.
		if h.extra.oldoverflow != nil {
			throw("oldoverflow is not nil")
		}
		h.extra.oldoverflow = h.extra.overflow
		h.extra.overflow = nil
	}
	if nextOverflow != nil {
		if h.extra == nil {
			h.extra = new(mapextra)
		}
		h.extra.nextOverflow = nextOverflow
	}
	// the actual copying of the hash table data is done incrementally
	// by growWork() and evacuate().
}
```

map中buckets的重新分配调整在`growWork`函数中，因此`mapassign`函数中，是先通过判断`overLoadFactor`函数和`tooManyOverflowBuckets`函数来检测是否需要扩容，如果需要扩容，则调用`hashGrow`函数来设置扩容状态，最后再跳回如何代码段中，调用growWork()来扩容

```go
if h.growing() {
    growWork(t, h, bucket)
}
```

扩容函数

```go
func growWork(t *maptype, h *hmap, bucket uintptr) {
	// 确保搬运的bucket是正在使用的
	evacuate(t, h, bucket&h.oldbucketmask())

	// 如果还是扩容状态，则再搬运一个
	if h.growing() {
		evacuate(t, h, h.nevacuate)
	}
}
```

搬运函数`evacuate`，该函数一次只能搬运一个bucket

```go
func evacuate(t *maptype, h *hmap, oldbucket uintptr) {
	// 定位当前bucket的位置
	b := (*bmap)(add(h.oldbuckets, oldbucket*uintptr(t.bucketsize)))
	// 记录扩容前map的容量大小，即扩容前的2^B
	newbit := h.noldbuckets()
	// 判断当前bucket是否已经完成扩容
	if !evacuated(b) {

		// x和y指向扩容后在新buckets中的位置
		var xy [2]evacDst
		x := &xy[0]
		// 定位目标bmap地址
		x.b = (*bmap)(add(h.buckets, oldbucket*uintptr(t.bucketsize)))
		// 定位目标bmap的keys地址
		x.k = add(unsafe.Pointer(x.b), dataOffset)
		// 定位目标bmap的values地址
		x.e = add(x.k, bucketCnt*uintptr(t.keysize))

		// 如果是增量扩容，则y表示扩容后的高位地址，在增量扩容时，由于是两倍放大，因此目标bmap有两个，一个低位一个高位
		if !h.sameSizeGrow() {
			// Only calculate y pointers if we're growing bigger.
			// Otherwise GC can see bad pointers.
			y := &xy[1]
			y.b = (*bmap)(add(h.buckets, (oldbucket+newbit)*uintptr(t.bucketsize)))
			y.k = add(unsafe.Pointer(y.b), dataOffset)
			y.e = add(y.k, bucketCnt*uintptr(t.keysize))
		}

		// 遍历old中的该bmap及其溢出链
		for ; b != nil; b = b.overflow(t) {
			k := add(unsafe.Pointer(b), dataOffset)
			e := add(k, bucketCnt*uintptr(t.keysize))
			// 遍历搬运所有cell
			for i := 0; i < bucketCnt; i, k, e = i+1, add(k, uintptr(t.keysize)), add(e, uintptr(t.elemsize)) {
				top := b.tophash[i]
				// 如果该位为空，则将其置为已经搬运过的状态
				if isEmpty(top) {
					b.tophash[i] = evacuatedEmpty
					continue
				}
				// 异常情况，未被搬运的只能是空或者tophash正常值的cell
				if top < minTopHash {
					throw("bad map state")
				}
				k2 := k
				if t.indirectkey() {
					k2 = *((*unsafe.Pointer)(k2))
				}
				var useY uint8
				if !h.sameSizeGrow() {
					// 对需要搬运的cell的key做一次hash运算，决定需要搬运到x位置还是y位置
					hash := t.hasher(k2, uintptr(h.hash0))
					if h.flags&iterator != 0 && !t.reflexivekey() && !t.key.equal(k2, k2) {
						// 针对key为NaN的特殊处理
						useY = top & 1
						top = tophash(hash)
					} else {
						// 对hash判断新的高位是否为1来决定搬运到x位置还是y位置
						// 例如对于oldhash我们关注后3位，因此增量扩容后，则需要关注后4位
						// 因此如果第4位为0，则表示还是在地位，如果为1，则表示搬运到高位
						if hash&newbit != 0 {
							useY = 1
						}
					}
				}

				if evacuatedX+1 != evacuatedY || evacuatedX^1 != evacuatedY {
					throw("bad evacuatedN")
				}

				// evacuatedX + 1 == evacuatedY
				// evacuatedX表示搬运到地位，evacuatedY表示搬运到高位，与上面算的useY做加表示新状态
				b.tophash[i] = evacuatedX + useY
				// 目的bmap的地址
				dst := &xy[useY]

				// 如果目标bmap已经满了，则创建溢出bmap
				if dst.i == bucketCnt {
					dst.b = h.newoverflow(t, dst.b)
					dst.i = 0
					dst.k = add(unsafe.Pointer(dst.b), dataOffset)
					dst.e = add(dst.k, bucketCnt*uintptr(t.keysize))
				}
				// 设置目标bmap的tophash
				dst.b.tophash[dst.i&(bucketCnt-1)] = top // mask dst.i as an optimization, to avoid a bounds check
				// 搬运key
				if t.indirectkey() {
					*(*unsafe.Pointer)(dst.k) = k2 // copy pointer
				} else {
					typedmemmove(t.key, dst.k, k) // copy elem
				}
				// 搬运value
				if t.indirectelem() {
					*(*unsafe.Pointer)(dst.e) = *(*unsafe.Pointer)(e)
				} else {
					typedmemmove(t.elem, dst.e, e)
				}
				// 目标bmap中的cell数量增加
				dst.i++
				// 将dst的k指针和e指针后移一个单位，指向新的空位
				dst.k = add(dst.k, uintptr(t.keysize))
				dst.e = add(dst.e, uintptr(t.elemsize))
			}
		}
		// 清除old bucket
		if h.flags&oldIterator == 0 && t.bucket.ptrdata != 0 {
			b := add(h.oldbuckets, oldbucket*uintptr(t.bucketsize))
			// 只清除keys和values，保留tophash来表示状态
			ptr := add(b, dataOffset)
			n := uintptr(t.bucketsize) - dataOffset
			memclrHasPointers(ptr, n)
		}
	}

	if oldbucket == h.nevacuate {
		// 更新搬运进度
		advanceEvacuationMark(h, t, newbit)
	}
}
```

搬运的后续处理函数

```go
func advanceEvacuationMark(h *hmap, t *maptype, newbit uintptr) {
	// 搬运进度+1，nevacuate表示在其之前的bucket都已经搬运完成了
	h.nevacuate++
	// Experiments suggest that 1024 is overkill by at least an order of magnitude.
	// Put it in there as a safeguard anyway, to ensure O(1) behavior.
	stop := h.nevacuate + 1024
	if stop > newbit {
		stop = newbit
	}
	// 寻找还未搬运的bucket
	for h.nevacuate != stop && bucketEvacuated(t, h, h.nevacuate) {
		h.nevacuate++
	}
	// 搬运进度等于old buckets的长度，则说明整个old buckets已经搬运完成了
	if h.nevacuate == newbit { // newbit == # of oldbuckets
		// 清除old buckets，函数根据h.oldbuckets是否为nil来表示是否正在扩容
		h.oldbuckets = nil
		// 清除old overflow bucket
		if h.extra != nil {
			h.extra.oldoverflow = nil
		}
		h.flags &^= sameSizeGrow
	}
}
```

调用一次`evacuate `函数只能搬运一个bucket，而且调用`evacuate `函数的地方只在`growWork`函数中，而`growWork`函数的调用只存在`mapassign`函数和`mapdelete`函数中，因此好像只有主动触发了这两个方法才会使得扩容继续进行。对于扩容边界的地方，再增添一个新的cell，触发扩容，此时查看hmap的oldbuckets为非nil，表示处于扩容状态，推测需要后续手动触发增加扩容进度。这样的设计占用oldbuckets和newbuckets两份内存不会浪费吗？

### 参考

- [https://docs.kilvn.com/go-internals/02.3.html](https://docs.kilvn.com/go-internals/02.3.html)
- [https://github.com/qcrao/Go-Questions/tree/master/map](https://github.com/qcrao/Go-Questions/tree/master/map)
- [https://github.com/cch123/golang-notes/blob/master/map.md](https://github.com/cch123/golang-notes/blob/master/map.md)
- [https://draveness.me/golang/docs/part2-foundation/ch03-datastructure/golang-hashmap/#33-%E5%93%88%E5%B8%8C%E8%A1%A8](https://draveness.me/golang/docs/part2-foundation/ch03-datastructure/golang-hashmap/#33-%E5%93%88%E5%B8%8C%E8%A1%A8)