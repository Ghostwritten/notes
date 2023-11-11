

## 1. 问题
![在这里插入图片描述](https://img-blog.csdnimg.cn/52bc1783ad8b4253a95f7cedda93e5c7.png)


```bash
XFS (dm-0): Internal error XFS_WAIT_CORRUPTED at line 1600 of file fs/xfs/libxfs/xfs_alloc.c. Caller xfs_free_extent+0xf9/0x130 [xfs]
XFS (dm-0): Failed to recover EFIs
```

```bash
...
Mounting /sysroot...
[  ***] A start job is running for /sysroot (3min 59s / 4min 31s)[240.527013] INFO: task mount:406 blocked for more than 120 seconds.
[  240.527056] "echo 0 > /proc/sys/kernel/hung_task_timeout+secs" disables this message."
[FAILED] Failed to mount /sysroot.
See 'systemctl status sysroot.mount' for more details.
[DEPEND] Dependency failed for Initrd Root File System.
[DEPEND] Dependency failed for Reload Configration from the Real Root.
[  OK  ] Stopped dracut pre-pivot and cleanup hook.
[  OK  ] Stopped target Initrd Default Target.
[  OK  ] Reached target Initrd File System.
[  OK  ] Stopped dracut mount hook.
[  OK  ] Stopped target Basic System.
[  OK  ] Stopped System Initialization.
         Starting Emergency Shell...

Genrating "/run/initramfs/rdsosreport.txt"

Entering emergancy mode. Exit the shell to continue.
Type "journalctl" to view system logs.
You might want to save "/run/initramfs/rdsosreport.txt" to usb stick or /boot
after mounting them and attach it to a bug report.

:/#
```


## 2. 解决方法

```bash
/sbin/lvm vgscan
/sbin/lvm vgchange -a y
lvm lvscan
xfs_repair -v -L /dev/dm-0
reboot
```

