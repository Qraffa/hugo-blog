---
title: "LFU 缓存"
date: 2021-02-28T23:17:10+08:00
url: lfu-cache
tags:
- 算法
categories: 
- 算法
---

## LFU 缓存

### 题目描述

请你为 最不经常使用（LFU）缓存算法设计并实现数据结构。

实现 LFUCache 类：

LFUCache(int capacity) - 用数据结构的容量 capacity 初始化对象
int get(int key) - 如果键存在于缓存中，则获取键的值，否则返回 -1。
void put(int key, int value) - 如果键已存在，则变更其值；如果键不存在，请插入键值对。当缓存达到其容量时，则应该在插入新项之前，使最不经常使用的项无效。在此问题中，当存在平局（即两个或更多个键具有相同使用频率）时，应该去除 最久未使用 的键。
注意「项的使用次数」就是自插入该项以来对其调用 get 和 put 函数的次数之和。使用次数会在对应项被移除后置为 0 。

为了确定最不常使用的键，可以为缓存中的每个键维护一个 使用计数器 。使用计数最小的键是最久未使用的键。

当一个键首次插入到缓存中时，它的使用计数器被设置为 1 (由于 put 操作)。对缓存中的键执行 get 或 put 操作，使用计数器的值将会递增。

来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/lfu-cache
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

你是否可以在 `O(1)` 时间复杂度内完成这两种操作？

<!--more-->

### 思路

首先需要考虑这题[LRU 缓存机制](https://leetcode-cn.com/problems/lru-cache/)，两题大体思路相近

这题中，当缓存满时，删除策略首先考虑使用频次，再考虑使用时间

因此需要对每个k-v保存一个使用频次的计数器，还需要对使用时间做一个排序链表，这个地方和LRU思路一致

因此在LFU中，存在两个主要结构：

- cache：key为使用频次，value为该使用频次下的值的链表
- hash：与k-v一致，其中v与cache的链表中的node一致

对于删除还需要考虑目前该结构中使用频次最低的值是多少，因此还需要一个minCnt来记录，当前需要删除的频次是多少

**get操作：**

1. 如果hash中不存在该key，则直接返回-1
2. 如果hash中存在该key
   1. 首先判断，minCnt是否为该node的使用频次use，且该使用频次下的元素只有这一个、
      1. 如果是，则需要将minCnt调整为use+1，即表示最低的使用频次更新
   2. 将该node从cache中对应的使用频次列表中删除
   3. 将该node的使用频次+1
   4. 将该node添加到cache的新的频次的列表中的最后一位

**put操作：**

1. 首先是特殊处理，cap为0的情况，直接return
2. 判断hash中是否存在该key，如果存在，该情况表示更新值
   1. 更新hash中该key对应的node的val
   2. 对该key执行一次get操作，表示更新一次使用频次
3. hash中不存在该key，则该情况表示新添加k-v
4. 首先判断当前lfu是否为满
   1. 如果满，则删除cache中最低频次minCnt对应的链表的第一个元素，因为在更新时，是从尾部添加的，因此第一个就是最久未使用的
   2. 从hash中delete该k-v
5. 构建新的node，将其添加到cache中频次为1的链表中
6. hash中添加该k-v
7. 将最低使用频次调整为1，因为首次插入时，视作一次操作，此时最低频次就是1

### code

```go
type node struct {
	prev, next    *node
	key, val, use int
}

type list struct {
	head, tail *node
	size       int
}

type LFUCache struct {
	cache  map[int]*list
	hash   map[int]*node
	minCnt int
	size   int
	cap    int
}

func makeList() *list {
	var link *list
	var head, tail *node
	head = &node{
		prev: nil,
		next: nil,
		key:  -1,
		val:  -1,
		use:  -1,
	}
	tail = &node{
		prev: nil,
		next: nil,
		key:  -1,
		val:  -1,
		use:  -1,
	}
	head.next = tail
	tail.prev = head
	link = &list{
		head: head,
		tail: tail,
		size: 0,
	}
	return link
}

func (l *list) remove(n *node) {
	n.prev.next = n.next
	n.next.prev = n.prev
	l.size--
}

func (l *list) removeFirst() int {
	if l != nil && l.size >= 1 {
		key := l.head.next.key
		l.head.next.next.prev = l.head
		l.head.next = l.head.next.next
		l.size--
		return key
	}
	return 0
}

func (l *list) addLast(n *node) {
	n.prev = l.tail.prev
	n.next = l.tail
	l.tail.prev.next = n
	l.tail.prev = n
	l.size++
}

func Constructor(capacity int) LFUCache {
	return LFUCache{
		cache:  make(map[int]*list),
		hash:   make(map[int]*node),
		minCnt: 2,
		size:   0,
		cap:    capacity,
	}
}

func (this *LFUCache) Get(key int) int {
	if node, ok := this.hash[key]; !ok {
		return -1
	} else {
		if this.minCnt == node.use && this.cache[node.use].size == 1 {
			this.minCnt = node.use + 1
		}
		// remove
		this.cache[node.use].remove(node)
		node.use++
		// addToLast
		if _, ok := this.cache[node.use]; !ok {
			this.cache[node.use] = makeList()
		}
		this.cache[node.use].addLast(node)
		return node.val
	}
}

func (this *LFUCache) Put(key int, value int) {
	if this.cap == 0 {
		return
	}
	if _, ok := this.hash[key]; ok {
		this.hash[key].val = value
		this.Get(key)
		return
	}
	if this.size >= this.cap {
		delKey := this.cache[this.minCnt].removeFirst()
		delete(this.hash, delKey)
	}
	node := &node{
		prev: nil,
		next: nil,
		key:  key,
		val:  value,
		use:  1,
	}
	if _, ok := this.cache[1]; !ok {
		this.cache[1] = makeList()
	}
	this.cache[1].addLast(node)
	this.hash[key] = node
	this.minCnt = 1
	this.size++
}

/**
 * Your LFUCache object will be instantiated and called as such:
 * obj := Constructor(capacity);
 * param_1 := obj.Get(key);
 * obj.Put(key,value);
 */
```

