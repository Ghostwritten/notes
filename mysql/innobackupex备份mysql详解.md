## 介绍
## 参数
--parallel
此参数用于开启多个子进程并发备份多个数据文件（注意，一个数据文件只会有一个进程完成备份）。可以加快备份速度。但是在服务器资源不足时，谨慎使用。

```bash
innobackupex --user=root --password=123456 --parallel=16 /tmp

```
--throttle
此参数用于限制备份过程中每秒的IO次数。

```bash
innobackupex --user=root --password=123456 --throttle=100 /tmp
```

两个参数同时使用

```bash
innobackupex --user=root --password=123456 --parallel=16 --throttle=100 /tmp
```

[XtraBackup 的流式和压缩备份](https://cloud.tencent.com/developer/article/1396304)
