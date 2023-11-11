#  Shell 磁盘空间
tags: 监控



---
## 1. Shell 磁盘空间使用率

 - `disk-space.sh`

```bash
MAX=95
EMAIL=server@127.0.0.1
PART=sda1

USE=$(df -h | grep $PART | awk '{ print $5 }' | cut -d'%' -f1)
USE=`printf "%.0f\n" $USE`
if [ $USE -gt $MAX ]; then
   echo "Percent used: $USE" | mail -s "Running out of disk space" $EMAIL
else
   echo "all is well"
fi
```
执行:

```bash
$ bash disk-space.sh
all is well
```
更多阅读：

 - [fio测试磁盘io工具](https://blog.csdn.net/xixihahalelehehe/article/details/118679214)
 - [磁盘 I/O 性能优化的几个思路](https://blog.csdn.net/xixihahalelehehe/article/details/118964708)
 - [为什么我的磁盘I/O延迟很高？](https://blog.csdn.net/xixihahalelehehe/article/details/118702233)

