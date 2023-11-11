#  shell 显示系统信息菜单
tags: 监控



---

![在这里插入图片描述](https://img-blog.csdnimg.cn/8d1be848585e417a9bb9b5f896ad6ef7.gif#pic_center)


## 1. 一次交互
 - `Read-Menu.sh`

```bash
#!/usr/bin/env bash
# read-menu: a menu driven system information program
clear
cat << EOF
Please Select:
    1. Display System Information
    2. Display Disk Space
    3. Display Home Space Utilization
    0. Quit
EOF
echo -n 'Enter selection [0-3]: '
read -r sel

case $sel in
	0) echo "Program terminated.";;
	1) echo "Hostname: $HOSTNAME"; uptime;;
	2) df -h;;
	3)
		if [ "$UID" = 0 ]; then
			echo "Home Space Utilization (All Users)"
			du -sh /home/*
		else
			echo "Home Space Utilization ($USER)"
			du -sh "$HOME"
		fi
	;;
	*)
		echo "Invalid entry." >&2
		exit 1
esac
```
执行：

```bash
$ bash read-menu.sh
Please Select:
    1. Display System Information
    2. Display Disk Space
    3. Display Home Space Utilization
    0. Quit
Enter selection [0-3]: 0
Program terminated.

$ bash read-menu.sh
Please Select:
    1. Display System Information
    2. Display Disk Space
    3. Display Home Space Utilization
    0. Quit
Enter selection [0-3]: 1
Hostname: yourdomain.com
 04:27:46 up 1 day,  8:40,  2 users,  load average: 1.74, 1.66, 1.64

$ bash read-menu.sh
Please Select:
    1. Display System Information
    2. Display Disk Space
    3. Display Home Space Utilization
    0. Quit
Enter selection [0-3]: 2
Filesystem                         Size  Used Avail Use% Mounted on
udev                               3.6G     0  3.6G   0% /dev
tmpfs                              742M   75M  667M  11% /run
/dev/mapper/ubuntu--vg-ubuntu--lv   19G   19G     0 100% /
tmpfs                              3.7G     0  3.7G   0% /dev/shm
tmpfs                              5.0M     0  5.0M   0% /run/lock
tmpfs                              3.7G     0  3.7G   0% /sys/fs/cgroup
/dev/mapper/data-lvdata             40G   15G   26G  37% /data
/dev/sda2                          976M  220M  690M  25% /boot
overlay                             40G   15G   26G  37% /data/docker/overlay2/e0be8abd3b18e4c43604eb7a21bc4a6cd40d26290dcaf126a7ecc4ce4463803f/merged
overlay                             40G   15G   26G  37% /data/docker/overlay2/263af3bac5540d800f30cb0302129810f7a12b9c0aa075c5bb5ef9c3e404e694/merged
overlay                             40G   15G   26G  37% /data/docker/overlay2/c8c0ac7f5b991aa62b7786dc0adacf00685d03c42556080572936da9053eb89a/merged
tmpfs                              742M     0  742M   0% /run/user/0

$ bash read-menu.sh
Please Select:
    1. Display System Information
    2. Display Disk Space
    3. Display Home Space Utilization
    0. Quit
Enter selection [0-3]: 3
Home Space Utilization (All Users)
32K     /home/ghostwritten


```
##  2. 循环交互

 - `while-menu.sh`

```bash
#!/bin/bash
# while-menu: a menu driven system information program
DELAY=1 # Number of seconds to display results
while true; do
    clear
        cat << EOF
        Please Select:
        1. Display System Information
        2. Display Disk Space
        3. Display Home Space Utilization
        0. Quit
EOF
    read -p "Enter selection [0-3] > "
    case "$REPLY" in
        0)
            break
            ;;
        1)
            echo "Hostname: $HOSTNAME"
            uptime
            ;;
        2)
            df -h
            ;;
        3)
            if [[ $(id -u) -eq 0 ]]; then
                echo "Home Space Utilization (All Users)"
                du -sh /home/*
            else
                echo "Home Space Utilization ($USER)"
                du -sh $HOME
            fi
            ;;
        *)
            echo "Invalid entry."
            ;;
    esac
    sleep "$DELAY"
done
echo "Program terminated."
```
执行：

```bash
$ bash read-menu.sh
Please Select:
    1. Display System Information
    2. Display Disk Space
    3. Display Home Space Utilization
    0. Quit
Enter selection [0-3]: 0
Program terminated.

Please Select:
    1. Display System Information
    2. Display Disk Space
    3. Display Home Space Utilization
    0. Quit
Enter selection [0-3]: 1
Hostname: yourdomain.com
 04:27:46 up 1 day,  8:40,  2 users,  load average: 1.74, 1.66, 1.64

Please Select:
    1. Display System Information
    2. Display Disk Space
    3. Display Home Space Utilization
    0. Quit
Enter selection [0-3]: 2
Filesystem                         Size  Used Avail Use% Mounted on
udev                               3.6G     0  3.6G   0% /dev
tmpfs                              742M   75M  667M  11% /run
/dev/mapper/ubuntu--vg-ubuntu--lv   19G   19G     0 100% /
tmpfs                              3.7G     0  3.7G   0% /dev/shm
tmpfs                              5.0M     0  5.0M   0% /run/lock
tmpfs                              3.7G     0  3.7G   0% /sys/fs/cgroup
/dev/mapper/data-lvdata             40G   15G   26G  37% /data
/dev/sda2                          976M  220M  690M  25% /boot
overlay                             40G   15G   26G  37% /data/docker/overlay2/e0be8abd3b18e4c43604eb7a21bc4a6cd40d26290dcaf126a7ecc4ce4463803f/merged
overlay                             40G   15G   26G  37% /data/docker/overlay2/263af3bac5540d800f30cb0302129810f7a12b9c0aa075c5bb5ef9c3e404e694/merged
overlay                             40G   15G   26G  37% /data/docker/overlay2/c8c0ac7f5b991aa62b7786dc0adacf00685d03c42556080572936da9053eb89a/merged
tmpfs                              742M     0  742M   0% /run/user/0

Please Select:
    1. Display System Information
    2. Display Disk Space
    3. Display Home Space Utilization
    0. Quit
Enter selection [0-3]: 3
Home Space Utilization (All Users)
32K     /home/ghostwritten
```

