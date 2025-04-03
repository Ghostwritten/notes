报错内容：

```bash
rpmdb: PANIC: fatal region error detected; run recovery
error: db3 error(-30974) from dbenv->open: DB_RUNRECOVERY: Fatal error, run database recovery
error: cannot open Packages index using db3 - (-30974)
error: cannot open Packages database in /var/lib/rpm
CRITICAL:yum.main:

Error: rpmdb open failed
```


修复方法：

备份
```bash
mkdir /root/backups.rpm.mm_dd_yyyy/
cp -avr /var/lib/rpm/ /root/backups.rpm.mm_dd_yyyy/
```
修复

```bash
rm -f /var/lib/rpm/__db*
db_verify /var/lib/rpm/Packages
rpm --rebuilddb
yum clean all
```

