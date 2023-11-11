#  Linux Command pmap 测试进程内存映射工具




--------
## 1. 简介
pmap命令用于报告进程的内存映射关系，是Linux调试及运维一个很好的工具。

## 2. 语法

```bash
 pmap [ -x | -d ] [ -q ] pids...
    pmap -V
```

## 3. 参数

```bash
 -x  extended    Show the extended format. 显示扩展格式
    -d  device     Show the device format.  显示设备格式
    -q  quiet     Do not display some header/footer lines. 不显示头尾行
    -V  show version  Displays version of program. 显示版本
```

扩展格式和设备格式域：

```bash
Address: start address of map 映像起始地址
    Kbytes: size of map in kilobytes 映像大小
    RSS: resident set size in kilobytes 驻留集大小
    Dirty: dirty pages (both shared and private) in kilobytes 脏页大小
    Mode: permissions on map 映像权限: r=read, w=write, x=execute, s=shared, p=private (copy on write) 
    Mapping: file backing the map , or '[ anon ]' for allocated memory, or '[ stack ]' for the program stack. 映像支持文件,[anon]为已分配内存 [stack]为程序堆栈
    Offset: offset into the file 文件偏移
    Device: device name (major:minor) 设备名
```

##  4. 实例
### 4.1 查看进程1设备格式

```bash
[root@C44 ~]# pmap -d 1
1:  init [5]          
Address  Kbytes Mode Offset      Device  Mapping
00934000   88 r-x-- 0000000000000000 008:00005 ld-2.3.4.so
0094a000    4 r---- 0000000000015000 008:00005 ld-2.3.4.so
0094b000    4 rw--- 0000000000016000 008:00005 ld-2.3.4.so
0094e000  1188 r-x-- 0000000000000000 008:00005 libc-2.3.4.so
00a77000    8 r---- 0000000000129000 008:00005 libc-2.3.4.so
00a79000    8 rw--- 000000000012b000 008:00005 libc-2.3.4.so
00a7b000    8 rw--- 0000000000a7b000 000:00000  [ anon ]
00a85000   52 r-x-- 0000000000000000 008:00005 libsepol.so.1
00a92000    4 rw--- 000000000000c000 008:00005 libsepol.so.1
00a93000   32 rw--- 0000000000a93000 000:00000  [ anon ]
00d9d000   52 r-x-- 0000000000000000 008:00005 libselinux.so.1
00daa000    4 rw--- 000000000000d000 008:00005 libselinux.so.1
08048000   28 r-x-- 0000000000000000 008:00005 init
0804f000    4 rw--- 0000000000007000 008:00005 init
084e1000   132 rw--- 00000000084e1000 000:00000  [ anon ]
b7f5d000    8 rw--- 00000000b7f5d000 000:00000  [ anon ]
bffee000   72 rw--- 00000000bffee000 000:00000  [ stack ]
ffffe000    4 ----- 0000000000000000 000:00000  [ anon ]
mapped: 1700K  writeable/private: 276K  shared: 0K
```
最后一行的值

 - mapped 表示该进程映射的虚拟地址空间大小，也就是该进程预先分配的虚拟内存大小，即ps出的vsz
 - writeable/private 表示进程所占用的私有地址空间大小，也就是该进程实际使用的内存大小
 - shared 表示进程和其他进程共享的内存大小

### 4.2 查看进程1设备格式不显示头尾行

```bash
[root@C44 ~]# pmap -d -q 1
1:  init [5]          
00934000   88 r-x-- 0000000000000000 008:00005 ld-2.3.4.so
0094a000    4 r---- 0000000000015000 008:00005 ld-2.3.4.so
0094b000    4 rw--- 0000000000016000 008:00005 ld-2.3.4.so
0094e000  1188 r-x-- 0000000000000000 008:00005 libc-2.3.4.so
00a77000    8 r---- 0000000000129000 008:00005 libc-2.3.4.so
00a79000    8 rw--- 000000000012b000 008:00005 libc-2.3.4.so
00a7b000    8 rw--- 0000000000a7b000 000:00000  [ anon ]
00a85000   52 r-x-- 0000000000000000 008:00005 libsepol.so.1
00a92000    4 rw--- 000000000000c000 008:00005 libsepol.so.1
00a93000   32 rw--- 0000000000a93000 000:00000  [ anon ]
00d9d000   52 r-x-- 0000000000000000 008:00005 libselinux.so.1
00daa000    4 rw--- 000000000000d000 008:00005 libselinux.so.1
08048000   28 r-x-- 0000000000000000 008:00005 init
0804f000    4 rw--- 0000000000007000 008:00005 init
084e1000   132 rw--- 00000000084e1000 000:00000  [ anon ]
b7f5d000    8 rw--- 00000000b7f5d000 000:00000  [ anon ]
bffee000   72 rw--- 00000000bffee000 000:00000  [ stack ]
ffffe000    4 ----- 0000000000000000 000:00000  [ anon ]
```

###  4.3 查看进程1扩展格式 

```bash
[root@C44 ~]# pmap -x 1
1:  init [5]          
Address  Kbytes   RSS  Anon Locked Mode  Mapping
00934000   88    -    -    - r-x-- ld-2.3.4.so
0094a000    4    -    -    - r---- ld-2.3.4.so
0094b000    4    -    -    - rw--- ld-2.3.4.so
0094e000  1188    -    -    - r-x-- libc-2.3.4.so
00a77000    8    -    -    - r---- libc-2.3.4.so
00a79000    8    -    -    - rw--- libc-2.3.4.so
00a7b000    8    -    -    - rw---  [ anon ]
00a85000   52    -    -    - r-x-- libsepol.so.1
00a92000    4    -    -    - rw--- libsepol.so.1
00a93000   32    -    -    - rw---  [ anon ]
00d9d000   52    -    -    - r-x-- libselinux.so.1
00daa000    4    -    -    - rw--- libselinux.so.1
08048000   28    -    -    - r-x-- init
0804f000    4    -    -    - rw--- init
084e1000   132    -    -    - rw---  [ anon ]
b7f5d000    8    -    -    - rw---  [ anon ]
bffee000   72    -    -    - rw---  [ stack ]
ffffe000    4    -    -    - -----  [ anon ]
-------- ------- ------- ------- -------
total kB  1700    -    -    -
```
###  4.4 循环显示进程3066设备格式最后1行
间隔2秒

```bash
[root@C44 ~]# while true; do pmap -d 3066 | tail -1; sleep 2; done
mapped: 5412K  writeable/private: 2028K  shared: 0K
mapped: 5412K  writeable/private: 2028K  shared: 0K
mapped: 5412K  writeable/private: 2028K  shared: 0K
mapped: 5412K  writeable/private: 2028K  shared: 0K
mapped: 5412K  writeable/private: 2028K  shared: 0K
mapped: 5412K  writeable/private: 2028K  shared: 0K
mapped: 5412K  writeable/private: 2028K  shared: 0K
mapped: 5412K  writeable/private: 2028K  shared: 0K
mapped: 5412K  writeable/private: 2028K  shared: 0K
mapped: 5412K  writeable/private: 2028K  shared: 0K
mapped: 5412K  writeable/private: 2028K  shared: 0K
mapped: 5412K  writeable/private: 2028K  shared: 0K
mapped: 5412K  writeable/private: 2028K  shared: 0K
```
更多阅读：

 - [pmap command in Linux with Examples](https://www.geeksforgeeks.org/pmap-command-in-linux-with-examples/)

