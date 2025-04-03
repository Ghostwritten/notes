![](https://i-blog.csdnimg.cn/blog_migrate/cd33599466765d6feb1e5cdd9fbb4696.png)


添加磁盘 /dev/sdb
```bash
root@registry01 ~]# fdisk -l

Disk /dev/sda: 53.7 GB, 53687091200 bytes, 104857600 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x00020a75

   Device Boot      Start         End      Blocks   Id  System
/dev/sda1   *        2048     2099199     1048576   83  Linux
/dev/sda2         2099200    62914559    30407680   8e  Linux LVM

Disk /dev/sdb: 107.4 GB, 107374182400 bytes, 209715200 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/mapper/centos-root: 27.9 GB, 27913093120 bytes, 54517760 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/mapper/centos-swap: 3221 MB, 3221225472 bytes, 6291456 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes

# 创建 pv
[root@registry01 ~]# pvcreate /dev/sdb
  Physical volume "/dev/sdb" successfully created.

[root@registry01 ~]# vgcreate data_vg /dev/sdb
  Volume group "data_vg" successfully created

#创建挂载目录
[root@registry01 ~]# mkdir /data

[root@registry01 ~]# vgs
  VG      #PV #LV #SN Attr   VSize    VFree   
  centos    1   2   0 wz--n-  <29.00g       0 
  data_vg   1   0   0 wz--n- <100.00g <100.00g

#创建 逻辑卷，名字data_lv
[root@registry01 ~]# lvcreate -L 99.5G -n data_lv data_vg
  Logical volume "data_lv" created.
[root@registry01 ~]# mkfs.xfs /dev/data_vg/data_lv 
meta-data=/dev/data_vg/data_lv   isize=512    agcount=4, agsize=6520832 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0, sparse=0
data     =                       bsize=4096   blocks=26083328, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal log           bsize=4096   blocks=12736, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0

#获取UUID
[root@registry01 ~]# blkid
/dev/sda1: UUID="6243936c-cd94-465f-b204-ae0841e0d212" TYPE="xfs" 
/dev/sda2: UUID="6SOs6z-YAgw-R32V-HXsi-i8hw-f20k-KPSRYo" TYPE="LVM2_member" 
/dev/sdb: UUID="ziwiW3-SpjB-Ex5t-6ujx-oSIS-H3g2-Lc0Yta" TYPE="LVM2_member" 
/dev/mapper/centos-root: UUID="202ea48b-32fe-4750-96af-669583ee7786" TYPE="xfs" 
/dev/mapper/centos-swap: UUID="f9810a4e-572b-4e5f-86bb-fc442fb28884" TYPE="swap" 
/dev/mapper/data_vg-data_lv: UUID="91af294a-f615-4edc-8c30-7a99a11c1a4e" TYPE="xfs" 
```
挂载
```bash
echo "UUID="91af294a-f615-4edc-8c30-7a99a11c1a4e" /data xfs defaults 0 0" >> /etc/fstab
mount -a
df -Th
```
查看
```bash
[root@registry01 ~]# df -Th
Filesystem                  Type      Size  Used Avail Use% Mounted on
devtmpfs                    devtmpfs  3.9G     0  3.9G   0% /dev
tmpfs                       tmpfs     3.9G     0  3.9G   0% /dev/shm
tmpfs                       tmpfs     3.9G  8.7M  3.9G   1% /run
tmpfs                       tmpfs     3.9G     0  3.9G   0% /sys/fs/cgroup
/dev/mapper/centos-root     xfs        26G  2.1G   24G   8% /
/dev/sda1                   xfs      1014M  221M  794M  22% /boot
tmpfs                       tmpfs     795M     0  795M   0% /run/user/0
/dev/mapper/data_vg-data_lv xfs       100G   33M  100G   1% /data
```

参考：

-  [https://access.redhat.com/documentation/zh-cn/red_hat_enterprise_linux/7/html/logical_volume_manager_administration/index](https://access.redhat.com/documentation/zh-cn/red_hat_enterprise_linux/7/html/logical_volume_manager_administration/index)
