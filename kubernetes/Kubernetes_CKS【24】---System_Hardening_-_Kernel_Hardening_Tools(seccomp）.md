



---

k8s官网：

 - [https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet/](https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet/)
 - [https://kubernetes.io/docs/tutorials/clusters/seccomp/](https://kubernetes.io/docs/tutorials/clusters/seccomp/)

---
##  1. Seccomp介绍
Seccomp 代表安全计算模式，自 2.6.12 版本以来一直是 Linux 内核的一个特性。它可用于沙盒进程的特权，限制它能够从用户空间向内核进行的调用。Kubernetes 允许您自动应用加载到 节点 到您的 Pod 和容器。

确定工作负载所需的权限可能很困难。在本教程中，您将了解如何将 seccomp 配置文件加载到本地 Kubernetes 集群、如何将它们应用到 Pod，以及如何开始制作配置文件，只为您的容器进程提供必要的权限

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210525101359576.png?shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
## 2. 目标

 - 了解如何在节点上加载 seccomp 配置文件
 - 了解如何将 seccomp 配置文件应用于容器
 - 观察容器进程对系统调用的审计
 - 指定缺失配置文件时观察行为
 - 观察违反 seccomp 配置文件的情况
 - 了解如何创建细粒度的 seccomp 配置文件
 - 了解如何应用容器运行时默认 seccomp 配置文件


##  3. Seccomp for Docker Nginx

```bash
root@master:~/cks/apparmor# docker run --security-opt seccomp=default.json nginx
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
10-listen-on-ipv6-by-default.sh: info: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
```

##  4. 启用 RuntimeDefault
SeccompDefault 是一个可选的 [kubelet 特性门控](https://kubernetes.io/zh/docs/reference/command-line-tools-reference/feature-gates/)， 相应地，--seccomp-default 是此特性门控的 [命令行标志](https://kubernetes.io/zh/docs/reference/command-line-tools-reference/kubelet/)。 必须同时启用两者才能使用该功能。

如果启用，kubelet 将默认使用 RuntimeDefault seccomp 配置， 而不是使用 Unconfined（禁用 seccomp）模式，该配置由容器运行时定义。 默认配置旨在提供一组强大的安全默认值设置，同时避免影响工作负载的功能。 不同的容器运行时之间及其不同的发布版本之间的默认配置可能不同， 例如在比较 CRI-O 和 containerd 的配置文件时（就会发现这点）。

某些工作负载可能相比其他工作负载需要更少的系统调用限制。 这意味着即使使用 RuntimeDefault 配置文件，它们也可能在运行时失败。 要处理此类失效，你可以：

 - 将工作负载显式运行为 Unconfined。
 - 禁用节点的 SeccompDefault 功能。 还要确保工作负载被安排在禁用该功能的节点上。
 - 为工作负载创建自定义 seccomp 配置文件。

由于该功能处于 `alpha` 状态，因此默认情况下是被禁用的。要启用它， 请将标志 `--feature-gates=SeccompDefault=true --seccomp-default` 传递给 kubelet CLI 或通过 kubelet 配置文件启用它。 要在 kind 中启用特性门控， 请确保 kind 提供所需的最低 Kubernetes 版本并 在 kind 配置中 启用 `SeccompDefault` 功能：

```bash
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
featureGates:
  SeccompDefault: true
```

##  5. 创建 Seccomp 文件

```bash
vim audit.json
{
    "defaultAction": "SCMP_ACT_LOG"
}

vim violation.json
{
    "defaultAction": "SCMP_ACT_ERRNO"
}

vim fine-grained.json
{
    "defaultAction": "SCMP_ACT_ERRNO",
    "architectures": [
        "SCMP_ARCH_X86_64",
        "SCMP_ARCH_X86",
        "SCMP_ARCH_X32"
    ],
    "syscalls": [
        {
            "names": [
                "accept4",
                "epoll_wait",
                "pselect6",
                "futex",
                "madvise",
                "epoll_ctl",
                "getsockname",
                "setsockopt",
                "vfork",
                "mmap",
                "read",
                "write",
                "close",
                "arch_prctl",
                "sched_getaffinity",
                "munmap",
                "brk",
                "rt_sigaction",
                "rt_sigprocmask",
                "sigaltstack",
                "gettid",
                "clone",
                "bind",
                "socket",
                "openat",
                "readlinkat",
                "exit_group",
                "epoll_create1",
                "listen",
                "rt_sigreturn",
                "sched_yield",
                "clock_gettime",
                "connect",
                "dup2",
                "epoll_pwait",
                "execve",
                "exit",
                "fcntl",
                "getpid",
                "getuid",
                "ioctl",
                "mprotect",
                "nanosleep",
                "open",
                "poll",
                "recvfrom",
                "sendto",
                "set_tid_address",
                "setitimer",
                "writev"
            ],
            "action": "SCMP_ACT_ALLOW"
        }
    ]
}

```
##  6. Kind 创建一个本地 Kubernetes 集群

```bash
apiVersion: kind.x-k8s.io/v1alpha4
kind: Cluster
nodes:
- role: control-plane
  extraMounts:
  - hostPath: "./profiles"
    containerPath: "/var/lib/kubelet/seccomp/profiles"
```

```bash
$ kind create cluster --name kind2 --config kind2.yaml 

$ docker exec -it 6a96207fed4b ls /var/lib/kubelet/seccomp/profiles
audit.json  fine-grained.json  violation.json
```

 
##  7. seccomp 配置文件创建 Pod 以进行系统调用审核

```bash
$ docker pull hashicorp/http-echo:0.2.3
$ kind load docker-image hashicorp/http-echo:0.2.3 --name kind2
$ vim audit-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: audit-pod
  labels:
    app: audit-pod
spec:
  securityContext:
    seccompProfile:
      type: Localhost
      localhostProfile: profiles/audit.json
  containers:
  - name: test-container
    image: hashicorp/http-echo:0.2.3
    args:
    - "-text=just made some syscalls!"
    securityContext:
      allowPrivilegeEscalation: false


$ kubectl apply -f audit-pod.yaml

$ kubectl get pod/audit-pod
NAME        READY   STATUS    RESTARTS   AGE
audit-pod   1/1     Running   0          30s
```
为了能够与该容器公开的端点进行交互，请创建一个 NodePort 服务， 该服务允许从 kind 控制平面容器内部访问该端点。

```bash
$ kubectl expose pod/audit-pod --type NodePort --port 5678
$ kubectl get svc/audit-pod
NAME        TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
audit-pod   NodePort   10.111.36.142   <none>        5678:32373/TCP   72s

$ docker exec -it 6a96207fed4b curl localhost:32373
just made some syscalls!

$ tail -f /var/log/syslog | grep 'http-echo'
Jul  6 15:37:40 my-machine kernel: [369128.669452] audit: type=1326 audit(1594067860.484:14536): auid=4294967295 uid=0 gid=0 ses=4294967295 pid=29064 comm="http-echo" exe="/http-echo" sig=0 arch=c000003e syscall=51 compat=0 ip=0x46fe1f code=0x7ffc0000
Jul  6 15:37:40 my-machine kernel: [369128.669453] audit: type=1326 audit(1594067860.484:14537): auid=4294967295 uid=0 gid=0 ses=4294967295 pid=29064 comm="http-echo" exe="/http-echo" sig=0 arch=c000003e syscall=54 compat=0 ip=0x46fdba code=0x7ffc0000
Jul  6 15:37:40 my-machine kernel: [369128.669455] audit: type=1326 audit(1594067860.484:14538): auid=4294967295 uid=0 gid=0 ses=4294967295 pid=29064 comm="http-echo" exe="/http-echo" sig=0 arch=c000003e syscall=202 compat=0 ip=0x455e53 code=0x7ffc0000
Jul  6 15:37:40 my-machine kernel: [369128.669456] audit: type=1326 audit(1594067860.484:14539): auid=4294967295 uid=0 gid=0 ses=4294967295 pid=29064 comm="http-echo" exe="/http-echo" sig=0 arch=c000003e syscall=288 compat=0 ip=0x46fdba code=0x7ffc0000
Jul  6 15:37:40 my-machine kernel: [369128.669517] audit: type=1326 audit(1594067860.484:14540): auid=4294967295 uid=0 gid=0 ses=4294967295 pid=29064 comm="http-echo" exe="/http-echo" sig=0 arch=c000003e syscall=0 compat=0 ip=0x46fd44 code=0x7ffc0000
Jul  6 15:37:40 my-machine kernel: [369128.669519] audit: type=1326 audit(1594067860.484:14541): auid=4294967295 uid=0 gid=0 ses=4294967295 pid=29064 comm="http-echo" exe="/http-echo" sig=0 arch=c000003e syscall=270 compat=0 ip=0x4559b1 code=0x7ffc0000
Jul  6 15:38:40 my-machine kernel: [369188.671648] audit: type=1326 audit(1594067920.488:14559): auid=4294967295 uid=0 gid=0 ses=4294967295 pid=29064 comm="http-echo" exe="/http-echo" sig=0 arch=c000003e syscall=270 compat=0 ip=0x4559b1 code=0x7ffc0000
Jul  6 15:38:40 my-machine kernel: [369188.671726] audit: type=1326 audit(1594067920.488:14560): auid=4294967295 uid=0 gid=0 ses=4294967295 pid=29064 comm="http-echo" exe="/http-echo" sig=0 arch=c000003e syscall=202 compat=0 ip=0x455e53 code=0x7ffc0000
```
通过查看每一行上的 syscall= 条目，你可以开始了解 http-echo 进程所需的系统调用。 尽管这些不太可能包含它使用的所有系统调用，但它可以作为该容器的 seccomp 配置文件的基础。

##  8. 使用导致违规的 seccomp 配置文件创建 Pod

```bash
$  vim violation-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: violation-pod
  labels:
    app: violation-pod
spec:
  securityContext:
    seccompProfile:
      type: Localhost
      localhostProfile: profiles/violation.json
  containers:
  - name: test-container
    image: hashicorp/http-echo:0.2.3
    args:
    - "-text=just made some syscalls!"
    securityContext:
      allowPrivilegeEscalation: false


$ kubectl apply -f violation-pod.yaml
$ kubectl get pod/violation-pod
NAME            READY   STATUS             RESTARTS   AGE
violation-pod   0/1     CrashLoopBackOff   1          6s
```
如上例所示，http-echo 进程需要大量的系统调用。通过设置 `"defaultAction": "SCMP_ACT_ERRNO"`， 来指示 seccomp 在任何系统调用上均出错。


##  9. 设置仅允许需要的系统调用的 seccomp 配置文件来创建 Pod
如果你看一下 `fine-pod.json` 文件，你会注意到在第一个示例中配置文件设置为 `"defaultAction": "SCMP_ACT_LOG"` 的一些系统调用。 现在，配置文件设置为 `"defaultAction": "SCMP_ACT_ERRNO"`，但是在 `"action": "SCMP_ACT_ALLOW"` 块中明确允许一组系统调用。 理想情况下，容器将成功运行，并且你将不会看到任何发送到 syslog 的消息。

```bash
$ vim  fine-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: fine-pod
  labels:
    app: fine-pod
spec:
  securityContext:
    seccompProfile:
      type: Localhost
      localhostProfile: profiles/fine-grained.json
  containers:
  - name: test-container
    image: hashicorp/http-echo:0.2.3
    args:
    - "-text=just made some syscalls!"
    securityContext:
      allowPrivilegeEscalation: false


$ kubectl apply -f fine-pod.yaml

$ kubectl get pod/fine-pod
NAME        READY   STATUS    RESTARTS   AGE
fine-pod   1/1     Running   0          30s

$ tail -f /var/log/syslog | grep 'http-echo'

$ kubectl expose pod/fine-pod --type NodePort --port 5678
$ kubectl get svc/fine-pod
NAME        TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
fine-pod    NodePort   10.111.36.142   <none>        5678:32373/TCP   72s

$ docker exec -it 6a96207fed4b curl localhost:32373
just made some syscalls!
```


##  10. 容器运行时默认的 seccomp 配置文件创建 Pod

```bash
apiVersion: v1
kind: Pod
metadata:
  name: audit-pod
  labels:
    app: audit-pod
spec:
  securityContext:
    seccompProfile:
      type: RuntimeDefault
  containers:
  - name: test-container
    image: hashicorp/http-echo:0.2.3
    args:
    - "-text=just made some syscalls!"
    securityContext:
      allowPrivilegeEscalation: false
```




##  11. 自定义 Seccomp for Kubernetes Nginx





```bash

root@node2:~# mkdir /var/lib/kubelet/seccomp
root@node2:~# cat /var/lib/kubelet/seccomp/default.json 
{
  "defaultAction": "SCMP_ACT_ERRNO",
  "archMap": [
    {
      "architecture": "SCMP_ARCH_X86_64",
      "subArchitectures": [
        "SCMP_ARCH_X86",
        "SCMP_ARCH_X32"
      ]
    },
    {
      "architecture": "SCMP_ARCH_AARCH64",
      "subArchitectures": [
        "SCMP_ARCH_ARM"
      ]
    },
    {
      "architecture": "SCMP_ARCH_MIPS64",
      "subArchitectures": [
        "SCMP_ARCH_MIPS",
        "SCMP_ARCH_MIPS64N32"
      ]
    },
    {
      "architecture": "SCMP_ARCH_MIPS64N32",
      "subArchitectures": [
        "SCMP_ARCH_MIPS",
        "SCMP_ARCH_MIPS64"
      ]
    },
    {
      "architecture": "SCMP_ARCH_MIPSEL64",
      "subArchitectures": [
        "SCMP_ARCH_MIPSEL",
        "SCMP_ARCH_MIPSEL64N32"
      ]
    },
    {
      "architecture": "SCMP_ARCH_MIPSEL64N32",
      "subArchitectures": [
        "SCMP_ARCH_MIPSEL",
        "SCMP_ARCH_MIPSEL64"
      ]
    },
    {
      "architecture": "SCMP_ARCH_S390X",
      "subArchitectures": [
        "SCMP_ARCH_S390"
      ]
    }
  ],
  "syscalls": [
    {
      "names": [
        "accept",
        "accept4",
        "access",
        "adjtimex",
        "alarm",
        "bind",
        "brk",
        "capget",
        "capset",
        "chdir",
        "chmod",
        "chown",
        "chown32",
        "clock_adjtime",
        "clock_adjtime64",
        "clock_getres",
        "clock_getres_time64",
        "clock_gettime",
        "clock_gettime64",
        "clock_nanosleep",
        "clock_nanosleep_time64",
        "close",
        "connect",
        "copy_file_range",
        "creat",
        "dup",
        "dup2",
        "dup3",
        "epoll_create",
        "epoll_create1",
        "epoll_ctl",
        "epoll_ctl_old",
        "epoll_pwait",
        "epoll_wait",
        "epoll_wait_old",
        "eventfd",
        "eventfd2",
        "execve",
        "execveat",
        "exit",
        "exit_group",
        "faccessat",
        "faccessat2",
        "fadvise64",
        "fadvise64_64",
        "fallocate",
        "fanotify_mark",
        "fchdir",
        "fchmod",
        "fchmodat",
        "fchown",
        "fchown32",
        "fchownat",
        "fcntl",
        "fcntl64",
        "fdatasync",
        "fgetxattr",
        "flistxattr",
        "flock",
        "fork",
        "fremovexattr",
        "fsetxattr",
        "fstat",
        "fstat64",
        "fstatat64",
        "fstatfs",
        "fstatfs64",
        "fsync",
        "ftruncate",
        "ftruncate64",
        "futex",
        "futex_time64",
        "futimesat",
        "getcpu",
        "getcwd",
        "getdents",
        "getdents64",
        "getegid",
        "getegid32",
        "geteuid",
        "geteuid32",
        "getgid",
        "getgid32",
        "getgroups",
        "getgroups32",
        "getitimer",
        "getpeername",
        "getpgid",
        "getpgrp",
        "getpid",
        "getppid",
        "getpriority",
        "getrandom",
        "getresgid",
        "getresgid32",
        "getresuid",
        "getresuid32",
        "getrlimit",
        "get_robust_list",
        "getrusage",
        "getsid",
        "getsockname",
        "getsockopt",
        "get_thread_area",
        "gettid",
        "gettimeofday",
        "getuid",
        "getuid32",
        "getxattr",
        "inotify_add_watch",
        "inotify_init",
        "inotify_init1",
        "inotify_rm_watch",
        "io_cancel",
        "ioctl",
        "io_destroy",
        "io_getevents",
        "io_pgetevents",
        "io_pgetevents_time64",
        "ioprio_get",
        "ioprio_set",
        "io_setup",
        "io_submit",
        "io_uring_enter",
        "io_uring_register",
        "io_uring_setup",
        "ipc",
        "kill",
        "lchown",
        "lchown32",
        "lgetxattr",
        "link",
        "linkat",
        "listen",
        "listxattr",
        "llistxattr",
        "_llseek",
        "lremovexattr",
        "lseek",
        "lsetxattr",
        "lstat",
        "lstat64",
        "madvise",
        "membarrier",
        "memfd_create",
        "mincore",
        "mkdir",
        "mkdirat",
        "mknod",
        "mknodat",
        "mlock",
        "mlock2",
        "mlockall",
        "mmap",
        "mmap2",
        "mprotect",
        "mq_getsetattr",
        "mq_notify",
        "mq_open",
        "mq_timedreceive",
        "mq_timedreceive_time64",
        "mq_timedsend",
        "mq_timedsend_time64",
        "mq_unlink",
        "mremap",
        "msgctl",
        "msgget",
        "msgrcv",
        "msgsnd",
        "msync",
        "munlock",
        "munlockall",
        "munmap",
        "nanosleep",
        "newfstatat",
        "_newselect",
        "open",
        "openat",
        "openat2",
        "pause",
        "pipe",
        "pipe2",
        "poll",
        "ppoll",
        "ppoll_time64",
        "prctl",
        "pread64",
        "preadv",
        "preadv2",
        "prlimit64",
        "pselect6",
        "pselect6_time64",
        "pwrite64",
        "pwritev",
        "pwritev2",
        "read",
        "readahead",
        "readlink",
        "readlinkat",
        "readv",
        "recv",
        "recvfrom",
        "recvmmsg",
        "recvmmsg_time64",
        "recvmsg",
        "remap_file_pages",
        "removexattr",
        "rename",
        "renameat",
        "renameat2",
        "restart_syscall",
        "rmdir",
        "rseq",
        "rt_sigaction",
        "rt_sigpending",
        "rt_sigprocmask",
        "rt_sigqueueinfo",
        "rt_sigreturn",
        "rt_sigsuspend",
        "rt_sigtimedwait",
        "rt_sigtimedwait_time64",
        "rt_tgsigqueueinfo",
        "sched_getaffinity",
        "sched_getattr",
        "sched_getparam",
        "sched_get_priority_max",
        "sched_get_priority_min",
        "sched_getscheduler",
        "sched_rr_get_interval",
        "sched_rr_get_interval_time64",
        "sched_setaffinity",
        "sched_setattr",
        "sched_setparam",
        "sched_setscheduler",
        "sched_yield",
        "seccomp",
        "select",
        "semctl",
        "semget",
        "semop",
        "semtimedop",
        "semtimedop_time64",
        "send",
        "sendfile",
        "sendfile64",
        "sendmmsg",
        "sendmsg",
        "sendto",
        "setfsgid",
        "setfsgid32",
        "setfsuid",
        "setfsuid32",
        "setgid",
        "setgid32",
        "setgroups",
        "setgroups32",
        "setitimer",
        "setpgid",
        "setpriority",
        "setregid",
        "setregid32",
        "setresgid",
        "setresgid32",
        "setresuid",
        "setresuid32",
        "setreuid",
        "setreuid32",
        "setrlimit",
        "set_robust_list",
        "setsid",
        "setsockopt",
        "set_thread_area",
        "set_tid_address",
        "setuid",
        "setuid32",
        "setxattr",
        "shmat",
        "shmctl",
        "shmdt",
        "shmget",
        "shutdown",
        "sigaltstack",
        "signalfd",
        "signalfd4",
        "sigprocmask",
        "sigreturn",
        "socket",
        "socketcall",
        "socketpair",
        "splice",
        "stat",
        "stat64",
        "statfs",
        "statfs64",
        "statx",
        "symlink",
        "symlinkat",
        "sync",
        "sync_file_range",
        "syncfs",
        "sysinfo",
        "tee",
        "tgkill",
        "time",
        "timer_create",
        "timer_delete",
        "timer_getoverrun",
        "timer_gettime",
        "timer_gettime64",
        "timer_settime",
        "timer_settime64",
        "timerfd_create",
        "timerfd_gettime",
        "timerfd_gettime64",
        "timerfd_settime",
        "timerfd_settime64",
        "times",
        "tkill",
        "truncate",
        "truncate64",
        "ugetrlimit",
        "umask",
        "uname",
        "unlink",
        "unlinkat",
        "utime",
        "utimensat",
        "utimensat_time64",
        "utimes",
        "vfork",
        "vmsplice",
        "wait4",
        "waitid",
        "waitpid",
        "write",
        "writev"
      ],
      "action": "SCMP_ACT_ALLOW",
      "args": [],
      "comment": "",
      "includes": {},
      "excludes": {}
    },
    {
      "names": [
        "ptrace"
      ],
      "action": "SCMP_ACT_ALLOW",
      "args": null,
      "comment": "",
      "includes": {
        "minKernel": "4.8"
      },
      "excludes": {}
    },
    {
      "names": [
        "personality"
      ],
      "action": "SCMP_ACT_ALLOW",
      "args": [
        {
          "index": 0,
          "value": 0,
          "op": "SCMP_CMP_EQ"
        }
      ],
      "comment": "",
      "includes": {},
      "excludes": {}
    },
    {
      "names": [
        "personality"
      ],
      "action": "SCMP_ACT_ALLOW",
      "args": [
        {
          "index": 0,
          "value": 8,
          "op": "SCMP_CMP_EQ"
        }
      ],
      "comment": "",
      "includes": {},
      "excludes": {}
    },
    {
      "names": [
        "personality"
      ],
      "action": "SCMP_ACT_ALLOW",
      "args": [
        {
          "index": 0,
          "value": 131072,
          "op": "SCMP_CMP_EQ"
        }
      ],
      "comment": "",
      "includes": {},
      "excludes": {}
    },
    {
      "names": [
        "personality"
      ],
      "action": "SCMP_ACT_ALLOW",
      "args": [
        {
          "index": 0,
          "value": 131080,
          "op": "SCMP_CMP_EQ"
        }
      ],
      "comment": "",
      "includes": {},
      "excludes": {}
    },
    {
      "names": [
        "personality"
      ],
      "action": "SCMP_ACT_ALLOW",
      "args": [
        {
          "index": 0,
          "value": 4294967295,
          "op": "SCMP_CMP_EQ"
        }
      ],
      "comment": "",
      "includes": {},
      "excludes": {}
    },
    {
      "names": [
        "sync_file_range2"
      ],
      "action": "SCMP_ACT_ALLOW",
      "args": [],
      "comment": "",
      "includes": {
        "arches": [
          "ppc64le"
        ]
      },
      "excludes": {}
    },
    {
      "names": [
        "arm_fadvise64_64",
        "arm_sync_file_range",
        "sync_file_range2",
        "breakpoint",
        "cacheflush",
        "set_tls"
      ],
      "action": "SCMP_ACT_ALLOW",
      "args": [],
      "comment": "",
      "includes": {
        "arches": [
          "arm",
          "arm64"
        ]
      },
      "excludes": {}
    },
    {
      "names": [
        "arch_prctl"
      ],
      "action": "SCMP_ACT_ALLOW",
      "args": [],
      "comment": "",
      "includes": {
        "arches": [
          "amd64",
          "x32"
        ]
      },
      "excludes": {}
    },
    {
      "names": [
        "modify_ldt"
      ],
      "action": "SCMP_ACT_ALLOW",
      "args": [],
      "comment": "",
      "includes": {
        "arches": [
          "amd64",
          "x32",
          "x86"
        ]
      },
      "excludes": {}
    },
    {
      "names": [
        "s390_pci_mmio_read",
        "s390_pci_mmio_write",
        "s390_runtime_instr"
      ],
      "action": "SCMP_ACT_ALLOW",
      "args": [],
      "comment": "",
      "includes": {
        "arches": [
          "s390",
          "s390x"
        ]
      },
      "excludes": {}
    },
    {
      "names": [
        "open_by_handle_at"
      ],
      "action": "SCMP_ACT_ALLOW",
      "args": [],
      "comment": "",
      "includes": {
        "caps": [
          "CAP_DAC_READ_SEARCH"
        ]
      },
      "excludes": {}
    },
    {
      "names": [
        "bpf",
        "clone",
        "fanotify_init",
        "lookup_dcookie",
        "mount",
        "name_to_handle_at",
        "perf_event_open",
        "quotactl",
        "setdomainname",
        "sethostname",
        "setns",
        "syslog",
        "umount",
        "umount2",
        "unshare"
      ],
      "action": "SCMP_ACT_ALLOW",
      "args": [],
      "comment": "",
      "includes": {
        "caps": [
          "CAP_SYS_ADMIN"
        ]
      },
      "excludes": {}
    },
    {
      "names": [
        "clone"
      ],
      "action": "SCMP_ACT_ALLOW",
      "args": [
        {
          "index": 0,
          "value": 2114060288,
          "op": "SCMP_CMP_MASKED_EQ"
        }
      ],
      "comment": "",
      "includes": {},
      "excludes": {
        "caps": [
          "CAP_SYS_ADMIN"
        ],
        "arches": [
          "s390",
          "s390x"
        ]
      }
    },
    {
      "names": [
        "clone"
      ],
      "action": "SCMP_ACT_ALLOW",
      "args": [
        {
          "index": 1,
          "value": 2114060288,
          "op": "SCMP_CMP_MASKED_EQ"
        }
      ],
      "comment": "s390 parameter ordering for clone is different",
      "includes": {
        "arches": [
          "s390",
          "s390x"
        ]
      },
      "excludes": {
        "caps": [
          "CAP_SYS_ADMIN"
        ]
      }
    },
    {
      "names": [
        "reboot"
      ],
      "action": "SCMP_ACT_ALLOW",
      "args": [],
      "comment": "",
      "includes": {
        "caps": [
          "CAP_SYS_BOOT"
        ]
      },
      "excludes": {}
    },
    {
      "names": [
        "chroot"
      ],
      "action": "SCMP_ACT_ALLOW",
      "args": [],
      "comment": "",
      "includes": {
        "caps": [
          "CAP_SYS_CHROOT"
        ]
      },
      "excludes": {}
    },
    {
      "names": [
        "delete_module",
        "init_module",
        "finit_module"
      ],
      "action": "SCMP_ACT_ALLOW",
      "args": [],
      "comment": "",
      "includes": {
        "caps": [
          "CAP_SYS_MODULE"
        ]
      },
      "excludes": {}
    },
    {
      "names": [
        "acct"
      ],
      "action": "SCMP_ACT_ALLOW",
      "args": [],
      "comment": "",
      "includes": {
        "caps": [
          "CAP_SYS_PACCT"
        ]
      },
      "excludes": {}
    },
    {
      "names": [
        "kcmp",
        "process_vm_readv",
        "process_vm_writev",
        "ptrace"
      ],
      "action": "SCMP_ACT_ALLOW",
      "args": [],
      "comment": "",
      "includes": {
        "caps": [
          "CAP_SYS_PTRACE"
        ]
      },
      "excludes": {}
    },
    {
      "names": [
        "iopl",
        "ioperm"
      ],
      "action": "SCMP_ACT_ALLOW",
      "args": [],
      "comment": "",
      "includes": {
        "caps": [
          "CAP_SYS_RAWIO"
        ]
      },
      "excludes": {}
    },
    {
      "names": [
        "settimeofday",
        "stime",
        "clock_settime"
      ],
      "action": "SCMP_ACT_ALLOW",
      "args": [],
      "comment": "",
      "includes": {
        "caps": [
          "CAP_SYS_TIME"
        ]
      },
      "excludes": {}
    },
    {
      "names": [
        "vhangup"
      ],
      "action": "SCMP_ACT_ALLOW",
      "args": [],
      "comment": "",
      "includes": {
        "caps": [
          "CAP_SYS_TTY_CONFIG"
        ]
      },
      "excludes": {}
    },
    {
      "names": [
        "get_mempolicy",
        "mbind",
        "set_mempolicy"
      ],
      "action": "SCMP_ACT_ALLOW",
      "args": [],
      "comment": "",
      "includes": {
        "caps": [
          "CAP_SYS_NICE"
        ]
      },
      "excludes": {}
    },
    {
      "names": [
        "syslog"
      ],
      "action": "SCMP_ACT_ALLOW",
      "args": [],
      "comment": "",
      "includes": {
        "caps": [
          "CAP_SYSLOG"
        ]
      },
      "excludes": {}
    }
  ]
}



root@master:~/cks/apparmor# cat pod2.yaml 
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: secure
  name: secure
spec:
  securityContext:
    seccompProfile:
      type: Localhost
      localhostProfile: profiles/audit.json
  containers:
  - image: nginx
    name: secure
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}



root@master:~/cks/apparmor# k get pods
NAME       READY   STATUS    RESTARTS   AGE
accessor   1/1     Running   0          26h
secure     1/1     Running   0          5h2m
root@master:~/cks/apparmor# k -f pod2.yaml delete --force --grace-period 0
warning: Immediate deletion does not wait for confirmation that the running resource has been terminated. The resource may continue to run on the cluster indefinitely.
pod "secure" force deleted
root@master:~/cks/apparmor# k -f pod2.yaml create
pod/secure created
root@master:~/cks/apparmor# k get pods 
NAME       READY   STATUS              RESTARTS   AGE
accessor   1/1     Running             0          26h
secure     0/1     ContainerCreating   0          4s

root@master:~/cks/apparmor# k get pods  -w
NAME       READY   STATUS                 RESTARTS   AGE
accessor   1/1     Running                0          26h
secure     0/1     CreateContainerError   0          23s




root@master:~/cks/apparmor# k describe pod secure
Name:         secure
Namespace:    default
Priority:     0
Node:         node2/192.168.211.42
Start Time:   Tue, 25 May 2021 04:56:56 -0700
Labels:       run=secure
Annotations:  cni.projectcalico.org/podIP: 10.244.104.13/32
              cni.projectcalico.org/podIPs: 10.244.104.13/32
              seccomp.security.alpha.kubernetes.io/pod: localhost/profiles/audit.json
Status:       Pending
IP:           10.244.104.13
IPs:
  IP:  10.244.104.13
Containers:
  secure:
    Container ID:   
    Image:          nginx
    Image ID:       
    Port:           <none>
    Host Port:      <none>
    State:          Waiting
      Reason:       CreateContainerError
    Ready:          False
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-4lh26 (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             False 
  ContainersReady   False 
  PodScheduled      True 
Volumes:
  default-token-4lh26:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-4lh26
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                 node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type     Reason     Age                From               Message
  ----     ------     ----               ----               -------
  Normal   Scheduled  32s                default-scheduler  Successfully assigned default/secure to node2
  Normal   Pulled     15s                kubelet            Successfully pulled image "nginx" in 19.058071723s
  Warning  Failed     15s                kubelet            Error: failed to generate security options for container "secure": failed to generate seccomp security options for container: cannot load seccomp profile "/var/lib/kubelet/seccomp/profiles/audit.json": open /var/lib/kubelet/seccomp/profiles/audit.json: no such file or directory
  Normal   Pulling    14s (x2 over 34s)  kubelet            Pulling image "nginx"


#修改pod2.yaml
root@master:~/cks/apparmor# cat pod2.yaml 
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: secure
  name: secure
spec:
  securityContext:
    seccompProfile:
      type: Localhost
      localhostProfile: default.json
  containers:
  - image: nginx
    name: secure
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}



root@master:~/cks/apparmor# k -f pod2.yaml delete --force --grace-period 0
warning: Immediate deletion does not wait for confirmation that the running resource has been terminated. The resource may continue to run on the cluster indefinitely.
pod "secure" force deleted
root@master:~/cks/apparmor# k -f pod2.yaml create
pod/secure created


#运行成功
root@master:~/cks/apparmor# k get pod secure
NAME     READY   STATUS    RESTARTS   AGE
secure   1/1     Running   0          78s


```








