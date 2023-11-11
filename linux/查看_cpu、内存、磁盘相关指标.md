

## 1. top查看cpu最高
```bash
top -b | head -50
top -c -b | head -50
```

```bash
## 参数
-b：批次档模式
head -50：显示输出结果的前 50 个
PID：进程的 ID
USER：进程的归属者
PR：进程的等级
NI：进程的 NICE 值
VIRT：进程使用的虚拟内存
RES：进程使用的物理内存
SHR：进程使用的共享内存
S：这个值表示进程的状态: S = 睡眠，R = 运行，Z = 僵尸进程
%CPU：进程占用的 CPU 比例
%MEM：进程使用的 RAM 比例
TIME+：进程运行了多长时间
COMMAND：进程名字
```
## 2. ps查询cpu最高

```bash
显示命令绝对路径
ps -eo pid,ppid,%mem,%cpu,cmd --sort=-%cpu | head
显示相对路径
ps -eo pid,ppid,%mem,%cpu,comm --sort=-%cpu | head
```

```bash
-e：选择所有进程
-o：自定义输出格式
–sort=-%cpu：基于 CPU 使用率对输出结果排序
head：显示结果的前 10 行
PID：进程的 ID
PPID：父进程的 ID
%MEM：进程使用的 RAM 比例
%CPU：进程占用的 CPU 比例
Command：进程名字
```

##  3. 查看cpu核数

```bash
 总核数 = 物理CPU个数 X 每颗物理CPU的核数 
# 总逻辑CPU数 = 物理CPU个数 X 每颗物理CPU的核数 X 超线程数

# 查看物理CPU个数
cat /proc/cpuinfo| grep "physical id"| sort| uniq| wc -l

# 查看每个物理CPU中core的个数(即核数)
cat /proc/cpuinfo| grep "cpu cores"| uniq

# 查看逻辑CPU的个数
cat /proc/cpuinfo| grep "processor"| wc -l
```

查看CPU信息（型号）
cat /proc/cpuinfo | grep name | cut -f2 -d: | uniq -c

 

## 4. 查看内存信息
如何查看linux 系统内存大小的信息，可以查看总内存，剩余内存，可使用内存等信息  

```bash
# cat /proc/meminfo
```

 

## 5. 查看Linux 内核
uname -a
cat /proc/version

 

## 6. 查看机器型号（机器硬件型号）

dmidecode | grep "Product Name"
dmidecode



## 7. 查看linux 系统版本

```bash
cat /etc/redhat-release
lsb_release -a
cat  /etc/issue
```

 





##  8. top与ps结合查看
top 命令，回车，按下shift+p，按照cpu利用率排序，找到对应的进程，第一列是pid，拿到pid后，执行 ps -ef|grep pid 可以看到是哪个程序在跑，具体执行路径，或者用 lsof |grep pid ，找到有关联的文件，kill -9 pid，rm 删除关联文件，查看crontab是否有被改动。然后更改用户密码，加固防火墙，对端口及ip做限制
