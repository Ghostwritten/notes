



-----
##  1.	检查当前kdump服务状态
### 1.1 检查是否已经配置加载kdump环境
检查命令：

```bash
grep crashkernel /proc/cmdline 
```

结果确认：
如果能看到`crashkernel=auto`字样，表示已经加载

 - 备注1：如果当前没有加载运行，以下操作都将无效（因为kdump服务必须要在操作系统启动前先加载1个kdump的内核环境）。
 - 备注2：auto也可能是具体的内存大小，如128M或512M等。

###  1.2 检查kdump服务是否正在运行
检查命令：

```bash
systemctl status kdump 
```

结果确认：
确认为运行状态active (running)


###  1.3	检查当前系统内存使用情况
备注：请特别关注已用内存，
检查命令：

```bash
free -h 
```

结果确认：
记录：Mem行的total和used列

###  1.4	检查kdump存储目录空间情况
kdump.conf 配置文件里的coredump存储目录，确认目录位置和目录的空间（或目录所在的挂载点文件系统可用空间）
检查命令：

```bash
cat /etc/kdump.conf 
```

结果确认：
确认这2行已经开启

```bash
path /var/crash  
core_collector makedumpfile -l --message-level 1 -d 31
```

备注1：默认`coredump`位置是 `/var/crash/`，这个目录可以根据实际情况修改crash目录空间必须要大于步骤2检查的内存used使用【极限情况下，crash目录应该大于`memory+swap`的总量，比如主机内存和swap完全耗尽的场景，通常不需要这么大，但是至少是比已用内存的2倍为好】，如果crash目录容量不足，请修改到有足够容量的目录（或挂载点），重启kdump服务


###  1.5 修改sysconfig kdump参数

```bash
vi /etc/sysconfig/kdump
# 将下面这一行注释掉，然后复制一行，去掉里面的reset_devices配置
#KDUMP_COMMANDLINE_APPEND="irqpoll maxcpus=1 nr_cpus=1 reset_devices cgroup_disable=memory mce=off acpi_no_memhotplug"
修改后：
KDUMP_COMMANDLINE_APPEND="irqpoll maxcpus=1 nr_cpus=1 cgroup_disable=memory mce=off acpi_no_memhotplug"
```

###  1.6 触发coredump动作
手工触发coredump动作，开始收集coredump
执行命令：

```bash
echo 1 > /proc/sys/kernel/sysrq 
echo c > /proc/sysrq-trigger
```

备注：根据内存使用量和存储空间写入速度不同，`coredump`时间不同，无准确时间，coredump收集完成后，主机会自动重启。


##  2. Kdump结果验证
1.	收集coredump文件
正确结果：
如果crash目录下的127.0.0.1-时间戳的子目录，并且目录下有vmcore文件，则表示收集成功。
失败结果：
如果crash目录下没有vmcore文件，则表示coredump收集失败。
2.	发送vmcore文件
将crash目录下生成的127.0.0.1-时间戳的子目录下下的所有文件下载后发给原厂分析（该目录可能很大，与内存使用量有关，几百兆到几十G都可能）


##  3. Kdump失败回退
kdump属于故障信息单次收集操作，没有失败回退。

