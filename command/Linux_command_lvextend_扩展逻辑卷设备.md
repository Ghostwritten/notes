#  Linux command lvextend 扩展逻辑卷设备
tags: 设备

![](https://i-blog.csdnimg.cn/blog_migrate/c5d4bbe83c5f80e5fa746e019d997182.png)



## 1. 简介
lvextend 命令来自于英文词组“`logical volume extend`”的缩写，其功能是用于扩展逻辑卷设备。LVM逻辑卷管理器技术具有灵活调整卷组与逻辑卷的特点，逻辑卷设备容量可以在创建时规定，亦可以后期根据业务需求进行动态扩展或缩小。

## 2. 语法

```bash
lvextend [参数] 逻辑卷
```

## 3. 常用参数

```bash
-L	指定逻辑卷的大小（容量单位）
-l	指定逻辑卷的大小（PE个数）
```

## 4. 安装

```bash
$ sudo apt-get install lvm2

检查 LVM 的版本以验证安装
$ lvm version
```

## 5. 实例
###  5.1 通过设置大小进行扩展
将卷扩展至 290M
```bash
# 将卷扩展至 290M
$ lvextend -L 290M /dev/VolGroup00/LogVol00
Rounding size to boundary between physical extents: 292.00 MiB.
Size of logical volume storage/vo changed from 148 MiB (37 extents) to 292 MiB (73 extents).
Logical volume /dev/VolGroup00/LogVol00 successfully resized.

$ pvs
$ lvs
$ vgs

使用resizefs2命令重新加载逻辑卷的大小才能生效。 
$ resize2fs   /dev/VolGroup00/LogVol00
```
### 5.2 按特定单元扩展

将卷扩展增加 100M

```bash
lvextend -L +100M /dev/VolGroup00/LogVol00
```
![](https://i-blog.csdnimg.cn/blog_migrate/4de82a76fd42d0798fbc76d77d123096.png)
![](https://i-blog.csdnimg.cn/blog_migrate/d67b67e36edafc756b3f9a309458844f.png)
我们的初始大小为 `100Mb`，但我们已将其扩展到 `200Mb`。

### 5.3 百分比扩展


lvextend 还支持指定扩展逻辑卷的百分比。指定的百分比将当前大小扩展为总空间的百分比。例如，让我们扩展 5%。我们目前的大小是`332.00M`。
```bash
 lvextend -l +5%VG /dev/vg01/lv01
```
![](https://i-blog.csdnimg.cn/blog_migrate/9c49dd245e1fe732365f65df063ccd73.png)

### 5.4 使用剩余的可用空间进行扩展
上述方法扩展到总空间的一小部分。但是，此方法会根据可用空间的百分比进行扩展。因此，使用 100% 将扩展并使用所有可用的可用空间。

让我们使用下面的命令扩展 50% 的可用空间。
```bash
lvextend -l +50%FREE /dev/vg01/lv01
lvextend -l +100%FREE /dev/volgroup/logvol
```
![](https://i-blog.csdnimg.cn/blog_migrate/e59baca2dcfe9622e440d32c38b4cdd3.png)
![](https://i-blog.csdnimg.cn/blog_migrate/46706addc7f513aba1386e8c8d0c6311.png)

通过xfs_growfs 才能生效

```bash
xfs_growfs /dev/vg01/lv01
```


参考：
- [lvextend(8) — Linux manual page](https://man7.org/linux/man-pages/man8/lvextend.8.html)
- [The lvextend Linux Command](https://linuxhint.com/lvextend-linux-command/)
