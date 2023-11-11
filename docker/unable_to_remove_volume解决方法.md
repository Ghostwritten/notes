##
问题：
```bash
$ docker volume rm 92ba8612_zxtest001_LOG 
Error response from daemon: unable to remove volume: remove 92ba8612_zxtest001_LOG: VolumeDriver.Remove: exec fail:exit status 5;out:  Logical volume data-2-2_HDD_VG/92ba8612_zxtest001_LOG contains a filesystem in use.
```
解决方法：
1.尝试先删除挂载的本地目录 

```bash
$ rm -rf 92ba8612_zxtest001_LOG/
$  docker volume rm 92ba8612_zxtest001_LOG 
92ba8612_zxtest001_LOG
```



2. 如果还是无法删除

```bash
$ lsof |grep 92ba8612_zxtest001_LOG
$ for i in `lsof  |grep 92ba8612_zxtest001_LOG | awk '{print $2}'`;do kill -9 $i;done
```

3. 如果还是无法删除，执行：

```bash
lvremove /dev/data-2-1_HDD_VG/d728336a_wbLoCtovlFE001_DAT   Logical volume data-2-1_HDD_VG/d728336a_wbLoCtovlFE001_DAT contains a filesystem in use.
```
如果还是无法删除：

```bash
查看是否有用户正在使用、打开
$ lvdisplay /dev/data-2-1_HDD_VG/d728336a_wbLoCtovlFE001_DAT |grep open
查看是否有进程号正在使用
$ fuser -kuc /dev/data-2-1_HDD_VG/d728336a_wbLoCtovlFE001_DAT
如果有进程号 ，例如是1726
$ kill -9 1726
删除逻辑卷
$ lvremove /dev/data-2-1_HDD_VG/d728336a_wbLoCtovlFE001_DAT   Logical volume data-2-1_HDD_VG/d728336a_wbLoCtovlFE001_DAT
```

4.如果还是无法删除，

```bash
$ lvchange -an /dev/data-2-1_HDD_VG/d728336a_wbLoCtovlFE001_DAT
$ lvremove -f   /dev/data-2-1_HDD_VG/d728336a_wbLoCtovlFE001_DAT
```
5.如果还是无法删除

```bash
$ echo 1 > /proc/sys/vm/drop_caches
$ echo 2 > /proc/sys/vm/drop_caches
$ echo 3 > /proc/sys/vm/drop_caches
$ docker volume rm d728336a_wbLoCtovlFE001_DAT
```
6. 如果还是无法删除（大部分可以删除）

```bash
$ docker kill `docker ps -q`
$ sysetemctl restart docker
$ docker volume rm d728336a_wbLoCtovlFE001_DAT
$ docker start `docker ps -aq`
```
7.如果还是无法删除，比如僵尸了,重启。

```bash
$ reboot
.....
$ docker volume rm  d728336a_wbLoCtovlFE001_DAT
```

