init: starting sh
$ usertests
usertests starting
scause 0x000000000000000d
sepc=0x0000000080001d88 stval=0x000000021fdcc000
panic: kerneltrap
QEMU: Terminated

free 错 pagetable 


kernel.asm

80001d84:	01248f63          	beq	s1,s2,80001da2 <proc_freekpagetable+0x40>
pte_t p = level2[i];
80001d88:	6088                	ld	a0,0(s1)
if (p & PTE_V) {
80001d8a:	00157793          	andi	a5,a0,1







```
$ usertests
usertests starting
test execout: OK
test copyin: OK
test copyout: OK
test copyinstr1: OK
test copyinstr2: OK
test copyinstr3: OK
test truncate1: OK
test truncate2: OK
test truncate3: OK
test reparent2: OK
test pgbug: OK
test sbrkbugs: usertrap(): unexpected scause 0x000000000000000c pid=3234
            sepc=0x00000000000053fc stval=0x00000000000053fc
usertrap(): unexpected scause 0x000000000000000c pid=3235
            sepc=0x00000000000053fc stval=0x00000000000053fc
OK
test badarg: OK
test reparent: OK
test twochildren: OK
test forkfork: OK
test forkforkfork: OK
test argptest: OK
test createdelete: OK
test linkunlink: OK
test linktest: OK
test unlinkread: OK
test concreate: OK
test subdir: OK
test fourfiles: OK
test sharedfd: OK
test exectest: OK
test bigargtest: OK
test bigwrite: OK
test bsstest: OK
test sbrkbasic: OK
test sbrkmuch: OK
test kernmem: usertrap(): unexpected scause 0x000000000000000d pid=6214
            sepc=0x000000000000201a stval=0x0000000080000000
usertrap(): unexpected scause 0x000000000000000d pid=6215
            sepc=0x000000000000201a stval=0x000000008000c350
usertrap(): unexpected scause 0x000000000000000d pid=6216
            sepc=0x000000000000201a stval=0x00000000800186a0
usertrap(): unexpected scause 0x000000000000000d pid=6217
            sepc=0x000000000000201a stval=0x00000000800249f0
usertrap(): unexpected scause 0x000000000000000d pid=6218
            sepc=0x000000000000201a stval=0x0000000080030d40
usertrap(): unexpected scause 0x000000000000000d pid=6219
            sepc=0x000000000000201a stval=0x000000008003d090
usertrap(): unexpected scause 0x000000000000000d pid=6220
            sepc=0x000000000000201a stval=0x00000000800493e0
usertrap(): unexpected scause 0x000000000000000d pid=6221
            sepc=0x000000000000201a stval=0x0000000080055730
usertrap(): unexpected scause 0x000000000000000d pid=6222
            sepc=0x000000000000201a stval=0x0000000080061a80
usertrap(): unexpected scause 0x000000000000000d pid=6223
            sepc=0x000000000000201a stval=0x000000008006ddd0
usertrap(): unexpected scause 0x000000000000000d pid=6224
            sepc=0x000000000000201a stval=0x000000008007a120
usertrap(): unexpected scause 0x000000000000000d pid=6225
            sepc=0x000000000000201a stval=0x0000000080086470
usertrap(): unexpected scause 0x000000000000000d pid=6226
            sepc=0x000000000000201a stval=0x00000000800927c0
usertrap(): unexpected scause 0x000000000000000d pid=6227
            sepc=0x000000000000201a stval=0x000000008009eb10
usertrap(): unexpected scause 0x000000000000000d pid=6228
            sepc=0x000000000000201a stval=0x00000000800aae60
usertrap(): unexpected scause 0x000000000000000d pid=6229
            sepc=0x000000000000201a stval=0x00000000800b71b0
usertrap(): unexpected scause 0x000000000000000d pid=6230
            sepc=0x000000000000201a stval=0x00000000800c3500
usertrap(): unexpected scause 0x000000000000000d pid=6231
            sepc=0x000000000000201a stval=0x00000000800cf850
usertrap(): unexpected scause 0x000000000000000d pid=6232
            sepc=0x000000000000201a stval=0x00000000800dbba0
usertrap(): unexpected scause 0x000000000000000d pid=6233
            sepc=0x000000000000201a stval=0x00000000800e7ef0
usertrap(): unexpected scause 0x000000000000000d pid=6234
            sepc=0x000000000000201a stval=0x00000000800f4240
usertrap(): unexpected scause 0x000000000000000d pid=6235
            sepc=0x000000000000201a stval=0x0000000080100590
usertrap(): unexpected scause 0x000000000000000d pid=6236
            sepc=0x000000000000201a stval=0x000000008010c8e0
usertrap(): unexpected scause 0x000000000000000d pid=6237
            sepc=0x000000000000201a stval=0x0000000080118c30
usertrap(): unexpected scause 0x000000000000000d pid=6238
            sepc=0x000000000000201a stval=0x0000000080124f80
usertrap(): unexpected scause 0x000000000000000d pid=6239
            sepc=0x000000000000201a stval=0x00000000801312d0
usertrap(): unexpected scause 0x000000000000000d pid=6240
            sepc=0x000000000000201a stval=0x000000008013d620
usertrap(): unexpected scause 0x000000000000000d pid=6241
            sepc=0x000000000000201a stval=0x0000000080149970
usertrap(): unexpected scause 0x000000000000000d pid=6242
            sepc=0x000000000000201a stval=0x0000000080155cc0
usertrap(): unexpected scause 0x000000000000000d pid=6243
            sepc=0x000000000000201a stval=0x0000000080162010
usertrap(): unexpected scause 0x000000000000000d pid=6244
            sepc=0x000000000000201a stval=0x000000008016e360
usertrap(): unexpected scause 0x000000000000000d pid=6245
            sepc=0x000000000000201a stval=0x000000008017a6b0
usertrap(): unexpected scause 0x000000000000000d pid=6246
            sepc=0x000000000000201a stval=0x0000000080186a00
usertrap(): unexpected scause 0x000000000000000d pid=6247
            sepc=0x000000000000201a stval=0x0000000080192d50
usertrap(): unexpected scause 0x000000000000000d pid=6248
            sepc=0x000000000000201a stval=0x000000008019f0a0
usertrap(): unexpected scause 0x000000000000000d pid=6249
            sepc=0x000000000000201a stval=0x00000000801ab3f0
usertrap(): unexpected scause 0x000000000000000d pid=6250
            sepc=0x000000000000201a stval=0x00000000801b7740
usertrap(): unexpected scause 0x000000000000000d pid=6251
            sepc=0x000000000000201a stval=0x00000000801c3a90
usertrap(): unexpected scause 0x000000000000000d pid=6252
            sepc=0x000000000000201a stval=0x00000000801cfde0
usertrap(): unexpected scause 0x000000000000000d pid=6253
            sepc=0x000000000000201a stval=0x00000000801dc130
OK
test sbrkfail: usertrap(): unexpected scause 0x000000000000000d pid=6265
            sepc=0x0000000000003e76 stval=0x0000000000012000
OK
test sbrkarg: OK
test validatetest: OK
test stacktest: usertrap(): unexpected scause 0x000000000000000d pid=6269
            sepc=0x0000000000002188 stval=0x000000000000fbc0
OK
test opentest: OK
test writetest: OK
test writebig: OK
test createtest: OK
test openiput: OK
test exitiput: OK
test iput: OK
test mem: OK
test pipe1: OK
test preempt: kill... wait... OK
test exitwait: OK
test rmdot: OK
test fourteen: OK
test bigfile: OK
test dirfile: OK
test iref: OK
test forktest: OK
test bigdir: OK
ALL TESTS PASSED
```






for(int i = 0; i < max && srcva + i < p->sz; i++){
80006542:	cc85                	beqz	s1,8000657a <copyinstr_new+0x62>
80006544:	00990833          	add	a6,s2,s1
80006548:	87ca                	mv	a5,s2
8000654a:	6538                	ld	a4,72(a0)
8000654c:	00e7ff63          	bgeu	a5,a4,8000656a <copyinstr_new+0x52>
dst[i] = s[i];
80006550:	0007c683          	lbu	a3,0(a5)
80006554:	41278733          	sub	a4,a5,s2
80006558:	974e                	add	a4,a4,s3



init: starting sh
scause 0x000000000000000d
sepc=0x0000000080006550 stval=0x0000000000000968
panic: kerneltrap


fork() bug:
ukvmcopy(np->kpagetable, np->pagetable, np->sz, p->sz);
ukvmcopy(np->kpagetable, np->pagetable, np->sz, 0);




== Test usertests == Timeout! (300.2s)
== Test   usertests: copyin ==
usertests: copyin: FAIL
Failed copyin
== Test   usertests: copyinstr1 ==
usertests: copyinstr1: FAIL
Failed copyinstr1
== Test   usertests: copyinstr2 ==
usertests: copyinstr2: FAIL
Failed copyinstr2
== Test   usertests: copyinstr3 ==
usertests: copyinstr3: FAIL
Failed copyinstr3
== Test   usertests: sbrkmuch ==
usertests: sbrkmuch: FAIL
Failed sbrkmuch
== Test   usertests: all tests ==
usertests: all tests: FAIL (0.6s)
...
copy success
copy success
copy success
copy success
copy suqemu-system-riscv64: terminating on signal 15 from pid 92624 (<unknown process>)
MISSING '^ALL TESTS PASSED$'
== Test time ==
time: FAIL
Cannot read time.txt
Score: 15/66