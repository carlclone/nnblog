优化代码, memset 耗时并且可以并发执行 , 减小锁粒度 kfree
// Free the page of physical memory pointed at by v,
// which normally should have been returned by a
// call to kalloc().  (The exception is when
// initializing the allocator; see kinit above.)
void
kfree(void *pa)
{
struct run *r;

if(((uint64)pa % PGSIZE) != 0 || (char*)pa < end || (uint64)pa >= PHYSTOP)
panic("kfree");


r = (struct run*)pa;

acquire(&kmem.lock);
int paref = incr((uint64)pa,-1);
release(&kmem.lock);

if (paref==0) {
memset(pa, 1, PGSIZE);
}

acquire(&kmem.lock);
if (paref==0) {
// Fill with junk to catch dangling refs.
// (DONE) TODO; this is time consume , reduce the lock granularity
//      memset(pa, 1, PGSIZE);
r->next = kmem.freelist;
kmem.freelist = r;
}

release(&kmem.lock);
}



测试用例直接写在代码里

比如

kalloc(void)
{
struct run *r;

acquire(&kmem.lock);
r = kmem.freelist;
if(r) {
kmem.freelist = r->next;
if (incr((uint64(r),1)) !=1) {
panic("kalloc incr error");
}
}


actual , expected
           1





== Test   usertests: copyin ==
usertests: copyin: FAIL
Parent failed: test_usertests
== Test   usertests: copyout ==
usertests: copyout: FAIL
Parent failed: test_usertests
== Test   usertests: all tests ==
usertests: all tests: FAIL
Parent failed: test_usertests
== Test time ==
time: FAIL 




出 bug 了 , execout 执行太久, 先注释了

test truncate3: OK
test reparent2: usertrap(): unexpected scause 0x0000000000000002 pid=16
usertrap(): unexpected scause 0x0000000000000002 pid=2853
sepc=0x0000000000005520 stval=0x0000000000000000
sepc=0x0000000000005520 stval=0x0000000000000000
usertrap(): unexpected scause 0x0000000000000002 pid=3
sepc=0x0000000000005530 stval=0x0000000000000000
scause 0x000000000000000d
sepc=0x00000000800015fa stval=0x0404040404040000
panic: kerneltrap
QEMU: Terminated

15fa 的地方 freewalk

800015d2:	e44e                	sd	s3,8(sp)
800015d4:	e052                	sd	s4,0(sp)
800015d6:	1800                	addi	s0,sp,48
800015d8:	8a2a                	mv	s4,a0
// there are 2^9 = 512 PTEs in a page table.
for(int i = 0; i < 512; i++){
800015da:	84aa                	mv	s1,a0
800015dc:	6905                	lui	s2,0x1
800015de:	992a                	add	s2,s2,a0
pte_t pte = pagetable[i];
if((pte & PTE_V) && (pte & (PTE_R|PTE_W|PTE_X)) == 0){
800015e0:	4985                	li	s3,1
800015e2:	a821                	j	800015fa <freewalk+0x32>
// this PTE points to a lower-level page table.
uint64 child = PTE2PA(pte);
800015e4:	8129                	srli	a0,a0,0xa
freewalk((pagetable_t)child);
800015e6:	0532                	slli	a0,a0,0xc
800015e8:	00000097          	auipc	ra,0x0
800015ec:	fe0080e7          	jalr	-32(ra) # 800015c8 <freewalk>
pagetable[i] = 0;
800015f0:	0004b023          	sd	zero,0(s1)
for(int i = 0; i < 512; i++){
800015f4:	04a1                	addi	s1,s1,8
800015f6:	03248163          	beq	s1,s2,80001618 <freewalk+0x50>
pte_t pte = pagetable[i];
800015fa:	6088                	ld	a0,0(s1)
if((pte & PTE_V) && (pte & (PTE_R|PTE_W|PTE_X)) == 0){
800015fc:	00f57793          	andi	a5,a0,15
80001600:	ff3782e3          	beq	a5,s3,800015e4 <freewalk+0x1c>
} else if(pte & PTE_V){
80001604:	8905                	andi	a0,a0,1
80001606:	d57d                	beqz	a0,800015f4 <freewalk+0x2c>
panic("freewalk: leaf");



sudo dtruss -p 74312

psynch_mutexdrop(0x106DF1978, 0x76402903, 0x76402800)		 = 0 0
psynch_mutexwait(0x106DF1978, 0x76402903, 0x76402700)		 = 1983916291 0
psynch_mutexdrop(0x106DF1978, 0x76402A03, 0x76402900)		 = 0 0
psynch_mutexwait(0x106DF1978, 0x76402A03, 0x76402800)		 = 1983916547 0
psynch_mutexdrop(0x106DF1978, 0x76404603, 0x76404500)		 = 0 0
psynch_mutexwait(0x106DF1978, 0x76404603, 0x76404400)		 = 1983923715 0
psynch_mutexdrop(0x106DF1978, 0x76404703, 0x76404600)		 = 0 0
psynch_mutexwait(0x106DF1978, 0x76404703, 0x76404500)		 = 1983923971 0
psynch_mutexdrop(0x106DF1978, 0x76404803, 0x76404700)		 = 0 0
psynch_mutexwait(0x106DF1978, 0x76404803, 0x76404600)		 = 1983924227 0
psynch_mutexdrop(0x106DF1978, 0x76404903, 0x76404800)		 = 0 0
psynch_mutexwait(0x106DF1978, 0x76404903, 0x76404700)		 = 1983924483 0
psynch_mutexdrop(0x106DF1978, 0x76404A03, 0x76404900)		 = 0 0
psynch_mutexwait(0x106DF1978, 0x76404A03, 0x76404800)		 = 1983924739 0
psynch_mutexdrop(0x106DF1978, 0x76404B03, 0x76404A00)		 = 0 0
psynch_mutexwait(0x106DF1978, 0x76404B03, 0x76404900)		 = 1983924995 0
psynch_mutexdrop(0x106DF1978, 0x76404C03, 0x76404B00)		 = 0 0
psynch_mutexwait(0x106DF1978, 0x76404C03, 0x76404A00)		 = 1983925251 0
psynch_mutexdrop(0x106DF1978, 0x76405803, 0x76405700)		 = 0 0
psynch_mutexwait(0x106DF1978, 0x76405803, 0x76405600)		 = 1983928323 0
psynch_mutexdrop(0x106DF1978, 0x76405903, 0x76405800)		 = 0 0
psynch_mutexwait(0x106DF1978, 0x76405903, 0x76405700)		 = 1983928579 0
psynch_mutexdrop(0x106DF1978, 0x76405A03, 0x76405900)		 = 0 0
psynch_mutexwait(0x106DF1978, 0x76405A03, 0x76405800)		 = 1983928835 0
psynch_mutexdrop(0x106DF1978, 0x76405B03, 0x76405A00)		 = 0 0
psynch_mutexwait(0x106DF1978, 0x76405B03, 0x76405900)		 = 1983929091 0
psynch_mutexdrop(0x106DF1978, 0x76405C03, 0x76405B00)		 = 0 0
psynch_mutexwait(0x106DF1978, 0x76405C03, 0x76405A00)		 = 1983929347 0
psynch_mutexdrop(0x106DF1978, 0x76405D03, 0x76405C00)		 = 0 0
psynch_mutexwait(0x106DF1978, 0x76405D03, 0x76405B00)		 = 1983929603 0
psynch_mutexdrop(0x106DF1978, 0x76405E03, 0x76405D00)		 = 0 0
psynch_mutexwait(0x106DF1978, 0x76405E03, 0x76405C00)		 = 1983929859 0



只能 fork 一次

hart 1 starting
hart 2 starting
init: starting sh
$ usertests kernmem
usertests starting
test kernmem: aaaaaaaaawbaibbtbibngbbbkkkkxxxxxxxxxxxxxxx
x
QEMU: Terminated




又是弱类型的坑, 把 int 当做 uint64 了 return -1变成>0 的数


在内核里的cow , 因为不是 pgfault 触发的, 所以不是 cow 的也不需要 crash 掉 ! 把自己恶心到了

在用户态, 除了 cow 的 page fault 是可恢复的, 其他都直接 crash 掉

每个 printf error 的时候也打出 backtrace , 就好了

由于不是直接在 copyout crash 的, 错误往上传(都是-1) , 没办法识别是什么错误

同时因为有用户态的非法触发 pgfault测试用例 , 没想到内核态的问题 , 被掩盖了

之后 每部分代码都要先加注释写明定义, 然后再开写

理解问题 -> 参考别人 github.io -> 形成计划(加注释) -> 执行计划(写代码) -> 回顾解法(总结)