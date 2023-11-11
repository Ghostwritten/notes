

---
## 1. 介绍
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210524112209759.png?shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210524112336171.png?shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210524112349590.png?shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210524112522327.png?shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
## 2. Strace
参考链接：
[https://www.kernel.org/doc/man-pages/](https://www.kernel.org/doc/man-pages/)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210524112645744.png?shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210524112657373.png?shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

```c
root@master:~/imagev1.20.7# strace ls
execve("/bin/ls", ["ls"], [/* 27 vars */]) = 0
brk(NULL)                               = 0x13cf000
access("/etc/ld.so.nohwcap", F_OK)      = -1 ENOENT (No such file or directory)
access("/etc/ld.so.preload", R_OK)      = -1 ENOENT (No such file or directory)
openat(AT_FDCWD, "/etc/ld.so.cache", O_RDONLY|O_CLOEXEC) = 3
fstat(3, {st_mode=S_IFREG|0644, st_size=47425, ...}) = 0
mmap(NULL, 47425, PROT_READ, MAP_PRIVATE, 3, 0) = 0x7fe7d52c7000
close(3)                                = 0
access("/etc/ld.so.nohwcap", F_OK)      = -1 ENOENT (No such file or directory)
openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libselinux.so.1", O_RDONLY|O_CLOEXEC) = 3
read(3, "\177ELF\2\1\1\0\0\0\0\0\0\0\0\0\3\0>\0\1\0\0\0\260Z\0\0\0\0\0\0"..., 832) = 832
fstat(3, {st_mode=S_IFREG|0644, st_size=130224, ...}) = 0
mmap(NULL, 8192, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7fe7d52c5000
mmap(NULL, 2234080, PROT_READ|PROT_EXEC, MAP_PRIVATE|MAP_DENYWRITE, 3, 0) = 0x7fe7d4e88000
mprotect(0x7fe7d4ea7000, 2093056, PROT_NONE) = 0
mmap(0x7fe7d50a6000, 8192, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x1e000) = 0x7fe7d50a6000
mmap(0x7fe7d50a8000, 5856, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_ANONYMOUS, -1, 0) = 0x7fe7d50a8000
close(3)                                = 0
access("/etc/ld.so.nohwcap", F_OK)      = -1 ENOENT (No such file or directory)
openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libc.so.6", O_RDONLY|O_CLOEXEC) = 3
read(3, "\177ELF\2\1\1\3\0\0\0\0\0\0\0\0\3\0>\0\1\0\0\0\20\35\2\0\0\0\0\0"..., 832) = 832
fstat(3, {st_mode=S_IFREG|0755, st_size=2030928, ...}) = 0
mmap(NULL, 4131552, PROT_READ|PROT_EXEC, MAP_PRIVATE|MAP_DENYWRITE, 3, 0) = 0x7fe7d4a97000
mprotect(0x7fe7d4c7e000, 2097152, PROT_NONE) = 0
mmap(0x7fe7d4e7e000, 24576, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x1e7000) = 0x7fe7d4e7e000
mmap(0x7fe7d4e84000, 15072, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_ANONYMOUS, -1, 0) = 0x7fe7d4e84000
close(3)                                = 0
access("/etc/ld.so.nohwcap", F_OK)      = -1 ENOENT (No such file or directory)
openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libpcre.so.3", O_RDONLY|O_CLOEXEC) = 3
read(3, "\177ELF\2\1\1\0\0\0\0\0\0\0\0\0\3\0>\0\1\0\0\0 \25\0\0\0\0\0\0"..., 832) = 832
fstat(3, {st_mode=S_IFREG|0644, st_size=464824, ...}) = 0
mmap(NULL, 2560264, PROT_READ|PROT_EXEC, MAP_PRIVATE|MAP_DENYWRITE, 3, 0) = 0x7fe7d4825000
mprotect(0x7fe7d4895000, 2097152, PROT_NONE) = 0
mmap(0x7fe7d4a95000, 8192, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x70000) = 0x7fe7d4a95000
close(3)                                = 0
access("/etc/ld.so.nohwcap", F_OK)      = -1 ENOENT (No such file or directory)
openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libdl.so.2", O_RDONLY|O_CLOEXEC) = 3
read(3, "\177ELF\2\1\1\0\0\0\0\0\0\0\0\0\3\0>\0\1\0\0\0P\16\0\0\0\0\0\0"..., 832) = 832
fstat(3, {st_mode=S_IFREG|0644, st_size=14560, ...}) = 0
mmap(NULL, 2109712, PROT_READ|PROT_EXEC, MAP_PRIVATE|MAP_DENYWRITE, 3, 0) = 0x7fe7d4621000
mprotect(0x7fe7d4624000, 2093056, PROT_NONE) = 0
mmap(0x7fe7d4823000, 8192, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x2000) = 0x7fe7d4823000
close(3)                                = 0
access("/etc/ld.so.nohwcap", F_OK)      = -1 ENOENT (No such file or directory)
openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libpthread.so.0", O_RDONLY|O_CLOEXEC) = 3
read(3, "\177ELF\2\1\1\0\0\0\0\0\0\0\0\0\3\0>\0\1\0\0\0000b\0\0\0\0\0\0"..., 832) = 832
fstat(3, {st_mode=S_IFREG|0755, st_size=144976, ...}) = 0
mmap(NULL, 2221184, PROT_READ|PROT_EXEC, MAP_PRIVATE|MAP_DENYWRITE, 3, 0) = 0x7fe7d4402000
mprotect(0x7fe7d441c000, 2093056, PROT_NONE) = 0
mmap(0x7fe7d461b000, 8192, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x19000) = 0x7fe7d461b000
mmap(0x7fe7d461d000, 13440, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_ANONYMOUS, -1, 0) = 0x7fe7d461d000
close(3)                                = 0
mmap(NULL, 8192, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7fe7d52c3000
arch_prctl(ARCH_SET_FS, 0x7fe7d52c4040) = 0
mprotect(0x7fe7d4e7e000, 16384, PROT_READ) = 0
mprotect(0x7fe7d461b000, 4096, PROT_READ) = 0
mprotect(0x7fe7d4823000, 4096, PROT_READ) = 0
mprotect(0x7fe7d4a95000, 4096, PROT_READ) = 0
mprotect(0x7fe7d50a6000, 4096, PROT_READ) = 0
mprotect(0x61d000, 4096, PROT_READ)     = 0
mprotect(0x7fe7d52d3000, 4096, PROT_READ) = 0
munmap(0x7fe7d52c7000, 47425)           = 0
set_tid_address(0x7fe7d52c4310)         = 92749
set_robust_list(0x7fe7d52c4320, 24)     = 0
rt_sigaction(SIGRTMIN, {0x7fe7d4407cb0, [], SA_RESTORER|SA_SIGINFO, 0x7fe7d4414980}, NULL, 8) = 0
rt_sigaction(SIGRT_1, {0x7fe7d4407d50, [], SA_RESTORER|SA_RESTART|SA_SIGINFO, 0x7fe7d4414980}, NULL, 8) = 0
rt_sigprocmask(SIG_UNBLOCK, [RTMIN RT_1], NULL, 8) = 0
prlimit64(0, RLIMIT_STACK, NULL, {rlim_cur=8192*1024, rlim_max=RLIM64_INFINITY}) = 0
statfs("/sys/fs/selinux", 0x7ffdcc80a410) = -1 ENOENT (No such file or directory)
statfs("/selinux", 0x7ffdcc80a410)      = -1 ENOENT (No such file or directory)
brk(NULL)                               = 0x13cf000
brk(0x13f0000)                          = 0x13f0000
openat(AT_FDCWD, "/proc/filesystems", O_RDONLY) = 3
fstat(3, {st_mode=S_IFREG|0444, st_size=0, ...}) = 0
read(3, "nodev\tsysfs\nnodev\trootfs\nnodev\tr"..., 1024) = 420
read(3, "", 1024)                       = 0
close(3)                                = 0
openat(AT_FDCWD, "/usr/lib/locale/locale-archive", O_RDONLY|O_CLOEXEC) = 3
fstat(3, {st_mode=S_IFREG|0644, st_size=2999664, ...}) = 0
mmap(NULL, 2999664, PROT_READ, MAP_PRIVATE, 3, 0) = 0x7fe7d4125000
close(3)                                = 0
ioctl(1, TCGETS, {B38400 opost isig icanon echo ...}) = 0
ioctl(1, TIOCGWINSZ, {ws_row=16, ws_col=134, ws_xpixel=0, ws_ypixel=0}) = 0
openat(AT_FDCWD, ".", O_RDONLY|O_NONBLOCK|O_DIRECTORY|O_CLOEXEC) = 3
fstat(3, {st_mode=S_IFDIR|0755, st_size=4096, ...}) = 0
getdents(3, /* 11 entries */, 32768)    = 416
getdents(3, /* 0 entries */, 32768)     = 0
close(3)                                = 0
fstat(1, {st_mode=S_IFCHR|0600, st_rdev=makedev(136, 3), ...}) = 0
write(1, "coredns1.7.0.tar  kubeadm.conf\t\t"..., 104coredns1.7.0.tar  kubeadm.conf		    kube-controller-managerv1.20.7.takube-scheduler1.20.7.tar  tag.sh
) = 104
write(1, "etcd3.4.13.tar\t  kube-apiserver1"..., 78etcd3.4.13.tar	  kube-apiserver1.20.7.tar  kube-proxy1.20.7.tar	     pause3.2.tar
) = 78
close(1)                                = 0
close(2)                                = 0
exit_group(0)                           = ?
+++ exited with 0 +++




root@master:~/imagev1.20.7# strace -cw ls /
bin   data  etc   initrd.img  lib    lost+found  mnt  opt	proc  run   srv  tmp  var
boot  dev   home  kustomize   lib64  media	 nfs  overlays	root  sbin  sys  usr  vmlinuz
% time     seconds  usecs/call     calls    errors syscall
------ ----------- ----------- --------- --------- ----------------
 18.78    0.002930         172        17           mmap
 16.68    0.002603         237        11           close
 11.41    0.001780         148        12           mprotect
  7.45    0.001163         129         9           openat
  7.16    0.001118         112        10           fstat
  6.84    0.001067         152         7           read
  6.68    0.001043         522         2           write
  6.59    0.001029         147         7         7 access
  4.05    0.000632         632         1           execve
  2.42    0.000378         126         3           brk
  1.84    0.000287         287         1           set_tid_address
  1.69    0.000264         264         1           arch_prctl
  1.53    0.000239         120         2           ioctl
  1.45    0.000227         114         2           rt_sigaction
  1.44    0.000225         113         2         2 statfs
  1.07    0.000167          84         2           getdents
  0.76    0.000119         119         1           munmap
  0.65    0.000101         101         1           rt_sigprocmask
  0.56    0.000087          87         1           prlimit64
  0.49    0.000077          77         1           stat
  0.44    0.000069          69         1           set_robust_list
------ ----------- ----------- --------- --------- ----------------
100.00    0.015605                    94         9 total


root@master:~/imagev1.20.7# echo hello > test
root@master:~/imagev1.20.7# cat test 
hello
root@master:~/imagev1.20.7# strace cat test
execve("/bin/cat", ["cat", "test"], [/* 27 vars */]) = 0
brk(NULL)                               = 0xcb4000
access("/etc/ld.so.nohwcap", F_OK)      = -1 ENOENT (No such file or directory)
access("/etc/ld.so.preload", R_OK)      = -1 ENOENT (No such file or directory)
openat(AT_FDCWD, "/etc/ld.so.cache", O_RDONLY|O_CLOEXEC) = 3
fstat(3, {st_mode=S_IFREG|0644, st_size=47425, ...}) = 0
mmap(NULL, 47425, PROT_READ, MAP_PRIVATE, 3, 0) = 0x7f696f4d5000
close(3)                                = 0
access("/etc/ld.so.nohwcap", F_OK)      = -1 ENOENT (No such file or directory)
openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libc.so.6", O_RDONLY|O_CLOEXEC) = 3
read(3, "\177ELF\2\1\1\3\0\0\0\0\0\0\0\0\3\0>\0\1\0\0\0\20\35\2\0\0\0\0\0"..., 832) = 832
fstat(3, {st_mode=S_IFREG|0755, st_size=2030928, ...}) = 0
mmap(NULL, 8192, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7f696f4d3000
mmap(NULL, 4131552, PROT_READ|PROT_EXEC, MAP_PRIVATE|MAP_DENYWRITE, 3, 0) = 0x7f696eec7000
mprotect(0x7f696f0ae000, 2097152, PROT_NONE) = 0
mmap(0x7f696f2ae000, 24576, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x1e7000) = 0x7f696f2ae000
mmap(0x7f696f2b4000, 15072, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_ANONYMOUS, -1, 0) = 0x7f696f2b4000
close(3)                                = 0
arch_prctl(ARCH_SET_FS, 0x7f696f4d4540) = 0
mprotect(0x7f696f2ae000, 16384, PROT_READ) = 0
mprotect(0x60b000, 4096, PROT_READ)     = 0
mprotect(0x7f696f4e1000, 4096, PROT_READ) = 0
munmap(0x7f696f4d5000, 47425)           = 0
brk(NULL)                               = 0xcb4000
brk(0xcd5000)                           = 0xcd5000
openat(AT_FDCWD, "/usr/lib/locale/locale-archive", O_RDONLY|O_CLOEXEC) = 3
fstat(3, {st_mode=S_IFREG|0644, st_size=2999664, ...}) = 0
mmap(NULL, 2999664, PROT_READ, MAP_PRIVATE, 3, 0) = 0x7f696ebea000
close(3)                                = 0
fstat(1, {st_mode=S_IFCHR|0600, st_rdev=makedev(136, 3), ...}) = 0
openat(AT_FDCWD, "test", O_RDONLY)      = 3
fstat(3, {st_mode=S_IFREG|0644, st_size=6, ...}) = 0
fadvise64(3, 0, 0, POSIX_FADV_SEQUENTIAL) = 0
mmap(NULL, 139264, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7f696f4b1000
read(3, "hello\n", 131072)              = 6
write(1, "hello\n", 6hello
)                  = 6
read(3, "", 131072)                     = 0
munmap(0x7f696f4b1000, 139264)          = 0
close(3)                                = 0
close(1)                                = 0
close(2)                                = 0
exit_group(0)                           = ?
+++ exited with 0 +++

```
## 3. Strace and /proc on ETCD
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210524113942921.png?shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/2021052411400713.png?shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

```c
root@master:~/imagev1.20.7# docker ps |grep etcd
403089c86204        0369cf4303ff           "etcd --advertise-cl…"   9 days ago          Up 9 days                               k8s_etcd_etcd-master_kube-system_77699ae6105937dbb48c0a720843ce8e_0
26740d31fa82        k8s.gcr.io/pause:3.2   "/pause"                 9 days ago          Up 9 days                               k8s_POD_etcd-master_kube-system_77699ae6105937dbb48c0a720843ce8e_0


root@master:~/imagev1.20.7# ps -ef |grep etcd
root      26480  26376  2 19:31 ?        00:01:38 etcd --advertise-client-urls=https://192.168.211.40:2379 --cert-file=/etc/kubernetes/pki/etcd/server.crt --client-cert-auth=true --data-dir=/var/lib/etcd --initial-advertise-peer-urls=https://192.168.211.40:2380 --initial-cluster=master=https://192.168.211.40:2380 --key-file=/etc/kubernetes/pki/etcd/server.key --listen-client-urls=https://127.0.0.1:2379,https://192.168.211.40:2379 --listen-metrics-urls=http://127.0.0.1:2381 --listen-peer-urls=https://192.168.211.40:2380 --name=master --peer-cert-file=/etc/kubernetes/pki/etcd/peer.crt --peer-client-cert-auth=true --peer-key-file=/etc/kubernetes/pki/etcd/peer.key --peer-trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt --snapshot-count=10000 --trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
root     118382 118359 96 20:45 ?        00:00:08 kube-apiserver --advertise-address=192.168.211.40 --allow-privileged=true --authorization-mode=Node,RBAC --client-ca-file=/etc/kubernetes/pki/ca.crt --enable-admission-plugins=NodeRestriction --enable-bootstrap-token-auth=true --etcd-cafile=/etc/kubernetes/pki/etcd/ca.crt --etcd-certfile=/etc/kubernetes/pki/apiserver-etcd-client.crt --etcd-keyfile=/etc/kubernetes/pki/apiserver-etcd-client.key --etcd-servers=https://127.0.0.1:2379 --insecure-port=0 --kubelet-client-certificate=/etc/kubernetes/pki/apiserver-kubelet-client.crt --kubelet-client-key=/etc/kubernetes/pki/apiserver-kubelet-client.key --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname --proxy-client-cert-file=/etc/kubernetes/pki/front-proxy-client.crt --proxy-client-key-file=/etc/kubernetes/pki/front-proxy-client.key --requestheader-allowed-names=front-proxy-client --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.crt --requestheader-extra-headers-prefix=X-Remote-Extra- --requestheader-group-headers=X-Remote-Group --requestheader-username-headers=X-Remote-User --secure-port=6443 --service-account-issuer=https://kubernetes.default.svc.cluster.local --service-account-key-file=/etc/kubernetes/pki/sa.pub --service-account-signing-key-file=/etc/kubernetes/pki/sa.key --service-cluster-ip-range=10.96.0.0/12 --tls-cert-file=/etc/kubernetes/pki/apiserver.crt --tls-private-key-file=/etc/kubernetes/pki/apiserver.key
root     118707  39023  0 20:45 pts/3    00:00:00 grep --color=auto etcd



root@master:~/imagev1.20.7# strace -p 118382 -f
strace: Process 118382 attached with 9 threads
[pid 118403] futex(0x7501780, FUTEX_WAIT_PRIVATE, 0, NULL <unfinished ...>
[pid 118412] futex(0x7501638, FUTEX_WAIT_PRIVATE, 0, NULL <unfinished ...>
[pid 118654] madvise(0xc00f6b6000, 8192, MADV_DONTNEED) = 0
[pid 118413] futex(0xc00097e148, FUTEX_WAIT_PRIVATE, 0, NULL <unfinished ...>
[pid 118411] futex(0xc00025c548, FUTEX_WAIT_PRIVATE, 0, NULL <unfinished ...>
[pid 118409] futex(0xc00025d148, FUTEX_WAIT_PRIVATE, 0, NULL <unfinished ...>
[pid 118402] futex(0xc000068548, FUTEX_WAIT_PRIVATE, 0, NULL <unfinished ...>
[pid 118401] futex(0xc00025c548, FUTEX_WAKE_PRIVATE, 1 <unfinished ...>
[pid 118382] futex(0x74cffe8, FUTEX_WAIT_PRIVATE, 0, NULL <unfinished ...>
[pid 118654] epoll_pwait(3,  <unfinished ...>
[pid 118411] <... futex resumed> )      = -1 EAGAIN (Resource temporarily unavailable)
[pid 118401] <... futex resumed> )      = 0
[pid 118654] <... epoll_pwait resumed> [], 128, 0, NULL, 824681960376) = 0
[pid 118411] epoll_pwait(3,  <unfinished ...>
[pid 118401] nanosleep({0, 20000},  <unfinished ...>
[pid 118654] epoll_pwait(3,  <unfinished ...>
[pid 118411] <... epoll_pwait resumed> [], 128, 0, NULL, 824684053600) = 0
[pid 118401] <... nanosleep resumed> NULL) = 0
[pid 118411] futex(0xc00025c548, FUTEX_WAIT_PRIVATE, 0, NULL <unfinished ...>
[pid 118401] nanosleep({0, 20000}, NULL) = 0
[pid 118401] futex(0x74cf6d8, FUTEX_WAIT_PRIVATE, 0, {0, 2445921}) = -1 ETIMEDOUT (Connection timed out)
[pid 118401] futex(0xc00025c548, FUTEX_WAKE_PRIVATE, 1 <unfinished ...>
[pid 118654] <... epoll_pwait resumed> [], 128, 5, NULL, 0) = 0
[pid 118411] <... futex resumed> )      = 0
[pid 118401] <... futex resumed> )      = 1
[pid 118654] epoll_pwait(3, [], 128, 0, NULL, 824964667168) = 0
[pid 118654] epoll_pwait(3,  <unfinished ...>
[pid 118411] epoll_pwait(3,  <unfinished ...>
[pid 118654] <... epoll_pwait resumed> [], 128, 1, NULL, 0) = 0
[pid 118411] <... epoll_pwait resumed> [], 128, 0, NULL, 0) = 0
[pid 118401] nanosleep({0, 20000},  <unfinished ...>
[pid 118654] epoll_pwait(3,  <unfinished ...>
[pid 118411] epoll_pwait(3,  <unfinished ...>
[pid 118401] <... nanosleep resumed> NULL) = 0
[pid 118654] <... epoll_pwait resumed> [], 128, 0, NULL, 824679171360) = 0
[pid 118401] futex(0xc00025d148, FUTEX_WAKE_PRIVATE, 1 <unfinished ...>
[pid 118654] futex(0xc004dfb148, FUTEX_WAIT_PRIVATE, 0, NULL <unfinished ...>
[pid 118411] <... epoll_pwait resumed> [], 128, 1, NULL, 0) = 0
[pid 118409] <... futex resumed> )      = 0
[pid 118411] epoll_pwait(3,  <unfinished ...>
[pid 118409] epoll_pwait(3,  <unfinished ...>
[pid 118401] <... futex resumed> )      = 1
[pid 118411] <... epoll_pwait resumed> [], 128, 0, NULL, 824765465504) = 0
[pid 118411] epoll_pwait(3,  <unfinished ...>
[pid 118409] <... epoll_pwait resumed> [], 128, 0, NULL, 824666970944) = 0
[pid 118409] futex(0xc00025d148, FUTEX_WAIT_PRIVATE, 0, NULL <unfinished ...>
[pid 118401] nanosleep({0, 20000}, NULL) = 0
[pid 118401] futex(0x74cf6d8, FUTEX_WAIT_PRIVATE, 0, {0, 5192348}) = -1 ETIMEDOUT (Connection timed out)
[pid 118401] futex(0xc00025d148, FUTEX_WAKE_PRIVATE, 1 <unfinished ...>
[pid 118409] <... futex resumed> )      = 0
[pid 118401] <... futex resumed> )      = 1
[pid 118409] futex(0xc004dfb148, FUTEX_WAKE_PRIVATE, 1 <unfinished ...>
[pid 118401] nanosleep({0, 20000},  <unfinished ...>
[pid 118654] <... futex resumed> )      = 0
[pid 118409] <... futex resumed> )      = 1



root@master:~/imagev1.20.7# strace -p 118382 -f -cw
strace: Process 118382 attached with 11 threads
^Cstrace: Process 118382 detached
strace: Process 118401 detached
strace: Process 118402 detached
strace: Process 118403 detached
strace: Process 118409 detached
strace: Process 118411 detached
strace: Process 118412 detached
strace: Process 118413 detached
strace: Process 118654 detached
strace: Process 52091 detached
strace: Process 66307 detached
% time     seconds  usecs/call     calls    errors syscall
------ ----------- ----------- --------- --------- ----------------
 81.28  299.370417        7380     40567      9152 futex
 16.06   59.149398        1571     37660        19 epoll_pwait
  2.15    7.910671         430     18390         1 nanosleep
  0.33    1.215610         304      3998           write
  0.12    0.436564          84      5196      1585 read
  0.03    0.109598          21      5169           sched_yield
  0.01    0.030087          72       417           getrandom
  0.01    0.022096          43       509           setsockopt
  0.01    0.019983         100       199           getpid
  0.01    0.019575          98       199        21 rt_sigreturn
  0.00    0.014075          71       199           tgkill
  0.00    0.009442          57       166        20 epoll_ctl
  0.00    0.009203          64       144        72 accept4
  0.00    0.004030          55        73           getsockname
  0.00    0.003636          44        83           close
  0.00    0.000830         830         1         1 restart_syscall
  0.00    0.000404          40        10           openat
  0.00    0.000261          26        10           fstat
  0.00    0.000187         187         1         1 connect
  0.00    0.000059          59         1           socket
  0.00    0.000033          33         1           getpeername
  0.00    0.000030          30         1           getsockopt
------ ----------- ----------- --------- --------- ----------------
100.00  368.326189                112994     10872 total


root@master:~/imagev1.20.7# ls /proc/118382/
attr        cmdline          environ  io         mem         ns             pagemap      schedstat  stat     timers
autogroup   comm             exe      limits     mountinfo   numa_maps      personality  sessionid  statm    uid_map
auxv        coredump_filter  fd       loginuid   mounts      oom_adj        projid_map   setgroups  status   wchan
cgroup      cpuset           fdinfo   map_files  mountstats  oom_score      root         smaps      syscall
clear_refs  cwd              gid_map  maps       net         oom_score_adj  sched        stack      task



root@master:~/imagev1.20.7# ls -l /proc/118382/exe
lrwxrwxrwx 1 root root 0 May 23 20:45 /proc/118382/exe -> /usr/local/bin/kube-apiserver


root@master:~/imagev1.20.7# ls  /proc/118382/fd
0    101  105  109  112  13   148  16   19   21  25  29  32  36  4   43  47  50  54  58  61  65  69  72  76  8   83  87  90  94  98
1    102  106  11   115  134  15   165  191  22  26  3   33  37  40  44  48  51  55  59  62  66  7   73  77  80  84  88  91  95  99
10   103  107  110  12   135  154  17   2    23  27  30  34  38  41  45  49  52  56  6   63  67  70  74  78  81  85  89  92  96
100  104  108  111  121  14   156  18   20   24  28  31  35  39  42  46  5   53  57  60  64  68  71  75  79  82  86  9   93  97root@master:~/imagev1.20.7# ls  /proc/118382/fd
0    101  105  109  112  13   148  16   19   21  25  29  32  36  4   43  47  50  54  58  61  65  69  72  76  8   83  87  90  94  98
1    102  106  11   115  134  15   165  191  22  26  3   33  37  40  44  48  51  55  59  62  66  7   73  77  80  84  88  91  95  99
10   103  107  110  12   135  154  17   2    23  27  30  34  38  41  45  49  52  56  6   63  67  70  74  78  81  85  89  92  96
100  104  108  111  121  14   156  18   20   24  28  31  35  39  42  46  5   53  57  60  64  68  71  75  79  82  86  9   93  97


root@master:~/imagev1.20.7# ls -l /proc/118382/fd
total 0
lrwx------ 1 root root 64 May 14 01:37 0 -> /dev/null
l-wx------ 1 root root 64 May 14 01:37 1 -> pipe:[86495]
lrwx------ 1 root root 64 May 14 01:37 10 -> socket:[87394]
lrwx------ 1 root root 64 May 14 01:50 100 -> socket:[372798]
lrwx------ 1 root root 64 May 14 01:50 101 -> socket:[372800]
lrwx------ 1 root root 64 May 14 01:50 102 -> socket:[371678]
lrwx------ 1 root root 64 May 14 01:50 103 -> socket:[371689]
lrwx------ 1 root root 64 May 14 01:50 104 -> socket:[372819]
lrwx------ 1 root root 64 May 14 01:51 105 -> socket:[376085]
l-wx------ 1 root root 64 May 14 01:37 11 -> /var/lib/etcd/member/wal/0.tmp
lrwx------ 1 root root 64 May 14 01:37 12 -> socket:[87310]
lrwx------ 1 root root 64 May 14 01:37 13 -> socket:[87395]
lrwx------ 1 root root 64 May 14 01:37 14 -> socket:[87396]
lrwx------ 1 root root 64 May 14 01:37 15 -> socket:[87398]
lrwx------ 1 root root 64 May 14 01:37 16 -> socket:[372255]
lrwx------ 1 root root 64 May 14 01:37 17 -> socket:[372258]
lrwx------ 1 root root 64 May 14 01:37 18 -> socket:[371212]
lrwx------ 1 root root 64 May 14 01:37 19 -> socket:[371207]
l-wx------ 1 root root 64 May 14 01:37 2 -> pipe:[86496]
lrwx------ 1 root root 64 May 14 01:37 20 -> socket:[371214]
lrwx------ 1 root root 64 May 14 01:37 21 -> socket:[371218]
lrwx------ 1 root root 64 May 14 01:37 22 -> socket:[372265]
lrwx------ 1 root root 64 May 14 01:37 23 -> socket:[372268]
lrwx------ 1 root root 64 May 14 01:37 24 -> socket:[372271]
lrwx------ 1 root root 64 May 14 01:37 25 -> socket:[371222]
lrwx------ 1 root root 64 May 14 01:37 26 -> socket:[372275]
lrwx------ 1 root root 64 May 14 01:37 27 -> socket:[372278]
lrwx------ 1 root root 64 May 14 01:37 28 -> socket:[371229]
lrwx------ 1 root root 64 May 14 01:37 29 -> socket:[372284]
lrwx------ 1 root root 64 May 14 01:37 3 -> socket:[87303]
lrwx------ 1 root root 64 May 14 01:37 30 -> socket:[372287]
lrwx------ 1 root root 64 May 14 01:37 31 -> socket:[372294]
lrwx------ 1 root root 64 May 14 01:37 32 -> socket:[372304]
lrwx------ 1 root root 64 May 14 01:37 33 -> socket:[372307]
lrwx------ 1 root root 64 May 14 01:37 34 -> socket:[372309]
lrwx------ 1 root root 64 May 14 01:37 35 -> socket:[372312]
lrwx------ 1 root root 64 May 14 01:37 36 -> socket:[372315]
lrwx------ 1 root root 64 May 14 01:37 37 -> socket:[371250]
lrwx------ 1 root root 64 May 14 01:37 38 -> socket:[371253]
lrwx------ 1 root root 64 May 14 01:37 39 -> socket:[371256]
lrwx------ 1 root root 64 May 14 01:37 4 -> anon_inode:[eventpoll]
lrwx------ 1 root root 64 May 14 01:37 40 -> socket:[371259]
lrwx------ 1 root root 64 May 14 01:37 41 -> socket:[371262]
lrwx------ 1 root root 64 May 14 01:37 42 -> socket:[372319]
lrwx------ 1 root root 64 May 14 01:37 43 -> socket:[371264]
lrwx------ 1 root root 64 May 14 01:37 44 -> socket:[371266]
lrwx------ 1 root root 64 May 14 01:37 45 -> socket:[371268]
lrwx------ 1 root root 64 May 14 01:37 46 -> socket:[371270]
lrwx------ 1 root root 64 May 14 01:37 47 -> socket:[371272]
lrwx------ 1 root root 64 May 14 01:37 48 -> socket:[372327]
lrwx------ 1 root root 64 May 14 01:37 49 -> socket:[371274]
lrwx------ 1 root root 64 May 14 01:37 5 -> socket:[87307]
lrwx------ 1 root root 64 May 14 01:37 50 -> socket:[371276]
lrwx------ 1 root root 64 May 14 01:37 51 -> socket:[371278]
lrwx------ 1 root root 64 May 14 01:37 52 -> socket:[371280]
lrwx------ 1 root root 64 May 14 01:37 53 -> socket:[371283]
lrwx------ 1 root root 64 May 14 01:37 54 -> socket:[371285]
lrwx------ 1 root root 64 May 14 01:37 55 -> socket:[371287]
lrwx------ 1 root root 64 May 14 01:37 56 -> socket:[371289]
lrwx------ 1 root root 64 May 14 01:37 57 -> socket:[372338]
lrwx------ 1 root root 64 May 14 01:37 58 -> socket:[371291]
lrwx------ 1 root root 64 May 14 01:37 59 -> socket:[371293]
lrwx------ 1 root root 64 May 14 01:37 6 -> socket:[87308]
lrwx------ 1 root root 64 May 14 01:37 60 -> socket:[371295]
lrwx------ 1 root root 64 May 14 01:37 61 -> socket:[372344]
lrwx------ 1 root root 64 May 14 01:37 62 -> socket:[371297]
lrwx------ 1 root root 64 May 14 01:37 63 -> socket:[372348]
lrwx------ 1 root root 64 May 14 01:37 64 -> socket:[371299]
lrwx------ 1 root root 64 May 14 01:37 65 -> socket:[371301]
lrwx------ 1 root root 64 May 14 01:37 66 -> socket:[371303]
lrwx------ 1 root root 64 May 14 01:37 67 -> socket:[371305]
lrwx------ 1 root root 64 May 14 01:37 68 -> socket:[372355]
lrwx------ 1 root root 64 May 14 01:37 69 -> socket:[371307]
lrwx------ 1 root root 64 May 14 01:37 7 -> /var/lib/etcd/member/snap/db
lrwx------ 1 root root 64 May 14 01:37 70 -> socket:[371310]
lrwx------ 1 root root 64 May 14 01:37 71 -> socket:[371312]
lrwx------ 1 root root 64 May 14 01:37 72 -> socket:[371315]
lrwx------ 1 root root 64 May 14 01:37 73 -> socket:[372361]
lrwx------ 1 root root 64 May 14 01:37 74 -> socket:[371318]
lrwx------ 1 root root 64 May 14 01:37 75 -> socket:[371319]
lrwx------ 1 root root 64 May 14 01:37 76 -> socket:[371321]
lrwx------ 1 root root 64 May 14 01:37 77 -> socket:[371323]
lrwx------ 1 root root 64 May 14 01:37 78 -> socket:[371325]
lrwx------ 1 root root 64 May 14 01:37 79 -> socket:[372368]
l-wx------ 1 root root 64 May 14 01:37 8 -> /var/lib/etcd/member/wal/0000000000000000-0000000000000000.wal
lrwx------ 1 root root 64 May 14 01:37 80 -> socket:[372371]
lrwx------ 1 root root 64 May 14 01:37 81 -> socket:[371327]
lrwx------ 1 root root 64 May 14 01:37 82 -> socket:[371329]
lrwx------ 1 root root 64 May 14 01:37 83 -> socket:[371332]
lrwx------ 1 root root 64 May 14 01:37 84 -> socket:[371337]
lrwx------ 1 root root 64 May 14 01:37 85 -> socket:[371339]
lrwx------ 1 root root 64 May 14 01:37 86 -> socket:[371342]
lrwx------ 1 root root 64 May 14 01:37 87 -> socket:[371344]
lrwx------ 1 root root 64 May 14 01:37 88 -> socket:[371346]
lrwx------ 1 root root 64 May 14 01:37 89 -> socket:[372611]
lr-x------ 1 root root 64 May 14 01:37 9 -> /var/lib/etcd/member/wal
lrwx------ 1 root root 64 May 14 01:37 90 -> socket:[371362]
lrwx------ 1 root root 64 May 14 01:50 91 -> socket:[372816]
lrwx------ 1 root root 64 May 14 01:50 92 -> socket:[371669]
lrwx------ 1 root root 64 May 14 01:50 93 -> socket:[372782]
lrwx------ 1 root root 64 May 14 01:50 94 -> socket:[371671]
lrwx------ 1 root root 64 May 14 01:50 95 -> socket:[372785]
lrwx------ 1 root root 64 May 14 01:50 96 -> socket:[371674]
lrwx------ 1 root root 64 May 14 01:50 97 -> socket:[372789]
lrwx------ 1 root root 64 May 14 01:50 98 -> socket:[372792]
lrwx------ 1 root root 64 May 14 01:50 99 -> socket:[372795]




root@master:/proc/26480/fd# tail 7
4/registry/leases/kube-system/kube-controller-manager񿘱 µ*²k8s
$

oordination.k8s.io/v1beta1Lease 
¯ 
kube-controller-manager 
                       kube-system"*$bcb35dd3-5cb0-4460-99af-dcedb37a6bfa2ڭ򄏺Ɂ
kube-controller-managerUpdatecoordination.k8s.io/vڭFieldsV1:|
z{"f:spec":{"f:acquireTime":{},"f:holderIdentity":{},"f:leaseDurationSeconds":{},"f:leaseTransitions":{},"f:renewTime":{}}}M
+master_3df1ffaf-d977-40ec-addf-94dc8f121598 
                                           ڭ򄑈³"
Z©[¬\­]®^°_.`/a,b7c;deafgBhGijklmno¡pqr¥st¢u§v¤w£x¨y¬z©{­|°}«~ª¹󿾁±²Á·¼ǁµ¶́´ˁ¸ρсºԁ»s_|_____ _©_¬_²_¹_
_²@AB±C__s_Z\bÿ__³¥«®°ޟ֟ԟ̟__      _	
  µD	 	 CY\¸	AD ¯alarmauth 
p                                   authRevisionauthRolesauthUsersclusterclusterVersion3.4.0key²lease  
t 
k'yj"                                                                                                                                󛈱 ­ޓkʜk'yj± ­ޓkʜk'yj"b± ­ޓkʜk'yj"       ¦± ­ޓkʜk'yj" ©± ­ޓkʜk'yj"­¬± ­ޓkʜk'yj"¿¯± ­ޓkʜk'yj"£³± ­ޓkʜk'yjʜk'yj"ں± ­ޓkʜk'yjܿ± ­ޓkʜk'yj"#ӆ± ­ޓkʜk'yj"&Ԍ± ­ޓkʜk'yj"± ­ޓkʜk'yj"*£Ո± ­ޓkʜk'yj"+Ԗ± ­ޓkk'yj"׈± ­ޓkmembersf9cd118806feeb27{"id":18000062561600334631,"peerURLs":["https://192.168.211.40:2380"],"name":"master","clientURLs":["https://192.168.211.40:2379"]}members_removedmeta8Kconsistent_index
                                                                                                                          finishedCompactRev¾_scheduledCompactReH_




root@master:/proc/26480/fd# k create secret generic credit-card --from-literal cc=111222333444 -oyaml --dry-run=client
apiVersion: v1
data:
  cc: MTExMjIyMzMzNDQ0
kind: Secret
metadata:
  creationTimestamp: null
  name: credit-card
root@master:/proc/26480/fd# k create secret generic credit-card --from-literal cc=111222333444 
secret/credit-card created
root@master:/proc/26480/fd# cat 7 | strings | grep 111222333444
111222333444


root@master:/proc/26480/fd# cat 7 | strings | grep 111222333444  -A 10 -B 10
192.168.211.40
%/registry/secrets/default/credit-card
Secret
credit-card
default"
*$d629496a-41e5-4e9c-ac11-862754779ca02
kubectl-create
Update
FieldsV1:+
){"f:data":{".":{},"f:cc":{}},"f:type":{}}
111222333444
Opaque
&/registry/leases/kube-node-lease/node2
coordination.k8s.io/v1beta1
Lease
node2
kube-node-lease"
*$05d1434a-36fe-4e0a-b00e-64da185a29412
Node
node2"$165aff9d-2e1b-4e7e-957a-809b3035f0cd*
kubelet

```
	
## 4. /proc and env variables
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210524140010274.png)

```c
root@master:~/cks/runtime-security# k run apache --image=httpd -oyaml --dry-run=client > pod.yaml
root@master:~/cks/runtime-security# vim pod.yaml 
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: apache
  name: apache
spec:
  containers:
  - image: httpd
    name: apache
    resources: {}
    env: 
    - name: SECRET
      value: "5555666677778888"
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
root@master:~/cks/runtime-security# k create -f pod.yaml 
pod/apache created


root@master:~/cks/runtime-security# k get pods -owide |grep apache
apache   1/1     Running   0          89s    10.244.104.3   node2   <none>           <none>

root@node2:~# ps aux |grep httpd
root     123888  0.0  0.2   5940  4420 ?        Ss   23:04   0:00 httpd -DFOREGROUND
daemon   123921  0.0  0.1 1210608 3536 ?        Sl   23:04   0:00 httpd -DFOREGROUND
daemon   123922  0.0  0.1 1210608 3536 ?        Sl   23:04   0:00 httpd -DFOREGROUND
daemon   123923  0.0  0.1 1210608 3536 ?        Sl   23:04   0:00 httpd -DFOREGROUND
root     126181  0.0  0.0  14408  1004 pts/0    S+   23:05   0:00 grep --color=auto httpd
root@node2:~# docker ps |grep httpd
ced29b338f66        httpd                  "httpd-foreground"       About a minute ago   Up About a minute                       k8s_apache_apache_default_fa048b0d-e8a6-4144-8ecb-22dd66ea5f44_0
root@node2:~# docker ps |grep apache
ced29b338f66        httpd                  "httpd-foreground"       About a minute ago   Up About a minute                       k8s_apache_apache_default_fa048b0d-e8a6-4144-8ecb-22dd66ea5f44_0
55f829e84a8d        k8s.gcr.io/pause:3.2   "/pause"                 2 minutes ago        Up 2 minutes                            k8s_POD_apache_default_fa048b0d-e8a6-4144-8ecb-22dd66ea5f44_0
root@node2:~# 



root@node2:~# pstree -p

...................
     │                 ├─containerd-shim(123862)─┬─httpd(123888)─┬─httpd(123921)─┬─{httpd}(123953)
           │                 │                         │               │               ├─{httpd}(123954)
           │                 │                         │               │               ├─{httpd}(123955)
           │                 │                         │               │               ├─{httpd}(123957)
           │                 │                         │               │               ├─{httpd}(123961)
           │                 │                         │               │               ├─{httpd}(123963)
           │                 │                         │               │               ├─{httpd}(123965)
           │                 │                         │               │               ├─{httpd}(123967)
           │                 │                         │               │               ├─{httpd}(123969)
           │                 │                         │               │               ├─{httpd}(123971)
           │                 │                         │               │               ├─{httpd}(123974)
           │                 │                         │               │               ├─{httpd}(123976)
           │                 │                         │               │               ├─{httpd}(123978)
           │                 │                         │               │               ├─{httpd}(123980)
           │                 │                         │               │               ├─{httpd}(123982)
           │                 │                         │               │               ├─{httpd}(123984)
           │                 │                         │               │               ├─{httpd}(123986)
           │                 │                         │               │               ├─{httpd}(123988)
           │                 │                         │               │               ├─{httpd}(123990)
           │                 │                         │               │               ├─{httpd}(123992)
           │                 │                         │               │               ├─{httpd}(123994)
           │                 │                         │               │               ├─{httpd}(123996)
           │                 │                         │               │               ├─{httpd}(123998)
           │                 │                         │               │               ├─{httpd}(124000)
           │                 │                         │               │               ├─{httpd}(124002)
           │                 │                         │               │               └─{httpd}(124004)
           │                 │                         │               ├─httpd(123922)─┬─{httpd}(123956)
           │                 │                         │               │               ├─{httpd}(123958)
           │                 │                         │               │               ├─{httpd}(123959)
           │                 │                         │               │               ├─{httpd}(123960)
           │                 │                         │               │               ├─{httpd}(123962)
           │                 │                         │               │               ├─{httpd}(123964)
           │                 │                         │               │               ├─{httpd}(123966)
           │                 │                         │               │               ├─{httpd}(123968)
           │                 │                         │               │               ├─{httpd}(123970)
           │                 │                         │               │               ├─{httpd}(123972)
           │                 │                         │               │               ├─{httpd}(123975)
           │                 │                         │               │               ├─{httpd}(123977)
           │                 │                         │               │               ├─{httpd}(123979)
           │                 │                         │               │               ├─{httpd}(123981)
           │                 │                         │               │               ├─{httpd}(123983)
           │                 │                         │               │               ├─{httpd}(123985)
           │                 │                         │               │               ├─{httpd}(123987)
           │                 │                         │               │               ├─{httpd}(123989)
           │                 │                         │               │               ├─{httpd}(123991)
           │                 │                         │               │               ├─{httpd}(123993)
           │                 │                         │               │               ├─{httpd}(123995)
           │                 │                         │               │               ├─{httpd}(123997)
           │                 │                         │               │               ├─{httpd}(123999)
           │                 │                         │               │               ├─{httpd}(124001)
           │                 │                         │               │               ├─{httpd}(124003)
           │                 │                         │               │               └─{httpd}(124005)
           │                 │                         │               └─httpd(123923)─┬─{httpd}(123926)
           │                 │                         │                               ├─{httpd}(123927)
           │                 │                         │                               ├─{httpd}(123928)
           │                 │                         │                               ├─{httpd}(123929)
           │                 │                         │                               ├─{httpd}(123930)
           │                 │                         │                               ├─{httpd}(123931)
           │                 │                         │                               ├─{httpd}(123932)
           │                 │                         │                               ├─{httpd}(123933)
           │                 │                         │                               ├─{httpd}(123934)
           │                 │                         │                               ├─{httpd}(123935)
           │                 │                         │                               ├─{httpd}(123936)
           │                 │                         │                               ├─{httpd}(123937)
           │                 │                         │                               ├─{httpd}(123938)
           │                 │                         │                               ├─{httpd}(123939)
           │                 │                         │                               ├─{httpd}(123940)
           │                 │                         │                               ├─{httpd}(123941)
           │                 │                         │                               ├─{httpd}(123942)
           │                 │                         │                               ├─{httpd}(123943)
           │                 │                         │                               ├─{httpd}(123944)
           │                 │                         │                               ├─{httpd}(123945)
           │                 │                         │                               ├─{httpd}(123946)
           │                 │                         │                               ├─{httpd}(123947)
           │                 │                         │                               ├─{httpd}(123948)
           │                 │                         │                               ├─{httpd}(123949)
           │                 │                         │                               ├─{httpd}(123950)
           │                 │                         │                               └─{




root@node2:~# cd /proc/123888
root@node2:/proc/123888# ls
attr       cgroup      comm             cwd      fd       io        map_files  mountinfo   net        oom_adj        pagemap      root       sessionid  stack  status   timers
autogroup  clear_refs  coredump_filter  environ  fdinfo   limits    maps       mounts      ns         oom_score      personality  sched      setgroups  stat   syscall  uid_map
auxv       cmdline     cpuset           exe      gid_map  loginuid  mem        mountstats  numa_maps  oom_score_adj  projid_map   schedstat  smaps      statm  task     wchan
root@node2:/proc/123888# ls -lh exe 
lrwxrwxrwx 1 root root 0 May 23 23:04 exe -> /usr/local/apache2/bin/httpd
root@node2:/proc/123888# cd fd
root@node2:/proc/123888/fd# ls -lh 
total 0
lrwx------ 1 root root 64 May 23 23:04 0 -> /dev/null
l-wx------ 1 root root 64 May 23 23:04 1 -> pipe:[15919327]
l-wx------ 1 root root 64 May 23 23:04 2 -> pipe:[15919328]
lrwx------ 1 root root 64 May 23 23:04 3 -> socket:[15919481]
lr-x------ 1 root root 64 May 23 23:04 4 -> pipe:[15919497]
l-wx------ 1 root root 64 May 23 23:04 5 -> pipe:[15919497]
l-wx------ 1 root root 64 May 23 23:04 6 -> pipe:[15919327]
root@node2:/proc/123888/fd# cd ..
root@node2:/proc/123888# cat environ 
KUBERNETES_SERVICE_PORT=443KUBERNETES_PORT=tcp://10.96.0.1:443HTTPD_VERSION=2.4.46HOSTNAME=apacheHOME=/rootHTTPD_PATCHES=KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1PATH=/usr/local/apache2/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/binKUBERNETES_PORT_443_TCP_PORT=443KUBERNETES_PORT_443_TCP_PROTO=tcpHTTPD_SHA256=740eddf6e1c641992b22359cabc66e6325868c3c5e2e3f98faf349b61ecf41eaSECRET=5555666677778888HTTPD_PREFIX=/usr/local/apache2KUBERNETES_SERVICE_PORT_HTTPS=443KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443KUBERNETES_SERVICE_HOST=10.96.0.1PWD=/usr/local/apache2root@node2:/proc/123888# 
root@node2:/proc/123888# 
```


## 5. Falco and Installation

[falco官网](https://falco.org/)
github: [https://github.com/falcosecurity/falco](https://github.com/falcosecurity/falco)
k8s wtih falco: [https://v1-17.docs.kubernetes.io/docs/tasks/debug-application-cluster/falco/](https://v1-17.docs.kubernetes.io/docs/tasks/debug-application-cluster/falco/)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210524141120519.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210524141128843.png)
官方下载安装：[https://falco.org/docs/getting-started/installation/](https://falco.org/docs/getting-started/installation/)

```bash
# install falco
curl -s https://falco.org/repo/falcosecurity-3672BA8F.asc | apt-key add -
echo "deb https://download.falco.org/packages/deb stable main" | tee -a /etc/apt/sources.list.d/falcosecurity.list 
apt-get update -y
apt-get -y install linux-headers-$(uname -r)
apt-get install -y falco=0.26.1
```


```c
root@node2:~/falco# systemctl start falco
root@node2:~/falco# systemctl enable falco
Created symlink from /etc/systemd/system/multi-user.target.wants/falco.service to /usr/lib/systemd/system/falco.service.
root@node2:~/falco# systemctl status falco
● falco.service - Falco: Container Native Runtime Security
   Loaded: loaded (/usr/lib/systemd/system/falco.service; enabled; vendor preset: enabled)
   Active: active (running) since Sun 2021-05-23 23:20:59 PDT; 12s ago
     Docs: https://falco.org/docs/
 Main PID: 28817 (falco)
   CGroup: /system.slice/falco.service
           └─28817 /usr/bin/falco --pidfile=/var/run/falco.pid

May 23 23:21:00 node2 falco[28817]: Falco initialized with configuration file /etc/falco/falco.yaml
May 23 23:21:00 node2 falco[28817]: Loading rules from file /etc/falco/falco_rules.yaml:
May 23 23:21:00 node2 falco[28817]: Loading rules from file /etc/falco/falco_rules.local.yaml:
May 23 23:21:00 node2 falco[28817]: Sun May 23 23:21:00 2021: Loading rules from file /etc/falco/falco_rules.local.yaml:
May 23 23:21:00 node2 falco[28817]: Loading rules from file /etc/falco/k8s_audit_rules.yaml:
May 23 23:21:00 node2 falco[28817]: Sun May 23 23:21:00 2021: Loading rules from file /etc/falco/k8s_audit_rules.yaml:
May 23 23:21:00 node2 falco[28817]: Starting internal webserver, listening on port 8765
May 23 23:21:00 node2 falco[28817]: Sun May 23 23:21:00 2021: Starting internal webserver, listening on port 8765
May 23 23:21:05 node2 systemd[1]: [/usr/lib/systemd/system/falco.service:19] Unknown lvalue 'ProtectKernelTunables' in section 'Service'
May 23 23:21:05 node2 systemd[1]: [/usr/lib/systemd/system/falco.service:20] Unknown lvalue 'RestrictRealtime' in section 'Service'


root@node2:~/falco# ls /etc/falco/
falco_rules.local.yaml  falco_rules.yaml  falco.yaml  k8s_audit_rules.yaml  rules.available  rules.d
root@node2:~/falco# tail /var/log/syslog|grep falco
May 23 23:21:00 node2 kernel: [192079.038231] falco: initializing ring buffer for CPU 1
May 23 23:21:00 node2 kernel: [192079.088336] falco: CPU buffer initialized, size=8388608
May 23 23:21:00 node2 kernel: [192079.088339] falco: starting capture
May 23 23:21:00 node2 falco: Starting internal webserver, listening on port 8765

```

##  6. Use Falco to find malicious processes

![在这里插入图片描述](https://img-blog.csdnimg.cn/2021052414235822.png)

```cpp
root@master:~/cks/runtime-security# k exec -ti apache -- bash
root@apache:/usr/local/apache2# echo user >> /etc/passwd
root@apache:/usr/local/apache2# apt-get update
Get:1 http://deb.debian.org/debian buster InRelease [121 kB]
Get:2 http://deb.debian.org/debian buster-updates InRelease [51.9 kB]                  
Get:3 http://deb.debian.org/debian buster/main amd64 Packages [7907 kB]                 
Get:4 http://security.debian.org/debian-security buster/updates InRelease [65.4 kB]
Get:5 http://security.debian.org/debian-security buster/updates/main amd64 Packages [289 kB]
Get:6 http://deb.debian.org/debian buster-updates/main amd64 Packages [10.9 kB]
Fetched 8446 kB in 5s (1842 kB/s)                          
Reading package lists... Done


root@node2:~/falco# tail -f /var/log/syslog|grep falco
May 23 23:25:17 node2 falco[28817]: 23:25:16.992066800: Notice A shell was spawned in a container with an attached terminal (user=root user_loginuid=-1 k8s_apache_apache_default_fa048b0d-e8a6-4144-8ecb-22dd66ea5f44_0 (id=ced29b338f66) shell=bash parent=runc cmdline=bash terminal=34816 container_id=ced29b338f66 image=httpd)
May 23 23:25:17 node2 falco: 23:25:16.992066800: Notice A shell was spawned in a container with an attached terminal (user=root user_loginuid=-1 k8s_apache_apache_default_fa048b0d-e8a6-4144-8ecb-22dd66ea5f44_0 (id=ced29b338f66) shell=bash parent=runc cmdline=bash terminal=34816 container_id=ced29b338f66 image=httpd)
May 23 23:25:46 node2 falco[28817]: 23:25:46.131128350: Error File below /etc opened for writing (user=root user_loginuid=-1 command=bash parent=<NA> pcmdline=<NA> file=/etc/passwd program=bash gparent=<NA> ggparent=<NA> gggparent=<NA> container_id=ced29b338f66 image=httpd)
May 23 23:25:46 node2 falco: 23:25:46.131128350: Error File below /etc opened for writing (user=root user_loginuid=-1 command=bash parent=<NA> pcmdline=<NA> file=/etc/passwd program=bash gparent=<NA> ggparent=<NA> gggparent=<NA> container_id=ced29b338f66 image=httpd)
May 23 23:26:18 node2 falco[28817]: 23:26:18.336286131: Error Package management process launched in container (user=root user_loginuid=-1 command=apt-get update container_id=ced29b338f66 container_name=k8s_apache_apache_default_fa048b0d-e8a6-4144-8ecb-22dd66ea5f44_0 image=httpd:latest)
May 23 23:26:18 node2 falco: 23:26:18.336286131: Error Package management process launched in container (user=root user_loginuid=-1 command=apt-get update container_id=ced29b338f66 container_name=k8s_apache_apache_default_fa048b0d-e8a6-4144-8ecb-22dd66ea5f44_0 image=httpd:latest)
```

修改配置

```c
root@master:~/cks/runtime-security# vim pod.yaml 
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: apache
  name: apache
spec:
  containers:
  - image: httpd
    name: apache
    resources: {}
    env: 
    - name: SECRET
      value: "5555666677778888"
    readinessProbe: 
      exec:
        command:
        - apt-get
        - update
      initialDelaySeconds: 5
      periodSeconds: 3
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}



root@master:~/cks/runtime-security# k -f pod.yaml delete --force --grace-period 0
warning: Immediate deletion does not wait for confirmation that the running resource has been terminated. The resource may continue to run on the cluster indefinitely.
pod "apache" force deleted
root@master:~/cks/runtime-security# k -f pod.yaml create
pod/apache created
root@master:~/cks/runtime-security# k get pod apache -o wide
NAME     READY   STATUS    RESTARTS   AGE    IP             NODE    NOMINATED NODE   READINESS GATES
apache   0/1     Running   0          105s   10.244.104.4   node2   <none>           <none>


#发现报错进程
root@node2:~/falco# tail -f /var/log/syslog|grep falco
May 23 23:33:01 node2 falco[28817]: 23:33:01.783656151: Error Package management process launched in container (user=root user_loginuid=-1 command=apt-get update container_id=04d978b13984 container_name=k8s_apache_apache_default_7600fec6-b715-41a2-98e4-a0fe692f30e8_0 image=httpd:latest)
May 23 23:33:01 node2 falco: 23:33:01.783656151: Error Package management process launched in container (user=root user_loginuid=-1 command=apt-get update container_id=04d978b13984 container_name=k8s_apache_apache_default_7600fec6-b715-41a2-98e4-a0fe692f30e8_0 image=httpd:latest)
May 23 23:33:04 node2 falco[28817]: 23:33:04.833053968: Error Package management process launched in container (user=root user_loginuid=-1 command=apt-get update container_id=04d978b13984 container_name=k8s_apache_apache_default_7600fec6-b715-41a2-98e4-a0fe692f30e8_0 image=httpd:latest)

```


## 7. Practice - Investigate Falco rules
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210524144913460.png)

官方：[https://falco.org/docs/rules/](https://falco.org/docs/rules/)

## 8. Change Falco Rule
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210524145008291.png?shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

```c
root@master:~/cks/runtime-security# k get pods -owide
NAME     READY   STATUS    RESTARTS   AGE     IP             NODE    NOMINATED NODE   READINESS GATES
apache   1/1     Running   0          24s     10.244.104.5   node2   <none>           <none>
test     1/1     Running   0          3h32m   10.244.104.2   node2   <none>           <none>
root@master:~/cks/runtime-security# k exec -ti apache -- bash



root@node2:~# systemctl stop falco
root@node2:~# falco
Sun May 23 23:53:14 2021: Falco version 0.28.1 (driver version 5c0b863ddade7a45568c0ac97d037422c9efb750)
Sun May 23 23:53:14 2021: Falco initialized with configuration file /etc/falco/falco.yaml
Sun May 23 23:53:14 2021: Loading rules from file /etc/falco/falco_rules.yaml:
Sun May 23 23:53:14 2021: Loading rules from file /etc/falco/falco_rules.local.yaml:
Sun May 23 23:53:14 2021: Loading rules from file /etc/falco/k8s_audit_rules.yaml:
Sun May 23 23:53:15 2021: Starting internal webserver, listening on port 8765




23:53:30.491825091: Notice A shell was spawned in a container with an attached terminal (user=root user_loginuid=-1 k8s_apache_apache_default_3ece2efb-fe49-4111-899f-10d38a61bab6_0 (id=84dd6fe8a9ad) shell=bash parent=runc cmdline=bash terminal=34816 container_id=84dd6fe8a9ad image=httpd)


root@node2:~# cd /etc/falco/
root@node2:/etc/falco# ls
falco_rules.local.yaml  falco_rules.yaml  falco.yaml  k8s_audit_rules.yaml  rules.available  rules.d


root@node2:/etc/falco# grep -r "A shell was spawned in a container with an attached terminal" *
falco_rules.yaml:    A shell was spawned in a container with an attached terminal (user=%user.name user_loginuid=%user.loginuid %container.info

#更新配置
root@node2:/etc/falco# cat falco_rules.local.yaml
- rule: Terminal shell in container
  desc: A shell was used as the entrypoint/exec point into a container with an attached terminal.
  condition: >
    spawned_process and container
    and shell_procs and proc.tty != 0
    and container_entrypoint
    and not user_expected_terminal_shell_in_container_conditions
  output: >
    %evt.time,%user.name,%container.name,%container.id
    shell=%proc.name parent=%proc.pname cmdline=%proc.cmdline terminal=%proc.tty container_id=%container.id image=%container.image.repository)
  priority: WARNING
  tags: [container, shell, mitre_execution]




root@master:~/cks/runtime-security# k exec -ti apache -- bash
root@apache:/usr/local/apache2#




root@node2:/etc/falco# falco
Mon May 24 00:07:13 2021: Falco version 0.28.1 (driver version 5c0b863ddade7a45568c0ac97d037422c9efb750)
Mon May 24 00:07:13 2021: Falco initialized with configuration file /etc/falco/falco.yaml
Mon May 24 00:07:13 2021: Loading rules from file /etc/falco/falco_rules.yaml:
Mon May 24 00:07:13 2021: Loading rules from file /etc/falco/falco_rules.local.yaml:  #配置生效
Mon May 24 00:07:13 2021: Loading rules from file /etc/falco/k8s_audit_rules.yaml:
Mon May 24 00:07:14 2021: Starting internal webserver, listening on port 8765
00:07:30.297671117: Warning Shell history had been deleted or renamed (user=root user_loginuid=-1 type=openat command=bash fd.name=/root/.bash_history name=/root/.bash_history path=<NA> oldpath=<NA> k8s_apache_apache_default_3ece2efb-fe49-4111-899f-10d38a61bab6_0 (id=84dd6fe8a9ad))

格式改变
00:07:33.763063865: Warning 00:07:33.763063865,root,k8s_apache_apache_default_3ece2efb-fe49-4111-899f-10d38a61bab6_0,84dd6fe8a9ad shell=bash parent=runc cmdline=bash terminal=34816 container_id=84dd6fe8a9ad image=httpd)

```

