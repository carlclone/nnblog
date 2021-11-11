# mac 网易云音乐 cpu 占用高问题排查

## 问题描述 
`网易云音乐`在我的 macos 10.13.6 上运行半小时就会占用百分之 500 的 cpu, 找了一圈也没搜到类似问题 , 恰好最近学习了 linux cpu 问题排查

## 问题排查
- 打算先看看 cpu 各个指标,结果连 top 也不一样,只拿到个 pid 17125
- `perf` 的替代品 `sample 17125 -f output.prof`  `filtercalltree output.prof` , 得知有大量上下文切换函数和加锁解锁和文件操作
  
```
  Sort by top of stack, same collapsed (when >= 5):
        mach_msg_trap  (in libsystem_kernel.dylib)        22676         # 系统上下文切换的 trap
        __psynch_cvwait  (in libsystem_kernel.dylib)        6186        # 上下文切换等待
        __psynch_mutexwait  (in libsystem_kernel.dylib)        4828     # 加锁等待
        __workq_kernreturn  (in libsystem_kernel.dylib)        4124     # 上下文切换返回用户态
        __getattrlist  (in libsystem_kernel.dylib)        3670          # stat 加强版 (https://blog.csdn.net/kfy2011/article/details/50427878)
        __select  (in libsystem_kernel.dylib)        2062               # 多路复用那个?
        getattrlistbulk  (in libsystem_kernel.dylib)        792         # 类似 stat
        lstat$INODE64  (in libsystem_kernel.dylib)        620           # 类似 stat
        __psynch_mutexdrop  (in libsystem_kernel.dylib)        607      # 解锁
        __open  (in libsystem_kernel.dylib)        587                  # 打开文件
        close  (in libsystem_kernel.dylib)        251                   # 关闭文件
```
  
- 打开 `Activity Monitor.app` , 双击进程发现系统自带了 sample 功能
- 打开 `statistics` 视图的时候, 发现各项指标都好高
- 和 QQ 进程对比了一下,  mach 系统调用正常, 但是 上下文切换次数和 unix 系统调用超出了两个数量级, 有1亿次和6 亿次
  ![image-20191120140931373](/img/neteas-many-syscall.png)
  ![image-20191120140931373](/img/qq-syscall.png)
- 寻找 `strace` 的替代品, 看看系统调用 `sudo dtruss -p 17125`
- 发现不停有大量的`stat`系统调用和大量的加锁解锁操作,路径也显示出来了
```
getattrlist("/Users/lin/Library/Containers/com.netease.163music/Data/Caches/online_play_cache\0", 0x70000D03BD78, 0x70000D03B9D0)		 = 0 0
lstat64("/Users/lin/Library/Containers/com.netease.163music/Data/Caches/online_play_cache\0", 0x70000D247EB0, 0x0)		 = 0 0
getattrlist("/Users/lin/Library/Containers/com.netease.163music/Data/Caches/online_play_cache\0", 0x70000D0BEE90, 0x70000D0BF290)		 = 0 0
psynch_mutexdrop(0x10B68C070, 0xB1AB1703, 0xB1AB1400)		 = 0 0
psynch_mutexwait(0x10B68C070, 0xB1AB1503, 0xB1AB1100)		 = -1314187517 0
geteuid(0x0, 0x0, 0x0)		 = 501 0
psynch_mutexdrop(0x10B68C070, 0xB1AB1703, 0xB1AB1500)		 = 0 0
psynch_mutexwait(0x10B68C070, 0xB1AB1603, 0xB1AB1200)		 = -1314187517 0
getattrlist("/Users/lin/Library/Containers/com.netease.163music/Data/Caches/online_play_cache\0", 0x60C0001348EC, 0x7F8F3D926CB8)		 = 0 0
psynch_mutexdrop(0x10B68C070, 0xB1AB1903, 0xB1AB1600)		 = 0 0
getattrlist("/Users/lin/Library/Containers/com.netease.163music/Data/Caches/online_play_cache\0", 0x70000CD29E90, 0x70000CD2A290)		 = 0 0
getattrlist("/Users/lin/Library/Containers/com.netease.163music/Data/Caches/online_play_cache\0", 0x70000D247D78, 0x70000D2479D0)		 = 0 0
psynch_mutexwait(0x10B68C070, 0xB1AB1703, 0xB1AB1300)		 = -1314187005 0
lstat64("/Users/lin/Library/Containers/com.netease.163music/Data/Caches/online_play_cache\0", 0x70000CBA0EB0, 0x0)		 = 0 0
geteuid(0x0, 0x0, 0x0)		 = 501 0
getattrlist("/Users/lin/Library/Containers/com.netease.163music/Data/Caches/online_play_cache\0", 0x6040003248AC, 0x7F8F3E809AB8)		 = 0 0
psynch_mutexdrop(0x10B68C070, 0xB1AB1A03, 0xB1AB1700)		 = 0 0
psynch_mutexwait(0x10B68C070, 0xB1AB1803, 0xB1AB1500)		 = -1314186493 0
psynch_mutexdrop(0x10B68C070, 0xB1AB1B03, 0xB1AB1800)		 = 0 0
psynch_mutexwait(0x10B68C070, 0xB1AB1903, 0xB1AB1500)		 = -1314186493 0
getattrlist("/Users/lin/Library/Containers/com.netease.163music/Data/Caches/online_play_cache\0", 0x70000D1C4E90, 0x70000D1C5290)		 = 0 0
getattrlist("/Users/lin/Library/Containers/com.netease.163music/Data/Caches/online_play_cache\0", 0x70000CBA0D78, 0x70000CBA09D0)		 = 0 0
psynch_mutexdrop(0x10B68C070, 0xB1AB1D03, 0xB1AB1900)		 = 0 0
psynch_mutexwait(0x10B68C070, 0xB1AB1A03, 0xB1AB1600)		 = -1314185981 0
geteuid(0x0, 0x0, 0x0)		 = 501 0
psynch_mutexdrop(0x10B68C070, 0xB1AB1D03, 0xB1AB1A00)		 = 0 0
psynch_mutexwait(0x10B68C070, 0xB1AB1B03, 0xB1AB1700)		 = -1314185981 0
getattrlist("/Users/lin/Library/Containers/com.netease.163music/Data/Caches/online_play_cache\0", 0x70000CA9AE90, 0x70000CA9B290)		 = 0 0
getattrlist("/Users/lin/Library/Containers/com.netease.163music/Data/Caches/online_play_cache\0", 0x60800032264C, 0x7F8F3D0156B8)		 = 0 0
open("/Users/lin/Library/Containers/com.netease.163music/Data/Caches/online_play_cache\0", 0x1100004, 0x5)		 = 60 0
psynch_mutexdrop(0x10B68C070, 0xB1AB1E03, 0xB1AB1B00)		 = 0 0
getattrlistbulk(0x3C, 0x60C00031A8C8, 0x7F8F3D92B000)		 = 0 0
getattrlist("/Users/lin/Library/Containers/com.netease.163music/Data/Caches/online_play_cache\0", 0x604000322BEC, 0x7F8F3E8088B8)		 = 0 0
close(0x3C)		 = 0 0
psynch_mutexwait(0x10B68C070, 0xB1AB1C03, 0xB1AB1800)		 = -1314185725 0
psynch_mutexdrop(0x10B68C070, 0xB1AB2203, 0xB1AB1C00)		 = 0 0
psynch_mutexwait(0x10B68C070, 0xB1AB1D03, 0xB1AB1800)		 = -1314184701 0
psynch_mutexdrop(0x10B68C070, 0xB1AB2203, 0xB1AB1D00)		 = 0 0
psynch_mutexwait(0x10B68C070, 0xB1AB1E03, 0xB1AB1A00)		 = -1314184701 0
open("/Users/lin/Library/Containers/com.netease.163music/Data/Caches/online_play_cache\0", 0x1100004, 0x7)		 = 60 0
psynch_mutexdrop(0x10B68C070, 0xB1AB2203, 0xB1AB1E00)		 = 0 0
psynch_mutexwait(0x10B68C070, 0xB1AB1F03, 0xB1AB1B00)		 = -1314184701 0
getattrlist("/Users/lin/Library/Containers/com.netease.163music/Data/Caches/online_play_cache\0", 0x70000D03BE90, 0x70000D03C290)		 = 0 0
psynch_mutexdrop(0x10B68C070, 0xB1AB2303, 0xB1AB1F00)		 = 0 0
getattrlistbulk(0x3C, 0x604000300C78, 0x7F8F3E81F000)		 = 0 0
close(0x3C)		 = 0 0
getattrlist("/Users/lin/Library/Containers/com.netease.163music/Data/Caches/online_play_cache\0", 0x60800032228C, 0x7F8F3CA35AB8)		
```

- 基本可以确定是这个缓存的问题了,将缓存文件夹删了重建
- 跑了两小时,问题没有再出现

## 猜测

网易云音乐客户端从 4,5 年前一直更新过来, 可能是以前的旧缓存导致的问题? 或者是缓存损坏?  `\0` 看着像 C 语言里的字符串结束符

## 疑问

- 为什么系统调用高上下文切换也高呢?

因为系统调用需要从用户态陷入(trap)内核态,同样需要保存寄存器数据,切换页表, 参考 `6.S081 xv6` 课程和 `OSTEP`