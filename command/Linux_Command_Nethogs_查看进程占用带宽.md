# Linux Command Nethogs 查看进程占用带宽

[Nethogs](https://github.com/raboof/nethogs) 是一个终端下的网络流量监控工具可以直观的显示每个进程占用的带宽。
下载：http://sourceforge.net/projects/nethogs/files/nethogs/0.8/nethogs-0.8.0.tar.gz/download

```bash
$ yum  -y install libpcap-devel  ncurses-devel 
$ tar zxvf nethogs-0.8.0.tar.gz 
$ cd nethogs 
$ make && make install 
$ nethogs eth0 
```
参考：

 - [Nethogs – Monitor Linux Network Traffic Usage Per Process](https://www.tecmint.com/nethogs-monitor-per-process-network-bandwidth-usage-in-real-time/)
 - [How to Monitor Network Traffic using nethogs](https://linuxhint.com/monitor_network_traffic_nethogs/)
 - [nethogs(8) - Linux man page](https://linux.die.net/man/8/nethogs)
 - [Linux – Monitoring Network Traffic With nethogs](https://www.geeksforgeeks.org/linux-monitoring-network-traffic-with-nethogs/)
 - [NetHogs——Linux下按进程实时统计网络带宽利用率](https://linux.cn/article-2808-1.html)

