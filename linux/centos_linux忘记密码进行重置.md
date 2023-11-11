## 1.重启系统,进入 recovery 恢复模式

在读秒时候按e键，找到 `linux16 行`，按键盘`End`

末尾添加空格 `rd.break console=tty0`

按 `ctrl + x` 启动

## 2.以可写方式重新挂载 `/sysroot`,并切换到此环境

```c
switch_root# mount  -o  remount,rw  /sysroot

switch_root# chroot  /sysroot

sh-3.2#
```

## 3. 将root用户的密码设置为 redhat

```c
# echo  redhat  |  passwd  --stdin  root

)重设SELinux安全标签(安全增强版Linux)

# touch  /.autorelabel            //让  SElinux  失忆

# exit
switch_root# reboot 完成修复
```
