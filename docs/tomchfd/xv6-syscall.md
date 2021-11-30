
 为什么 exit 系统调用没有 trace 到
 
因为 exit syscall 没有 return , 永远不 return , 直到被父进程回收