
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/88e284eee41f9cf857ff11e32fe48301.png)
[下载nginx](http://nginx.org/)

```shell
wget http://nginx.org/download/nginx-1.20.2.tar.gz
tar -zxvf nginx-1.20.2.tar.gz
```
## 配置介绍

```bash
$ ls -l
总用量 788
drwxr-xr-x 6 1001 1001    326 2月   7 11:10 auto
-rw-r--r-- 1 1001 1001 312251 11月 16 22:44 CHANGES  #版本特性
-rw-r--r-- 1 1001 1001 476577 11月 16 22:44 CHANGES.ru  #俄罗斯版本特性说明
drwxr-xr-x 2 1001 1001    168 2月   7 11:10 conf  #配置文件
-rwxr-xr-x 1 1001 1001   2590 11月 16 22:44 configure  #编译前必备动作
drwxr-xr-x 4 1001 1001     72 2月   7 11:10 contrib  #提供vim色彩
drwxr-xr-x 2 1001 1001     40 2月   7 11:10 html  #默认html文件
-rw-r--r-- 1 1001 1001   1397 11月 16 22:44 LICENSE
drwxr-xr-x 2 1001 1001     21 2月   7 11:10 man  #帮助文件
-rw-r--r-- 1 1001 1001     49 11月 16 22:44 README
drwxr-xr-x 9 1001 1001     91 2月   7 11:10 src #源代码

#auto目录有四个子目录：cc编译目录、lib依赖库、os操作系统的判断模块的使用
$ ls -l auto/
总用量 188
drwxr-xr-x  2 1001 1001   133 2月   7 11:10 cc
-rw-r--r--  1 1001 1001   141 11月 16 22:44 define
-rw-r--r--  1 1001 1001   889 11月 16 22:44 endianness
-rw-r--r--  1 1001 1001  2812 11月 16 22:44 feature
-rw-r--r--  1 1001 1001   136 11月 16 22:44 have
-rw-r--r--  1 1001 1001   137 11月 16 22:44 have_headers
-rw-r--r--  1 1001 1001   411 11月 16 22:44 headers
-rw-r--r--  1 1001 1001  1020 11月 16 22:44 include
-rw-r--r--  1 1001 1001   768 11月 16 22:44 init
-rw-r--r--  1 1001 1001  4875 11月 16 22:44 install
drwxr-xr-x 11 1001 1001   163 2月   7 11:10 lib
-rw-r--r--  1 1001 1001 18324 11月 16 22:44 make
-rw-r--r--  1 1001 1001  3934 11月 16 22:44 module
-rw-r--r--  1 1001 1001 38640 11月 16 22:44 modules
-rw-r--r--  1 1001 1001   136 11月 16 22:44 nohave
-rw-r--r--  1 1001 1001 25469 11月 16 22:44 options
drwxr-xr-x  2 1001 1001    88 2月   7 11:10 os
-rw-r--r--  1 1001 1001  8752 11月 16 22:44 sources
-rw-r--r--  1 1001 1001   120 11月 16 22:44 stubs
-rw-r--r--  1 1001 1001  2014 11月 16 22:44 summary
-rw-r--r--  1 1001 1001   394 11月 16 22:44 threads
drwxr-xr-x  2 1001 1001    65 2月   7 11:10 types
-rw-r--r--  1 1001 1001 26166 11月 16 22:44 unix


#提供色彩方法
$ mkdir ~/.vim 
$ cp -r contrib/vim/* ~/.vim/

#两个html标准文件
$ tree  html/
html/
├── 50x.html
└── index.html

```
## 编译
查看帮助
```bash
$ ./configure --help

$ ./configure --prefix=/opt/nginx
checking for OS
 + Linux 3.10.0-1160.49.1.el7.x86_64 x86_64
checking for C compiler ... found
 + using GNU C compiler
 + gcc version: 4.8.5 20150623 (Red Hat 4.8.5-44) (GCC) 
checking for gcc -pipe switch ... found
checking for -Wl,-E switch ... found
checking for gcc builtin atomic operations ... found
checking for C99 variadic macros ... found
checking for gcc variadic macros ... found
checking for gcc builtin 64 bit byteswap ... found
checking for unistd.h ... found
checking for inttypes.h ... found
checking for limits.h ... found
checking for sys/filio.h ... not found
checking for sys/param.h ... found
checking for sys/mount.h ... found
checking for sys/statvfs.h ... found
checking for crypt.h ... found
checking for Linux specific features
checking for epoll ... found
checking for EPOLLRDHUP ... found
checking for EPOLLEXCLUSIVE ... not found
checking for eventfd() ... found
checking for O_PATH ... found
checking for sendfile() ... found
checking for sendfile64() ... found
checking for sys/prctl.h ... found
checking for prctl(PR_SET_DUMPABLE) ... found
checking for prctl(PR_SET_KEEPCAPS) ... found
checking for capabilities ... found
checking for crypt_r() ... found
checking for sys/vfs.h ... found
checking for nobody group ... found
checking for poll() ... found
checking for /dev/poll ... not found
checking for kqueue ... not found
checking for crypt() ... not found
checking for crypt() in libcrypt ... found
checking for F_READAHEAD ... not found
checking for posix_fadvise() ... found
checking for O_DIRECT ... found
checking for F_NOCACHE ... not found
checking for directio() ... not found
checking for statfs() ... found
checking for statvfs() ... found
checking for dlopen() ... not found
checking for dlopen() in libdl ... found
checking for sched_yield() ... found
checking for sched_setaffinity() ... found
checking for SO_SETFIB ... not found
checking for SO_REUSEPORT ... found
checking for SO_ACCEPTFILTER ... not found
checking for SO_BINDANY ... not found
checking for IP_TRANSPARENT ... found
checking for IP_BINDANY ... not found
checking for IP_BIND_ADDRESS_NO_PORT ... found
checking for IP_RECVDSTADDR ... not found
checking for IP_SENDSRCADDR ... not found
checking for IP_PKTINFO ... found
checking for IPV6_RECVPKTINFO ... found
checking for TCP_DEFER_ACCEPT ... found
checking for TCP_KEEPIDLE ... found
checking for TCP_FASTOPEN ... found
checking for TCP_INFO ... found
checking for accept4() ... found
checking for int size ... 4 bytes
checking for long size ... 8 bytes
checking for long long size ... 8 bytes
checking for void * size ... 8 bytes
checking for uint32_t ... found
checking for uint64_t ... found
checking for sig_atomic_t ... found
checking for sig_atomic_t size ... 4 bytes
checking for socklen_t ... found
checking for in_addr_t ... found
checking for in_port_t ... found
checking for rlim_t ... found
checking for uintptr_t ... uintptr_t found
checking for system byte ordering ... little endian
checking for size_t size ... 8 bytes
checking for off_t size ... 8 bytes
checking for time_t size ... 8 bytes
checking for AF_INET6 ... found
checking for setproctitle() ... not found
checking for pread() ... found
checking for pwrite() ... found
checking for pwritev() ... found
checking for strerrordesc_np() ... not found
checking for sys_nerr ... found
checking for localtime_r() ... found
checking for clock_gettime(CLOCK_MONOTONIC) ... found
checking for posix_memalign() ... found
checking for memalign() ... found
checking for mmap(MAP_ANON|MAP_SHARED) ... found
checking for mmap("/dev/zero", MAP_SHARED) ... found
checking for System V shared memory ... found
checking for POSIX semaphores ... not found
checking for POSIX semaphores in libpthread ... found
checking for struct msghdr.msg_control ... found
checking for ioctl(FIONBIO) ... found
checking for ioctl(FIONREAD) ... found
checking for struct tm.tm_gmtoff ... found
checking for struct dirent.d_namlen ... not found
checking for struct dirent.d_type ... found
checking for sysconf(_SC_NPROCESSORS_ONLN) ... found
checking for sysconf(_SC_LEVEL1_DCACHE_LINESIZE) ... found
checking for openat(), fstatat() ... found
checking for getaddrinfo() ... found
checking for PCRE library ... found
checking for PCRE JIT support ... found
checking for zlib library ... found
creating objs/Makefile

Configuration summary
  + using system PCRE library
  + OpenSSL library is not used
  + using system zlib library

  nginx path prefix: "/opt/nginx"
  nginx binary file: "/opt/nginx/sbin/nginx"
  nginx modules path: "/opt/nginx/modules"
  nginx configuration prefix: "/opt/nginx/conf"
  nginx configuration file: "/opt/nginx/conf/nginx.conf"
  nginx pid file: "/opt/nginx/logs/nginx.pid"
  nginx error log file: "/opt/nginx/logs/error.log"
  nginx http access log file: "/opt/nginx/logs/access.log"
  nginx http client request body temporary files: "client_body_temp"
  nginx http proxy temporary files: "proxy_temp"
  nginx http fastcgi temporary files: "fastcgi_temp"
  nginx http uwsgi temporary files: "uwsgi_temp"
  nginx http scgi temporary files: "scgi_temp"


```
编译后会生成中间件文件，在`objs`目录下，动态文件也会放置到`objs`，以及编译后文件放到`objs/src`目录下

```bash
$ ls -l objs/
总用量 80
-rw-r--r-- 1 root root 17882 2月   7 12:04 autoconf.err
-rw-r--r-- 1 root root 39886 2月   7 12:04 Makefile
-rw-r--r-- 1 root root  6950 2月   7 12:04 ngx_auto_config.h
-rw-r--r-- 1 root root   657 2月   7 12:03 ngx_auto_headers.h
-rw-r--r-- 1 root root  5856 2月   7 12:04 ngx_modules.c
drwxr-xr-x 9 root root    91 2月   7 12:04 src
```

`ngx_modules.c`决定了哪些模块被编译到nginx。

```bash
$ make && make install
$ ls  -l /opt/nginx/
总用量 0
drwxr-xr-x 2 root root 333 2月   7 12:11 conf
drwxr-xr-x 2 root root  40 2月   7 12:11 html
drwxr-xr-x 2 root root   6 2月   7 12:11 logs
drwxr-xr-x 2 root root  19 2月   7 12:11 sbin
```

