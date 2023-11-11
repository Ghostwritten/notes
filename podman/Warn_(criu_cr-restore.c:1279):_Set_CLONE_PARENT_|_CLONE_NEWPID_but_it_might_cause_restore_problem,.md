```bash
$ podman container restore exciting_neumann
Error: failed to restore container 2135a176f1b5f75668836c3a4e374cd1bcfc17545414ba5fb41be6558078f470: criu failed: type NOTIFY errno 17
log file: /var/lib/containers/storage/overlay-containers/2135a176f1b5f75668836c3a4e374cd1bcfc17545414ba5fb41be6558078f470/userdata/restore.log: OCI runtime error



$ cat /var/lib/containers/storage/overlay-containers/2135a176f1b5f75668836c3a4e374cd1bcfc17545414ba5fb41be6558078f470/userdata/restore.log
···········
(00.003217) mnt:   [/tmp/.criu.mntns.EnYddk/13-0000000000/sys/firmware](135->205)
(00.003219) mnt:   <--
(00.003220) mnt:  <--
(00.003221) mnt: <--
(00.003247) No pidns-9.img image
(00.003303) Warn  (criu/cr-restore.c:1279): Set CLONE_PARENT | CLONE_NEWPID but it might cause restore problem,because not all kernels support such clone flags combinations!
(00.003305) Forking task with 1 pid (flags 0x6c028000)
(00.003307) Creating process using clone3()
(00.003728) PID: real 5040 virt 1
(00.003986) Wait until namespaces are created
(00.004052) Error (criu/cr-restore.c:1757): Pid 5039 do not match expected 1
(00.005736) Error (criu/cr-restore.c:2447): Restoring FAILED.
```
解决方法：
查看是否有cgroup2
```bash
[root@localhost ~]# mount|grep cgroup
tmpfs on /sys/fs/cgroup type tmpfs (ro,nosuid,nodev,noexec,seclabel,mode=755)
cgroup on /sys/fs/cgroup/systemd type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,xattr,release_agent=/usr/lib/systemd/systemd-cgroups-agent,name=systemd)
cgroup on /sys/fs/cgroup/perf_event type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,perf_event)
cgroup on /sys/fs/cgroup/freezer type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,freezer)
cgroup on /sys/fs/cgroup/devices type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,devices)
cgroup on /sys/fs/cgroup/rdma type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,rdma)
cgroup on /sys/fs/cgroup/pids type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,pids)
cgroup on /sys/fs/cgroup/cpuset type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,cpuset)
cgroup on /sys/fs/cgroup/cpu,cpuacct type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,cpu,cpuacct)
cgroup on /sys/fs/cgroup/misc type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,misc)
cgroup on /sys/fs/cgroup/hugetlb type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,hugetlb)
cgroup on /sys/fs/cgroup/blkio type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,blkio)
cgroup on /sys/fs/cgroup/net_cls,net_prio type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,net_cls,net_prio)
cgroup on /sys/fs/cgroup/memory type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,memory)
[root@localhost ~]# mount|grep cgroup2
[root@localhost ~]# 
```
mount命令中显示的这些cgroup的目录，就是v1的样子。下面我们切换一下v2，看看有什么区别。切换方法其实也很简单，就是在重新启动的时候加上一个内核引导参数：

```bash
systemd.unified_cgroup_hierarchy=1
```

这个参数的意思是，打开cgroup的`unified`属性。是的，`unified`的`cgroup`就是`v2`了。我们加上参数重新引导之后看一下状态：

```bash
$ mkdir /mnt/cgroup2
$ mount -t cgroup2 none /mnt/cgroup2
$ grubby --update-kernel=ALL --args=systemd.unified_cgroup_hierarchy=1
$ reboot
```

