```bash
$ podman container checkpoint suspicious_jones
ERRO[0000] container is not destroyed                   
ERRO[0000] criu failed: type NOTIFY errno 0
log file: /var/lib/containers/storage/overlay-containers/51eb5d6f607c97392426b717326e6fdfc5e066ecd617ac273a34d320e3e9afca/userdata/dump.log 
Error: failed to checkpoint container 51eb5d6f607c97392426b717326e6fdfc5e066ecd617ac273a34d320e3e9afca: `/usr/bin/runc checkpoint --image-path /var/lib/containers/storage/overlay-containers/51eb5d6f607c97392426b717326e6fdfc5e066ecd617ac273a34d320e3e9afca/userdata/checkpoint --work-path /var/lib/containers/storage/overlay-containers/51eb5d6f607c97392426b717326e6fdfc5e066ecd617ac273a34d320e3e9afca/userdata 51eb5d6f607c97392426b717326e6fdfc5e066ecd617ac273a34d320e3e9afca` failed: exit status 1
```
日志

```bash
[root@localhost criu]# cat /var/lib/containers/storage/overlay-containers/71a2e93a6e20847f40c0d0a6db6f5d589d14798fd2a4adaed1b75b72161d4c9f/userdata/dump.log 
(00.000050) Version: 3.16 (gitid v3.16)
(00.000078) Running on localhost.localdomain Linux 3.10.0-957.1.3.el7.x86_64 #1 SMP Thu Nov 29 14:49:43 UTC 2018 x86_64
(00.000082) Would overwrite RPC settings with values from /etc/criu/runc.conf
(00.000105) Loaded kdat cache from /run/criu.kdat
(00.000205) ========================================
(00.000211) Dumping processes (pid: 81507)
(00.000213) ========================================
(00.000224) rlimit: RLIMIT_NOFILE unlimited for self
(00.000230) Running pre-dump scripts
(00.000232) 	RPC
(00.000369) irmap: Searching irmap cache in work dir
(00.000384) No irmap-cache image
(00.000387) irmap: Searching irmap cache in parent
(00.000393) No parent images directory provided
(00.000397) irmap: No irmap cache
(00.000417) cpu: x86_family 6 x86_vendor_id GenuineIntel x86_model_id Intel(R) Core(TM) i7-6600U CPU @ 2.60GHz
(00.000422) cpu: fpu: xfeatures_mask 0x15 xsave_size 1088 xsave_size_max 1088 xsaves_size 960
(00.000431) cpu: fpu: x87 floating point registers     xstate_offsets      0 / 0      xstate_sizes    160 / 160   
(00.000434) cpu: fpu: AVX registers                    xstate_offsets    576 / 576    xstate_sizes    256 / 256   
(00.000436) cpu: fpu: MPX CSR                          xstate_offsets   1024 / 832    xstate_sizes     64 / 64    
(00.000439) cpu: fpu:1 fxsr:1 xsave:1 xsaveopt:1 xsavec:1 xgetbv1:0 xsaves:1
(00.000705) cg-prop: Parsing controller "cpu"
(00.000720) cg-prop: 	Strategy "replace"
(00.000724) cg-prop: 	Property "cpu.shares"
(00.000730) cg-prop: 	Property "cpu.cfs_period_us"
(00.000733) cg-prop: 	Property "cpu.cfs_quota_us"
(00.000735) cg-prop: 	Property "cpu.rt_period_us"
(00.000737) cg-prop: 	Property "cpu.rt_runtime_us"
(00.000739) cg-prop: Parsing controller "memory"
(00.000742) cg-prop: 	Strategy "replace"
(00.000744) cg-prop: 	Property "memory.limit_in_bytes"
(00.000746) cg-prop: 	Property "memory.memsw.limit_in_bytes"
(00.000748) cg-prop: 	Property "memory.swappiness"
(00.000750) cg-prop: 	Property "memory.soft_limit_in_bytes"
(00.000752) cg-prop: 	Property "memory.move_charge_at_immigrate"
(00.000754) cg-prop: 	Property "memory.oom_control"
(00.000757) cg-prop: 	Property "memory.use_hierarchy"
(00.000759) cg-prop: 	Property "memory.kmem.limit_in_bytes"
(00.000761) cg-prop: 	Property "memory.kmem.tcp.limit_in_bytes"
(00.000763) cg-prop: Parsing controller "cpuset"
(00.000765) cg-prop: 	Strategy "replace"
(00.000767) cg-prop: 	Property "cpuset.cpus"
(00.000769) cg-prop: 	Property "cpuset.mems"
(00.000771) cg-prop: 	Property "cpuset.memory_migrate"
(00.000774) cg-prop: 	Property "cpuset.cpu_exclusive"
(00.000776) cg-prop: 	Property "cpuset.mem_exclusive"
(00.000778) cg-prop: 	Property "cpuset.mem_hardwall"
(00.000780) cg-prop: 	Property "cpuset.memory_spread_page"
(00.000782) cg-prop: 	Property "cpuset.memory_spread_slab"
(00.000784) cg-prop: 	Property "cpuset.sched_load_balance"
(00.000786) cg-prop: 	Property "cpuset.sched_relax_domain_level"
(00.000788) cg-prop: Parsing controller "blkio"
(00.000790) cg-prop: 	Strategy "replace"
(00.000792) cg-prop: 	Property "blkio.weight"
(00.000794) cg-prop: Parsing controller "freezer"
(00.000797) cg-prop: 	Strategy "replace"
(00.000799) cg-prop: Parsing controller "perf_event"
(00.000801) cg-prop: 	Strategy "replace"
(00.000803) cg-prop: Parsing controller "net_cls"
(00.000805) cg-prop: 	Strategy "replace"
(00.000807) cg-prop: 	Property "net_cls.classid"
(00.000809) cg-prop: Parsing controller "net_prio"
(00.000812) cg-prop: 	Strategy "replace"
(00.000814) cg-prop: 	Property "net_prio.ifpriomap"
(00.000818) cg-prop: Parsing controller "pids"
(00.000821) cg-prop: 	Strategy "replace"
(00.000823) cg-prop: 	Property "pids.max"
(00.000825) cg-prop: Parsing controller "devices"
(00.000827) cg-prop: 	Strategy "replace"
(00.000829) cg-prop: 	Property "devices.list"
(00.000882) Preparing image inventory (version 1)
(00.000919) Add pid ns 1 pid 81677
(00.000931) Add net ns 2 pid 81677
(00.000941) Add ipc ns 3 pid 81677
(00.000951) Add uts ns 4 pid 81677
(00.000956) Add time ns 5 pid 81677
(00.000964) Add mnt ns 6 pid 81677
(00.000977) Add user ns 7 pid 81677
(00.000982) Add cgroup ns 8 pid 81677
(00.000986) cg: Dumping cgroups for 81677
(00.001018) cg:  `- New css ID 1
(00.001022) cg:     `- [blkio] -> [/] [0]
(00.001024) cg:     `- [cpuacct,cpu] -> [/] [0]
(00.001026) cg:     `- [cpuset] -> [/] [0]
(00.001028) cg:     `- [devices] -> [/user.slice] [0]
(00.001031) cg:     `- [freezer] -> [/] [0]
(00.001033) cg:     `- [hugetlb] -> [/] [0]
(00.001035) cg:     `- [memory] -> [/] [0]
(00.001037) cg:     `- [name=systemd] -> [/user.slice/user-0.slice/session-9.scope] [0]
(00.001039) cg:     `- [net_prio,net_cls] -> [/] [0]
(00.001041) cg:     `- [perf_event] -> [/] [0]
(00.001043) cg:     `- [pids] -> [/user.slice] [0]
(00.001045) cg: Set 1 is criu one
(00.001088) Detected cgroup V1 freezer
(00.001090) freezing processes: 100000 attempts with 100 ms steps
(00.001107) freezer.state=THAWED
(00.001121) freezer.state=FREEZING
(00.103742) freezer.state=FROZEN
(00.103834) freezing processes: 1 attempts done
(00.104121) SEIZE 81507: success
(00.104190) SEIZE 81671: success
(00.106528) Error (compel/src/lib/ptrace.c:27): suspending seccomp failed: Invalid argument
(00.106674) Unlock network
(00.106727) Unfreezing tasks into 1
(00.106742) 	Unseizing 81507 into 1
(00.106754) Error (compel/src/lib/infect.c:355): Unable to detach from 81507: No such process
(00.106827) Error (criu/cr-dump.c:1781): Dumping FAILED.
```

```bash
[root@localhost criu]# criu check --all
Error (criu/cr-check.c:801): Kernel doesn't support PTRACE_O_SUSPEND_SECCOMP
Error (criu/cr-check.c:845): Dumping seccomp filters not supported: Input/output error
Error (criu/cr-check.c:1064): cgroupns not supported. This is not fatal.
Warn  (criu/cr-check.c:1284): Do not have API to map vDSO - will use mremap() to restore vDSO
Warn  (criu/cr-check.c:1273): clone3() with set_tid not supported
Error (criu/cr-check.c:1315): Time namespaces are not supported
Error (criu/cr-check.c:1325): IFLA_NEW_IFINDEX isn't supported
Warn  (criu/cr-check.c:1342): Pidfd store requires pidfd_open syscall which is not supported
Warn  (criu/cr-check.c:1368): Nftables based locking requires libnftables and set concatenations support
Warn  (criu/cr-check.c:1202): compat_cr is not supported. Requires kernel >= v4.12
Looks good but some kernel features are missing
which, depending on your process tree, may cause
dump or restore failure.
```
[**升级centos内核**](https://ghostwritten.blog.csdn.net/article/details/121618058)

竟然还报错
```bash
$ podman container checkpoint suspicious_jones
ERRO[0000] container is not destroyed                   
ERRO[0000] criu failed: type NOTIFY errno 0
log file: /var/lib/containers/storage/overlay-containers/51eb5d6f607c97392426b717326e6fdfc5e066ecd617ac273a34d320e3e9afca/userdata/dump.log 
Error: failed to checkpoint container 51eb5d6f607c97392426b717326e6fdfc5e066ecd617ac273a34d320e3e9afca: `/usr/bin/runc checkpoint --image-path /var/lib/containers/storage/overlay-containers/51eb5d6f607c97392426b717326e6fdfc5e066ecd617ac273a34d320e3e9afca/userdata/checkpoint --work-path /var/lib/containers/storage/overlay-containers/51eb5d6f607c97392426b717326e6fdfc5e066ecd617ac273a34d320e3e9afca/userdata 51eb5d6f607c97392426b717326e6fdfc5e066ecd617ac273a34d320e3e9afca` failed: exit status 1
```
log

```bash
$ cat /var/lib/containers/storage/overlay-containers/71a2e93a6e20847f40c0d0a6db6f5d589d14798fd2a4adaed1b75b72161d4c9f/userdata/dump.log 
......................
(03.637047) Set up parasite blob using memfd
(03.637084) Putting parasite blob into 0x7f6b0633f000->0x7fed3152d000
(03.637147) Dumping general registers for 1511 in native mode
(03.637151) Dumping GP/FPU registers for 1511
(03.637167) x86: xsave runtime structure
(03.637169) x86: -----------------------
(03.637170) x86: cwd:0x37f swd:0 twd:0 fop:0 mxcsr:0x1f80 mxcsr_mask:0xffff
(03.637172) x86: magic1:0x46505853 extended_size:1092 xstate_bv:0x17 xstate_size:1088
(03.637174) x86: xstate_bv: 0x17
(03.637175) x86: -----------------------
(03.637177) Putting tsock into pid 1511
(03.637521) Error (compel/src/lib/infect.c:650): Unable to connect a transport socket: Permission denied
(03.637533) Error (compel/src/lib/ptrace.c:73): POKEDATA failed: No such process
(03.637536) Error (compel/src/lib/ptrace.c:96): Can't poke 1511 @ 0x55a992213000 from 0x7ffee3586570 sized 8
(03.637538) Error (compel/src/lib/ptrace.c:73): POKEDATA failed: No such process
(03.637540) Error (compel/src/lib/ptrace.c:100): Can't restore the original data with poke
(03.637541) Error (compel/src/lib/infect.c:574): Can't inject syscall blob (pid: 1511)
(03.637543) Warn  (criu/parasite-syscall.c:629): Can't cure failed infection
(03.637547) Error (criu/cr-dump.c:1296): Can't infect (pid: 1511) with parasite
(03.638050) Unlock network
(03.638113) Running network-unlock scripts
(03.638117) 	RPC
(03.641597) Unfreezing tasks into 1
(03.641609) 	Unseizing 1511 into 1
(03.641613) Error (compel/src/lib/infect.c:355): Unable to detach from 1511: No such process
(03.641619) 	Unseizing 1522 into 1
(03.642000) 	Unseizing 1523 into 1
(03.642277) 	Unseizing 1524 into 1
```
so, i think about selinux

```bash
[root@localhost ~]# getenforce 
Enforcing
[root@localhost ~]# setenforce 0
[root@localhost ~]# getenforce 
Permissive
[root@localhost ~]# podman container checkpoint exciting_neumann
2135a176f1b5f75668836c3a4e374cd1bcfc17545414ba5fb41be6558078f470
```

