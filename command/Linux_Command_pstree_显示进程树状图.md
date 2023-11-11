#  Linux Command pstree 显示进程树状图
tags: 监控



-----
##  1. 简介
pstree命令 以树状图的方式展现进程之间的派生关系，显示效果比较直观。

## 2. 参数

```bash
-a：显示每个程序的完整指令，包含路径，参数或是常驻服务的标示；
-c：不使用精简标示法；
-G：使用VT100终端机的列绘图字符；
-h：列出树状图时，特别标明现在执行的程序；
-H<程序识别码>：此参数的效果和指定"-h"参数类似，但特别标明指定的程序；
-l：采用长列格式显示树状图；
-n：用程序识别码排序。预设是以程序名称来排序；
-p：显示程序识别码；
-u：显示用户名称；
-U：使用UTF-8列绘图字符；
-V：显示版本信息。
-t: 显示线程
```
##  3. 示例
### 3.1 显示当前所有进程的进程号和进程id

```bash
pstree -p
```

### 3.2 显示所有进程的所有详细信息
遇到相同的进程名可以压缩显示。

```bash
pstree  -a
```




### 3.3 特别表明在运行的进程

```bash
# pstree -apnh //显示进程间的关系
systemd,1 auto automatic-ubiquity noprompt
  ├─systemd-journal,471
  ├─lvmetad,488 -f
  ├─systemd-udevd,492
  ├─systemd-timesyn,675
  │   └─{systemd-timesyn},702
  ├─VGAuthService,745
  ├─vmtoolsd,746
  │   ├─{vmtoolsd},762
  │   └─{vmtoolsd},775
  ├─systemd-network,826
  ├─systemd-resolve,838
  ├─rsyslogd,902 -n
  │   ├─{rsyslogd},910
  │   ├─{rsyslogd},911
  │   └─{rsyslogd},912

```
### 3.4 查看某一个进程的树状图
```bash
# -t表示显示线程，-a表示显示命令行参数
$ pstree -t -a -p 27458
mysqld,27458 --log_bin=on --sync_binlog=1
...
  ├─{mysqld},27922
  ├─{mysqld},27923
  └─{mysqld},28014
```
### 3.5 获取 SSH 会话的 PID

```bash
pstree -p | grep ssh

#  |-sshd(1221)-+-sshd(2768)---bash(2770)-+-grep(2810)
#  |            `-sshd(2807)---sshd(2808)
```

从上方的输出中，你可以看到 sshd 进程与分支的树形图。sshd 的主进程是 sshd（1221），另两个分支分别为 sshd（2768） 和 sshd（2807）。
