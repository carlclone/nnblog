# LRU 线程安全和高并发 - Go 版本

## 主要实现手段

哈希表+双向链表+dummy

线程安全:读写锁

并发:分片



改写自课程的 Java 版本 , 分享在了 [leetcode discussion](https://leetcode.com/problems/lru-cache/discuss/896936/go-lru-cache-with-high-concurrency-and-thread-safe) 上

```go
package lru_cache

import "sync"

/*
thread safe: use Read/Write Lock on Get() / Put() / Size()
high concurrency : use sharding , noted that this code won't pass the leetcode test ,
the reason is the capacity of this cache is approximately , not absolutely , eviction won't act as leetcode expect

for example , concurrency=3 , approximate capacity=7 , 7/3=2 , 2 segments , every segment has 7/2=3 ,
we need extra 1 capacity , so add 1 at every segment 3+1=4 capacity ,
so the absolute capacity will be 4*2=8
 */


// segment lock inspired by java concurrent hashmap
// basically just sharding

type LRUCache struct {
	cacheSegs []*RegularLRUCache
}

func Constructor(cap int,concurrency int) LRUCache {

	segCap := cap / concurrency
	if cap&concurrency >= 1 {
		segCap++
	}

	// instantiate the segment
	segs := make([]*RegularLRUCache, concurrency)
	for i := 0; i < concurrency; i++ {
		lruv3 := ConstructorV3(segCap)
		segs[i] = &lruv3
	}

	ins := LRUCache{segs}
	return ins
}

func (this *LRUCache) Get(key int) int {
	return this.cache(key).Get(key)
}

func (this *LRUCache) Put(key int, value int) {
	this.cache(key).Put(key, value)
}

//get the segment index of this key
func (this *LRUCache) segmentIndex(key int) int {
	return this.hashcode(key) % len(this.cacheSegs)
}

// get the cache segment of this key
func (this *LRUCache) cache(key int) *RegularLRUCache {
	return this.cacheSegs[this.segmentIndex(key)]
}

func (this *LRUCache) hashcode(key int) int {
	//todo; just return origin value , not implemented yet
	return key
}

// regular RegularLRUCache implementation

type Node struct {
	next *Node
	prev *Node
	key  int
	val  int
}

type RegularLRUCache struct {
	table map[int]*Node
	head  *Node
	tail  *Node
	cap   int
	size  int
	mutex *sync.RWMutex
}

func ConstructorV3(capacity int) RegularLRUCache {
	head := &Node{}
	tail := &Node{}
	head.next = tail
	tail.prev = head

	return RegularLRUCache{make(map[int]*Node), head, tail, capacity, 0, &sync.RWMutex{}}
}

func (this *RegularLRUCache) Get(key int) int {
	this.mutex.Lock()
	defer this.mutex.Unlock()
	if node, ok := this.table[key]; ok {
		this.delNode(node)
		this.offerNode(node)
		return node.val
	}
	return -1
}

func (this *RegularLRUCache) Put(key int, value int) {
	this.mutex.Lock()
	defer this.mutex.Unlock()
	if node, ok := this.table[key]; ok {
		node.val = value
		this.delNode(node)
		this.offerNode(node)
	} else {
		if this.size+1 > this.cap {

			evict := this.tail.prev
			delete(this.table, evict.key)
			this.delNode(evict)
			this.size--
		}

		node := &Node{}
		node.key = key
		node.val = value
		this.table[key] = node
		this.offerNode(node)
		this.size++
	}
}

func (this *RegularLRUCache) delNode(node *Node) {
	prev := node.prev
	next := node.next
	prev.next = next
	next.prev = prev
}

func (this *RegularLRUCache) offerNode(node *Node) {
	next := this.head.next
	this.head.next = node
	node.next = next
	node.prev = this.head
	next.prev = node
}
```


## 资料

杨波 Java分布式课程