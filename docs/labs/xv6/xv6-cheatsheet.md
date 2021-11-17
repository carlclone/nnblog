# xv6 速查表

## 常用命令

每次调试要 make clean 清除编译缓存

make grade 全部成绩

./grade-lab-util pingpong

make GRADEFLAGS=pingpong grade

## c 语法

神书: K&R The C Programming Language

char *argv[] , 可以通过 argv[下标] 来访问

int abc[2]  定义并开辟 size 为 2 的数组 , 可通过 abc[0]访问

struct stat st 定义名为 st 的 stat 结构体

static char buf[DIRSIZ+1] 静态变量, 生命周期为整个运行期 , buf 名可以作为数组指针



## xv6 function

stat 结构体

st_mode 文件权限和文件类型信息
st_ino   与该文件关联的inode
st_dev   保存文件的设备
st_uid   文件属主的UID号
st_gid   文件属主的GID号
st_atime 文件上一次被访问的时间
st_ctime 文件的权限、属主、组或内容上一次被修改的时间
st_mtime 文件的内容上一次被修改的时间。（和st_ctime的不同之处显而易见）
st_nlink  该文件上硬连接的个数

fstat(fd, &st) 通过描述符获取文件信息
stat(filename,&st)  通过文件名获取文件信息

fd = open(path,O_FLAGS) , fd<0异常

```
1）O_RDONLY：以只读方式打开；

2）O_WRONLY：以只写方式打开；

3）O_RDWR ：以读写方式打开。

　　另外，还有以下几种可选模式的组合（ 用按位或 || 来操作 ）：

4）O_APPEND：把写入数据追加在文件的末尾；

5）O_TRUNC：把文件长度设置为零，丢弃已有的内容；

6）O_CREAT：如果需要，就按照参数 mode 中给出的访问模式创建文件；
....
```

fprintf 打印 , 参数 (描述符, 字节数组)
atoi(int) 字节数组->int

int fd_arr[2]
pipe(fd_arr)  创建管道
close() 关闭描述符
fork()==0 子进程 , 负值错误 , 正值父进程
wait(整数指针) , 如 wait((int *)0) 等待所有子进程
sizeof(int) int 的字节

返回的字节数 = read(描述符,字节数组,要读的字节数) , 如果描述符是个目录(目录也是文件,指向一块包含其中文件信息的字节block) , 可以这样遍历读 while(read(fd, &de, sizeof(de)) == sizeof(de)){
write(描述符,字节数组,要写的字节数)

"sadas" 也是字节数组

printf(字符串) fprintf()的封装,写到标准输出
getpid() 获取进程 id
dup()
strcpm() , ==不能对比字符串,而是对比指针值
strlen() 获取字符串(以\0 结尾的)长度  (sizeof 求数据类型所占空间)
memmove(dest,src,bytesize) 复制内容 http://c.biancheng.net/cpp/html/156.html

memset(dst,'字节',前几个)
```
memset(str,'$',7);
This is string.h library function
$$$$$$$ string.h library function

```

strcpy(dst,src) 顾名思义

strchr(str,ch) 返回第一次出现 ch 的指针 (指向 ch)
