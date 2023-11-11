#  Shell 报告服务器信息
tags: 监控

 - `server-health.sh`

```bash
#!/bin/bash
date
echo "uptime:"
uptime
echo "Currently connected:"
w
echo "--------------------"
echo "Last logins:"
last -a | head -3
echo "--------------------"
echo "Disk and memory usage:"
df -h | xargs | awk '{print "Free/total disk: " $11 " / " $9}'
free -m | xargs | awk '{print "Free/total memory: " $17 " / " $8 " MB"}'
echo "--------------------"
start_log=$(head -1 /var/log/messages | cut -c 1-12)
oom=$(grep -ci kill /var/log/messages)
echo -n "OOM errors since $start_log :" $oom
echo ""
echo "--------------------"
echo "Utilization and most expensive processes:"
top -b | head -3
echo
top -b | head -10 | tail -4
echo "--------------------"
echo "Open TCP ports:"
nmap -p -T4 127.0.0.1
echo "--------------------"
echo "Current connections:"
ss -s
echo "--------------------"
echo "processes:"
ps auxf --width=200
echo "--------------------"
echo "vmstat:"
vmstat 1 5
```
执行：

```bash
$  bash server-health.sh
Tue Jun  7 16:12:17 UTC 2022
uptime:
 16:12:17 up 1 day, 20:24,  2 users,  load average: 1.61, 1.61, 1.63
Currently connected:
 16:12:17 up 1 day, 20:24,  2 users,  load average: 1.61, 1.61, 1.63
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
root     tty1     -                Mon15   24:26m  0.66s  0.03s -bash
root     pts/0    192.168.211.1    02:54    1.00s  1.23s  0.01s w
--------------------
Last logins:
root     pts/0        Tue Jun  7 02:54   still logged in    192.168.211.1
root     pts/4        Mon Jun  6 15:45 - 20:15  (04:29)     192.168.211.1
root     tty1         Mon Jun  6 15:45   still logged in
--------------------
Disk and memory usage:
Free/total disk: 3.6G / 3.6G
Free/total memory: 0 / 7411 MB
--------------------
OOM errors since Jun  5 11:55 : 0
--------------------
Utilization and most expensive processes:
top - 16:12:17 up 1 day, 20:24,  2 users,  load average: 1.61, 1.61, 1.63
Tasks: 295 total,   2 running, 195 sleeping,   0 stopped,   0 zombie
%Cpu(s): 20.0 us,  8.7 sy,  0.0 ni, 70.9 id,  0.0 wa,  0.0 hi,  0.5 si,  0.0 st

   PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
 71469 998       20   0  152880  69196  12508 R 100.0  0.9   0:04.19 bundle
  3160 998       20   0 1438356 892316  25176 S  17.6 11.8 474:58.71 bundle
     8 root      20   0       0      0      0 I  11.8  0.0  29:12.26 rcu_sched
--------------------
Open TCP ports:
server-health.sh: line 26: nmap: command not found
--------------------
Current connections:
Total: 1543 (kernel 6994)
TCP:   29 (estab 2, closed 17, orphaned 0, synrecv 0, timewait 0/0), ports 0

Transport Total     IP        IPv6
*         6994      -         -        
RAW       1         0         1        
UDP       3         2         1        
TCP       12        11        1        
INET      16        13        3        
FRAG      0         0         0        

--------------------
processes:
USER        PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root          2  0.0  0.0      0     0 ?        S    Jun05   0:00 [kthreadd]
root          4  0.0  0.0      0     0 ?        I<   Jun05   0:00  \_ [kworker/0:0H]
root          6  0.0  0.0      0     0 ?        I<   Jun05   0:00  \_ [mm_percpu_wq]
root          7  0.1  0.0      0     0 ?        S    Jun05   3:19  \_ [ksoftirqd/0]
root          8  1.0  0.0      0     0 ?        I    Jun05  29:12  \_ [rcu_sched]
root          9  0.0  0.0      0     0 ?        I    Jun05   0:00  \_ [rcu_bh]
root         10  0.0  0.0      0     0 ?        S    Jun05   0:00  \_ [migration/0]
root         11  0.0  0.0      0     0 ?        S    Jun05   0:00  \_ [watchdog/0]
root         12  0.0  0.0      0     0 ?        S    Jun05   0:00  \_ [cpuhp/0]
root         13  0.0  0.0      0     0 ?        S    Jun05   0:00  \_ [cpuhp/1]
root         14  0.0  0.0      0     0 ?        S    Jun05   0:00  \_ [watchdog/1]
root         15  0.0  0.0      0     0 ?        S    Jun05   0:00  \_ [migration/1]
root         16  0.1  0.0      0     0 ?        S    Jun05   4:01  \_ [ksoftirqd/1]
root         18  0.0  0.0      0     0 ?        I<   Jun05   0:00  \_ [kworker/1:0H]
root         19  0.0  0.0      0     0 ?        S    Jun05   0:00  \_ [cpuhp/2]
root         20  0.0  0.0      0     0 ?        S    Jun05   0:01  \_ [watchdog/2]
root         21  0.0  0.0      0     0 ?        S    Jun05   0:00  \_ [migration/2]
root         22  0.1  0.0      0     0 ?        S    Jun05   3:40  \_ [ksoftirqd/2]
root         24  0.0  0.0      0     0 ?        I<   Jun05   0:00  \_ [kworker/2:0H]
root         25  0.0  0.0      0     0 ?        S    Jun05   0:00  \_ [cpuhp/3]
root         26  0.0  0.0      0     0 ?        S    Jun05   0:00  \_ [watchdog/3]
root         27  0.0  0.0      0     0 ?        S    Jun05   0:00  \_ [migration/3]
root         28  0.1  0.0      0     0 ?        S    Jun05   3:30  \_ [ksoftirqd/3]
root         30  0.0  0.0      0     0 ?        I<   Jun05   0:00  \_ [kworker/3:0H]
root         31  0.0  0.0      0     0 ?        S    Jun05   0:00  \_ [kdevtmpfs]
root         32  0.0  0.0      0     0 ?        I<   Jun05   0:00  \_ [netns]
root         33  0.0  0.0      0     0 ?        S    Jun05   0:00  \_ [rcu_tasks_kthre]
root         34  0.0  0.0      0     0 ?        S    Jun05   0:00  \_ [kauditd]
root         36  0.0  0.0      0     0 ?        S    Jun05   0:00  \_ [khungtaskd]
root         37  0.0  0.0      0     0 ?        S    Jun05   0:00  \_ [oom_reaper]
root         38  0.0  0.0      0     0 ?        I<   Jun05   0:00  \_ [writeback]
root         39  0.0  0.0      0     0 ?        S    Jun05   0:00  \_ [kcompactd0]
root         40  0.0  0.0      0     0 ?        SN   Jun05   0:00  \_ [ksmd]
root         41  0.0  0.0      0     0 ?        SN   Jun05   0:00  \_ [khugepaged]
root         42  0.0  0.0      0     0 ?        I<   Jun05   0:00  \_ [crypto]
root         43  0.0  0.0      0     0 ?        I<   Jun05   0:00  \_ [kintegrityd]
root         44  0.0  0.0      0     0 ?        I<   Jun05   0:00  \_ [kblockd]
root         45  0.0  0.0      0     0 ?        I    Jun05   1:17  \_ [kworker/2:1]
root         46  0.0  0.0      0     0 ?        I<   Jun05   0:00  \_ [ata_sff]
root         47  0.0  0.0      0     0 ?        I<   Jun05   0:00  \_ [md]
root         48  0.0  0.0      0     0 ?        I<   Jun05   0:00  \_ [edac-poller]
root         49  0.0  0.0      0     0 ?        I<   Jun05   0:00  \_ [devfreq_wq]
root         50  0.0  0.0      0     0 ?        I<   Jun05   0:00  \_ [watchdogd]
root         55  0.0  0.0      0     0 ?        S    Jun05   0:00  \_ [kswapd0]
root         56  0.0  0.0      0     0 ?        I<   Jun05   0:00  \_ [kworker/u257:0]
root         57  0.0  0.0      0     0 ?        S    Jun05   0:00  \_ [ecryptfs-kthrea]
root         99  0.0  0.0      0     0 ?        I<   Jun05   0:00  \_ [kthrotld]
root        100  0.0  0.0      0     0 ?        I<   Jun05   0:00  \_ [acpi_thermal_pm]
root        101  0.0  0.0      0     0 ?        S    Jun05   0:00  \_ [scsi_eh_0]
root        102  0.0  0.0      0     0 ?        I<   Jun05   0:00  \_ [scsi_tmf_0]
root        103  0.0  0.0      0     0 ?        S    Jun05   0:00  \_ [scsi_eh_1]
root        104  0.0  0.0      0     0 ?        I<   Jun05   0:00  \_ [scsi_tmf_1]
root        110  0.0  0.0      0     0 ?        I<   Jun05   0:00  \_ [ipv6_addrconf]
root        119  0.0  0.0      0     0 ?        I<   Jun05   0:00  \_ [kstrp]
root        136  0.0  0.0      0     0 ?        I<   Jun05   0:00  \_ [charger_manager]
root        203  0.0  0.0      0     0 ?        I<   Jun05   0:00  \_ [mpt_poll_0]
root        205  0.0  0.0      0     0 ?        I<   Jun05   0:00  \_ [mpt/0]
root        207  0.0  0.0      0     0 ?        S    Jun05   0:00  \_ [scsi_eh_2]
root        208  0.0  0.0      0     0 ?        I<   Jun05   0:00  \_ [scsi_tmf_2]
root        209  0.0  0.0      0     0 ?        I<   Jun05   0:00  \_ [ttm_swap]
root        210  0.0  0.0      0     0 ?        S    Jun05   0:00  \_ [irq/16-vmwgfx]
root        211  0.0  0.0      0     0 ?        S    Jun05   0:00  \_ [scsi_eh_3]
root        212  0.0  0.0      0     0 ?        I<   Jun05   0:00  \_ [scsi_tmf_3]
root        213  0.0  0.0      0     0 ?        S    Jun05   0:00  \_ [scsi_eh_4]
root        214  0.0  0.0      0     0 ?        I<   Jun05   0:00  \_ [scsi_tmf_4]
root        215  0.0  0.0      0     0 ?        S    Jun05   0:00  \_ [scsi_eh_5]
root        216  0.0  0.0      0     0 ?        I<   Jun05   0:00  \_ [scsi_tmf_5]
root        217  0.0  0.0      0     0 ?        S    Jun05   0:00  \_ [scsi_eh_6]
root        218  0.0  0.0      0     0 ?        I<   Jun05   0:00  \_ [scsi_tmf_6]
root        219  0.0  0.0      0     0 ?        S    Jun05   0:00  \_ [scsi_eh_7]
root        220  0.0  0.0      0     0 ?        I<   Jun05   0:00  \_ [scsi_tmf_7]
root        221  0.0  0.0      0     0 ?        S    Jun05   0:00  \_ [scsi_eh_8]
root        222  0.0  0.0      0     0 ?        I<   Jun05   0:00  \_ [scsi_tmf_8]
root        223  0.0  0.0      0     0 ?        S    Jun05   0:00  \_ [scsi_eh_9]
root        224  0.0  0.0      0     0 ?        I<   Jun05   0:00  \_ [scsi_tmf_9]
root        225  0.0  0.0      0     0 ?        S    Jun05   0:00  \_ [scsi_eh_10]
root        226  0.0  0.0      0     0 ?        I<   Jun05   0:00  \_ [scsi_tmf_10]
root        227  0.0  0.0      0     0 ?        S    Jun05   0:00  \_ [scsi_eh_11]
root        228  0.0  0.0      0     0 ?        I<   Jun05   0:00  \_ [scsi_tmf_11]
root        229  0.0  0.0      0     0 ?        S    Jun05   0:00  \_ [scsi_eh_12]
root        230  0.0  0.0      0     0 ?        I<   Jun05   0:00  \_ [scsi_tmf_12]
root        231  0.0  0.0      0     0 ?        S    Jun05   0:00  \_ [scsi_eh_13]
root        232  0.0  0.0      0     0 ?        I<   Jun05   0:00  \_ [scsi_tmf_13]
root        233  0.0  0.0      0     0 ?        S    Jun05   0:00  \_ [scsi_eh_14]
root        234  0.0  0.0      0     0 ?        I<   Jun05   0:00  \_ [scsi_tmf_14]
root        235  0.0  0.0      0     0 ?        S    Jun05   0:00  \_ [scsi_eh_15]
root        236  0.0  0.0      0     0 ?        I<   Jun05   0:00  \_ [scsi_tmf_15]
root        237  0.0  0.0      0     0 ?        S    Jun05   0:00  \_ [scsi_eh_16]
root        238  0.0  0.0      0     0 ?        I<   Jun05   0:00  \_ [scsi_tmf_16]
root        239  0.0  0.0      0     0 ?        S    Jun05   0:00  \_ [scsi_eh_17]
root        240  0.0  0.0      0     0 ?        I<   Jun05   0:00  \_ [scsi_tmf_17]
root        241  0.0  0.0      0     0 ?        S    Jun05   0:00  \_ [scsi_eh_18]
root        242  0.0  0.0      0     0 ?        I<   Jun05   0:00  \_ [scsi_tmf_18]
root        243  0.0  0.0      0     0 ?        S    Jun05   0:00  \_ [scsi_eh_19]
root        244  0.0  0.0      0     0 ?        I<   Jun05   0:00  \_ [scsi_tmf_19]
root        245  0.0  0.0      0     0 ?        S    Jun05   0:00  \_ [scsi_eh_20]
root        246  0.0  0.0      0     0 ?        I<   Jun05   0:00  \_ [scsi_tmf_20]
root        247  0.0  0.0      0     0 ?        S    Jun05   0:00  \_ [scsi_eh_21]
root        248  0.0  0.0      0     0 ?        I<   Jun05   0:00  \_ [scsi_tmf_21]
root        249  0.0  0.0      0     0 ?        S    Jun05   0:00  \_ [scsi_eh_22]
root        250  0.0  0.0      0     0 ?        I<   Jun05   0:00  \_ [scsi_tmf_22]
root        251  0.0  0.0      0     0 ?        S    Jun05   0:00  \_ [scsi_eh_23]
root        252  0.0  0.0      0     0 ?        I<   Jun05   0:00  \_ [scsi_tmf_23]
root        253  0.0  0.0      0     0 ?        S    Jun05   0:00  \_ [scsi_eh_24]
root        254  0.0  0.0      0     0 ?        I<   Jun05   0:00  \_ [scsi_tmf_24]
root        255  0.0  0.0      0     0 ?        S    Jun05   0:00  \_ [scsi_eh_25]
root        256  0.0  0.0      0     0 ?        I<   Jun05   0:00  \_ [scsi_tmf_25]
root        257  0.0  0.0      0     0 ?        S    Jun05   0:00  \_ [scsi_eh_26]
root        258  0.0  0.0      0     0 ?        I<   Jun05   0:00  \_ [scsi_tmf_26]
root        259  0.0  0.0      0     0 ?        S    Jun05   0:00  \_ [scsi_eh_27]
root        260  0.0  0.0      0     0 ?        I<   Jun05   0:00  \_ [scsi_tmf_27]
root        261  0.0  0.0      0     0 ?        S    Jun05   0:00  \_ [scsi_eh_28]
root        262  0.0  0.0      0     0 ?        I<   Jun05   0:00  \_ [scsi_tmf_28]
root        263  0.0  0.0      0     0 ?        S    Jun05   0:00  \_ [scsi_eh_29]
root        264  0.0  0.0      0     0 ?        I<   Jun05   0:00  \_ [scsi_tmf_29]
root        265  0.0  0.0      0     0 ?        S    Jun05   0:00  \_ [scsi_eh_30]
root        266  0.0  0.0      0     0 ?        I<   Jun05   0:00  \_ [scsi_tmf_30]
root        267  0.0  0.0      0     0 ?        S    Jun05   0:00  \_ [scsi_eh_31]
root        268  0.0  0.0      0     0 ?        I<   Jun05   0:00  \_ [scsi_tmf_31]
root        269  0.0  0.0      0     0 ?        S    Jun05   0:00  \_ [scsi_eh_32]
root        270  0.0  0.0      0     0 ?        I<   Jun05   0:00  \_ [scsi_tmf_32]
root        303  0.0  0.0      0     0 ?        I<   Jun05   0:01  \_ [kworker/2:1H]
root        304  0.0  0.0      0     0 ?        I<   Jun05   0:01  \_ [kworker/3:1H]
root        307  0.0  0.0      0     0 ?        I<   Jun05   0:02  \_ [kworker/0:1H]
root        312  0.0  0.0      0     0 ?        I<   Jun05   0:02  \_ [kworker/1:1H]
root        314  0.0  0.0      0     0 ?        I<   Jun05   0:00  \_ [kdmflush]
root        315  0.0  0.0      0     0 ?        I<   Jun05   0:00  \_ [bioset]
root        321  0.0  0.0      0     0 ?        I<   Jun05   0:00  \_ [kdmflush]
root        323  0.0  0.0      0     0 ?        I<   Jun05   0:00  \_ [bioset]
root        396  0.0  0.0      0     0 ?        I<   Jun05   0:00  \_ [raid5wq]
root        443  0.0  0.0      0     0 ?        S    Jun05   0:34  \_ [jbd2/dm-1-8]
root        444  0.0  0.0      0     0 ?        I<   Jun05   0:00  \_ [ext4-rsv-conver]
root        527  0.0  0.0      0     0 ?        I<   Jun05   0:00  \_ [iscsi_eh]
root        537  0.0  0.0      0     0 ?        I<   Jun05   0:00  \_ [ib-comp-wq]
root        538  0.0  0.0      0     0 ?        I<   Jun05   0:00  \_ [ib-comp-unb-wq]
root        539  0.0  0.0      0     0 ?        I<   Jun05   0:00  \_ [ib_mcast]
root        540  0.0  0.0      0     0 ?        I<   Jun05   0:00  \_ [ib_nl_sa_wq]
root        543  0.0  0.0      0     0 ?        I<   Jun05   0:00  \_ [rdma_cm]
root        599  0.0  0.0      0     0 ?        I<   Jun05   0:00  \_ [kworker/u257:2]
root        759  0.0  0.0      0     0 ?        I<   Jun05   0:00  \_ [xfsalloc]
root        760  0.0  0.0      0     0 ?        I<   Jun05   0:00  \_ [xfs_mru_cache]
root        766  0.0  0.0      0     0 ?        I<   Jun05   0:00  \_ [xfs-buf/dm-0]
root        768  0.0  0.0      0     0 ?        I<   Jun05   0:00  \_ [xfs-data/dm-0]
root        769  0.0  0.0      0     0 ?        I<   Jun05   0:00  \_ [xfs-conv/dm-0]
root        770  0.0  0.0      0     0 ?        I<   Jun05   0:00  \_ [xfs-cil/dm-0]
root        771  0.0  0.0      0     0 ?        I<   Jun05   0:00  \_ [xfs-reclaim/dm-]
root        777  0.0  0.0      0     0 ?        I<   Jun05   0:00  \_ [xfs-log/dm-0]
root        778  0.0  0.0      0     0 ?        I<   Jun05   0:00  \_ [xfs-eofblocks/d]
root        779  0.0  0.0      0     0 ?        S    Jun05   1:12  \_ [xfsaild/dm-0]
root        785  0.0  0.0      0     0 ?        S    Jun05   0:00  \_ [jbd2/sda2-8]
root        786  0.0  0.0      0     0 ?        I<   Jun05   0:00  \_ [ext4-rsv-conver]
root      21953  0.0  0.0      0     0 ?        I    Jun06   0:55  \_ [kworker/1:0]
root      42411  0.0  0.0      0     0 ?        I    00:11   0:36  \_ [kworker/3:1]
root      59695  0.0  0.0      0     0 ?        I    00:40   0:00  \_ [kworker/3:0]
root      54014  0.0  0.0      0     0 ?        I    07:58   0:00  \_ [kworker/1:2]
root      59626  0.0  0.0      0     0 ?        I    12:05   0:02  \_ [kworker/2:2]
root      51576  0.0  0.0      0     0 ?        I    15:37   0:01  \_ [kworker/u256:2]
root      59643  0.0  0.0      0     0 ?        I    15:51   0:00  \_ [kworker/u256:0]
root      63577  0.0  0.0      0     0 ?        I    15:58   0:00  \_ [kworker/0:0]
root      66848  0.0  0.0      0     0 ?        I    16:04   0:00  \_ [kworker/0:2]
root      68181  0.0  0.0      0     0 ?        I    16:06   0:00  \_ [kworker/u256:1]
root      70068  0.0  0.0      0     0 ?        I    16:09   0:00  \_ [kworker/0:1]
root          1  0.0  0.1 160052  9116 ?        Ss   Jun05   0:21 /sbin/init auto automatic-ubiquity noprompt
root        517  0.0  0.1 100540 13000 ?        S<s  Jun05   0:28 /lib/systemd/systemd-journald
root        544  0.0  0.0 171444  1848 ?        Ss   Jun05   0:00 /sbin/lvmetad -f
root        547  0.0  0.0  46512  5008 ?        Ss   Jun05   0:08 /lib/systemd/systemd-udevd
systemd+    826  0.0  0.0 141780  3008 ?        Ssl  Jun05   0:01 /lib/systemd/systemd-timesyncd
root        894  0.0  0.1  89864  9972 ?        Ss   Jun05   0:00 /usr/bin/VGAuthService
root        896  0.1  0.0 143284  7156 ?        S<sl Jun05   4:00 /usr/bin/vmtoolsd
systemd+    959  0.0  0.0  71712  5264 ?        Ss   Jun05   0:02 /lib/systemd/systemd-networkd
systemd+    979  0.0  0.0  70604  6032 ?        Ss   Jun05   0:01 /lib/systemd/systemd-resolved
root       1076  0.0  0.0  70444  5820 ?        Ss   Jun05   0:02 /lib/systemd/systemd-logind
root       1079  0.0  0.0  30032  3168 ?        Ss   Jun05   0:01 /usr/sbin/cron -f
syslog     1080  0.0  0.0 263044  5384 ?        Ssl  Jun05   0:38 /usr/sbin/rsyslogd -n
daemon     1084  0.0  0.0  28336  2384 ?        Ss   Jun05   0:00 /usr/sbin/atd -f
root       1153  0.0  0.1 434328  9592 ?        Ssl  Jun05   0:00 /usr/sbin/ModemManager --filter-policy=strict
message+   1156  0.0  0.0  50408  4704 ?        Rs   Jun05   0:06 /usr/bin/dbus-daemon --system --address=systemd: --nofork --nopidfile --systemd-activation --syslog-only
root       1189  0.0  0.0  45240  5484 ?        Ss   Jun05   0:00 /sbin/wpa_supplicant -u -s -O /run/wpa_supplicant
root       1190  0.0  0.2 479636 16192 ?        Ssl  Jun05   0:03 /usr/sbin/NetworkManager --no-daemon
root       1192  0.0  0.0 286248  6860 ?        Ssl  Jun05   0:02 /usr/lib/accountsservice/accounts-daemon
root       1196  0.1  0.9 22585756 72012 ?      Ssl  Jun05   3:30 /root/fastgithub_linux-x64/fastgithub
root       2051  0.0  0.3 712704 24416 ?        Sl   Jun05   1:02  \_ dnscrypt-proxy/dnscrypt-proxy
root       1226  0.0  0.0 110556  2012 ?        Ssl  Jun05   0:09 /usr/sbin/irqbalance --foreground
root       1228  0.0  0.2 169476 17680 ?        Ssl  Jun05   0:00 /usr/bin/python3 /usr/bin/networkd-dispatcher --run-startup-triggers
root       1230  0.0  0.0 531724  2796 ?        Ssl  Jun05   0:02 /usr/bin/lxcfs /var/lib/lxcfs/
root       1293  0.0  0.0 288884  6496 ?        Ssl  Jun05   0:00 /usr/lib/policykit-1/polkitd --no-debug
root       1407  0.0  0.2 185836 20160 ?        Ssl  Jun05   0:00 /usr/bin/python3 /usr/share/unattended-upgrades/unattended-upgrade-shutdown --wait-for-signal
root       1426  0.0  0.6 1499372 45932 ?       Ssl  Jun05   2:18 /usr/bin/containerd
root       1528  0.0  0.0  78872  3792 tty1     Ss   Jun05   0:00 /bin/login -p --
root      51612  0.0  0.0  21500  5088 tty1     S+   Jun06   0:00  \_ -bash
root       1553  0.0  0.0  72304  5600 ?        Ss   Jun05   0:00 /usr/sbin/sshd -D
root       6756  0.0  0.0 105692  7180 ?        Ss   02:54   0:10  \_ sshd: root@pts/0
root       6898  0.0  0.0  21632  5356 pts/0    Ss   02:54   0:00      \_ -bash
root      90226  0.0  0.0  20392  4000 pts/0    S    05:19   0:00          \_ /bin/bash
root      91413  0.0  0.0  20388  4052 pts/0    S    05:21   0:00              \_ /bin/bash
root      91673  0.0  0.0  20388  3912 pts/0    S    05:21   0:00                  \_ /bin/bash
root     107993  0.0  0.0  20388  3992 pts/0    S    05:50   0:00                      \_ /bin/bash
root     108274  0.0  0.0  20388  3944 pts/0    S    05:50   0:00                          \_ /bin/bash
root     108487  0.0  0.0  20388  3996 pts/0    S    05:50   0:00                              \_ /bin/bash
root     108726  0.0  0.0  20396  4052 pts/0    S    05:51   0:00                                  \_ /bin/bash
root     108934  0.0  0.0  20388  3968 pts/0    S    05:51   0:00                                      \_ /bin/bash
root     109714  0.0  0.0  20388  4040 pts/0    S    05:53   0:00                                          \_ /bin/bash
root      71517  2.0  0.0  11600  3260 pts/0    S+   16:12   0:00                                              \_ bash server-health.sh
root      71542  0.0  0.0  38792  3956 pts/0    R+   16:12   0:00                                                  \_ ps auxf --width=200
root       1753  0.6  1.0 1644264 83180 ?       Ssl  Jun05  17:52 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
root       2272  0.0  0.0 1226900 4060 ?        Sl   Jun05   0:00  \_ /usr/bin/docker-proxy -proto tcp -host-ip 0.0.0.0 -host-port 8081 -container-ip 172.17.0.2 -container-port 8081
root       2286  0.0  0.0 1300888 3748 ?        Sl   Jun05   0:00  \_ /usr/bin/docker-proxy -proto tcp -host-ip 0.0.0.0 -host-port 2222 -container-ip 172.17.0.2 -container-port 22
root       2437  0.0  0.0 1152912 3964 ?        Sl   Jun05   0:00  \_ /usr/bin/docker-proxy -proto tcp -host-ip 0.0.0.0 -host-port 27017 -container-ip 172.18.0.2 -container-port 27017
root       2310  0.5  0.1 712852 10512 ?        Sl   Jun05  13:41 /usr/bin/containerd-shim-runc-v2 -namespace moby -id e855e89ccaa42d74c6d074a034ff6092e09b53ca2846875652e70349f0842ff8 -address /run/co
root       2357  0.0  0.0   5856  3416 ?        Ss   Jun05   0:00  \_ /bin/bash /assets/wrapper
root       2737  0.0  0.0   2528  1336 ?        S    Jun05   0:02      \_ runsvdir -P /opt/gitlab/service log: .........................................................................................
root       2746  0.0  0.0   2376   708 ?        Ss   Jun05   0:00      |   \_ runsv sshd
root       2747  0.0  0.0   2520   708 ?        S    Jun05   0:00      |   |   \_ svlogd -tt /var/log/gitlab/sshd
root       2748  0.0  0.0  12180  6800 ?        S    Jun05   0:00      |   |   \_ sshd: /usr/sbin/sshd -D -f /assets/sshd_config -e [listener] 0 of 100-200 startups
root       3032  0.0  0.0   2376  1300 ?        Ss   Jun05   0:00      |   \_ runsv logrotate
root       3038  0.0  0.0   2520   640 ?        S    Jun05   0:00      |   |   \_ svlogd -tt /var/log/gitlab/logrotate
root      57801  0.0  0.0   2612  1576 ?        Ss   15:48   0:00      |   |   \_ /bin/sh /opt/gitlab/embedded/bin/gitlab-logrotate-wrapper
root      63532  0.0  0.0   2512   588 ?        S    15:58   0:00      |   |       \_ sleep 3000
root       3033  0.0  0.0   2376   644 ?        Ss   Jun05   0:00      |   \_ runsv redis
root       3080  0.0  0.0   2520   640 ?        S    Jun05   0:32      |   |   \_ svlogd -tt /var/log/gitlab/redis
997        3085  2.0  0.1  66804  8964 ?        Rsl  Jun05  53:53      |   |   \_ /opt/gitlab/embedded/bin/redis-server 127.0.0.1:0
root       3034  0.0  0.0   2376   708 ?        Ss   Jun05   0:00      |   \_ runsv gitaly
root       3051  0.0  0.0   2520   704 ?        S    Jun05   0:02      |   |   \_ svlogd /var/log/gitlab/gitaly
998        3053  0.0  0.1 1395004 9096 ?        Ssl  Jun05   0:32      |   |   \_ /opt/gitlab/embedded/bin/gitaly-wrapper /opt/gitlab/embedded/bin/gitaly /var/opt/gitlab/gitaly/config.toml
998        3117  0.2  1.5 1645376 118628 ?      Sl   Jun05   5:47      |   |       \_ /opt/gitlab/embedded/bin/gitaly /var/opt/gitlab/gitaly/config.toml
998        3190  0.2  1.8 2295408 138420 ?      Sl   Jun05   6:45      |   |           \_ ruby /opt/gitlab/embedded/service/gitaly-ruby/bin/gitaly-ruby 363 /var/opt/gitlab/gitaly/internal_sockets/ruby
998        3193  0.2  1.8 2296468 139760 ?      Sl   Jun05   6:44      |   |           \_ ruby /opt/gitlab/embedded/service/gitaly-ruby/bin/gitaly-ruby 363 /var/opt/gitlab/gitaly/internal_sockets/ruby
root       3035  0.0  0.0   2376   708 ?        Ss   Jun05   0:00      |   \_ runsv postgresql
root       3079  0.0  0.0   2520   712 ?        S    Jun05   0:03      |   |   \_ svlogd -tt /var/log/gitlab/postgresql
996        3083  0.0  0.2  34920 19928 ?        Ss   Jun05   0:06      |   |   \_ /opt/gitlab/embedded/bin/postgres -D /var/opt/gitlab/postgresql/data
996        3141  0.0  0.0  35020  5376 ?        Ss   Jun05   0:00      |   |       \_ postgres: checkpointer   
996        3142  0.0  0.0  34920  5236 ?        Ss   Jun05   0:33      |   |       \_ postgres: background writer   
996        3143  0.0  0.0  34920  3780 ?        Ss   Jun05   0:02      |   |       \_ postgres: walwriter   
996        3144  0.1  0.1  35612  8220 ?        Ss   Jun05   4:05      |   |       \_ postgres: autovacuum launcher   
996        3145  0.0  0.0  17728  4460 ?        Ss   Jun05   0:46      |   |       \_ postgres: stats collector   
996        3146  0.0  0.0  35456  6144 ?        Ss   Jun05   0:00      |   |       \_ postgres: logical replication launcher   
996       41229  0.0  0.2  41044 19144 ?        Ss   15:18   0:00      |   |       \_ postgres: gitlab gitlabhq_production [local] idle
996       41960  0.0  0.2  41024 19180 ?        Ss   15:19   0:00      |   |       \_ postgres: gitlab gitlabhq_production [local] idle
996       55915  0.0  0.1  37824 13088 ?        Ss   15:45   0:00      |   |       \_ postgres: gitlab gitlabhq_production [local] idle
996       57956  0.0  0.2  40300 17532 ?        Ss   15:48   0:00      |   |       \_ postgres: gitlab gitlabhq_production [local] idle
996       66439  0.0  0.2  41044 18680 ?        Ss   16:03   0:00      |   |       \_ postgres: gitlab gitlabhq_production [local] idle
996       71493  0.5  0.1  37568  9520 ?        Ss   16:12   0:00      |   |       \_ postgres: autovacuum worker   gitlabhq_production
root       3036  0.0  0.0   2376  1268 ?        Ss   Jun05   0:01      |   \_ runsv puma
root       3054  0.0  0.0   2520   712 ?        S    Jun05   0:17      |   |   \_ svlogd -tt /var/log/gitlab/puma
998       71469  105  0.9 152880 69456 ?        Rs   16:12   0:04      |   |   \_ puma 5.3.2 (unix:///var/opt/gitlab/gitlab-rails/sockets/gitlab.socket,tcp://127.0.0.1:8080) [gitlab-puma-worker]
root       3037  0.0  0.0   2376   708 ?        Ss   Jun05   0:00      |   \_ runsv sidekiq
root       3048  0.0  0.0   2520   704 ?        S    Jun05   1:47      |   |   \_ svlogd /var/log/gitlab/sidekiq
998        3049  0.0  0.4 111308 31144 ?        Ssl  Jun05   0:04      |   |   \_ ruby /opt/gitlab/embedded/service/gitlab-rails/bin/sidekiq-cluster -e production -r /opt/gitlab/embedded/service/gitla
998        3160 17.8 11.7 1438356 892316 ?      Sl   Jun05 474:58      |   |       \_ sidekiq 6.2.2 queues:authorized_project_update:authorized_project_update_project_create,authorized_project_update:
root       3039  0.0  0.0   2376   704 ?        Ss   Jun05   0:00      |   \_ runsv gitlab-workhorse
root       3076  0.0  0.0   2520   640 ?        S    Jun05   0:03      |   |   \_ svlogd /var/log/gitlab/gitlab-workhorse
998        3077  0.0  0.4 1497836 36344 ?       Ssl  Jun05   1:19      |   |   \_ /opt/gitlab/embedded/bin/gitlab-workhorse -listenNetwork unix -listenUmask 0 -listenAddr /var/opt/gitlab/gitlab-workho
root       3040  0.0  0.0   2376   708 ?        Ss   Jun05   0:00      |   \_ runsv nginx
root       3078  0.0  0.0   2520   636 ?        S    Jun05   0:00      |   |   \_ svlogd -tt /var/log/gitlab/nginx
root       3082  0.0  0.0  20116  6468 ?        Ss   Jun05   0:00      |   |   \_ nginx: master process /opt/gitlab/embedded/sbin/nginx -p /var/opt/gitlab/nginx
999        3103  0.0  0.0  24368  6788 ?        S    Jun05   0:00      |   |       \_ nginx: worker process
999        3104  0.0  0.0  24368  6788 ?        S    Jun05   0:20      |   |       \_ nginx: worker process
999        3105  0.0  0.0  24368  6788 ?        S    Jun05   0:00      |   |       \_ nginx: worker process
999        3106  0.0  0.0  24368  6788 ?        S    Jun05   0:00      |   |       \_ nginx: worker process
999        3107  0.0  0.0  20332  2556 ?        S    Jun05   0:00      |   |       \_ nginx: cache manager process
root       3041  0.0  0.0   2376   708 ?        Ss   Jun05   0:00      |   \_ runsv gitlab-exporter
root       3068  0.0  0.0   2520   704 ?        S    Jun05   0:00      |   |   \_ svlogd -tt /var/log/gitlab/gitlab-exporter
998        3069  0.0  0.5 125068 45208 ?        Ss   Jun05   0:02      |   |   \_ /opt/gitlab/embedded/bin/ruby /opt/gitlab/embedded/bin/gitlab-exporter web -c /var/opt/gitlab/gitlab-exporter/gitlab-e
root       3043  0.0  0.0   2376   708 ?        Ss   Jun05   0:00      |   \_ runsv redis-exporter
root       3050  0.0  0.0   2520   704 ?        S    Jun05   0:00      |   |   \_ svlogd -tt /var/log/gitlab/redis-exporter
997        3052  0.0  0.1 1093012 9476 ?        Ssl  Jun05   0:00      |   |   \_ /opt/gitlab/embedded/bin/redis_exporter --web.listen-address=localhost:9121 --redis.addr=unix:///var/opt/gitlab/redis/
root       3044  0.0  0.0   2376  1352 ?        Ss   Jun05   2:28      |   \_ runsv prometheus
root       3067  0.2  0.0   2520   640 ?        S    Jun05   6:34      |   |   \_ svlogd -tt /var/log/gitlab/prometheus
root       3045  0.0  0.0   2376   708 ?        Ss   Jun05   0:00      |   \_ runsv alertmanager
root       3081  0.0  0.0   2520   696 ?        S    Jun05   0:00      |   |   \_ svlogd -tt /var/log/gitlab/alertmanager
992        3084  0.1  0.3 1409236 27868 ?       Ssl  Jun05   4:26      |   |   \_ /opt/gitlab/embedded/bin/alertmanager --web.listen-address=localhost:9093 --storage.path=/var/opt/gitlab/alertmanager/
root       3046  0.0  0.0   2376   640 ?        Ss   Jun05   0:00      |   \_ runsv postgres-exporter
root       3074  0.0  0.0   2520   640 ?        S    Jun05   0:00      |   |   \_ svlogd -tt /var/log/gitlab/postgres-exporter
996        3075  0.0  0.1 1092976 9648 ?        Ssl  Jun05   0:00      |   |   \_ /opt/gitlab/embedded/bin/postgres_exporter --web.listen-address=localhost:9187 --extend.query-path=/var/opt/gitlab/pos
root       3047  0.0  0.0   2376   712 ?        Ss   Jun05   0:00      |   \_ runsv grafana
root       3058  0.0  0.0   2520   708 ?        S    Jun05   0:00      |       \_ svlogd -tt /var/log/gitlab/grafana
992        3059  0.0  0.7 1600600 58096 ?       Ssl  Jun05   1:19      |       \_ /opt/gitlab/embedded/bin/grafana-server -config /var/opt/gitlab/grafana/grafana.ini
root       3177  0.0  0.0   5732  3332 ?        S    Jun05   0:00      \_ /bin/bash /opt/gitlab/bin/gitlab-ctl tail
root       3178  0.0  0.5  95576 41628 ?        S    Jun05   0:03      |   \_ /opt/gitlab/embedded/bin/ruby /opt/gitlab/embedded/bin/omnibus-ctl gitlab /opt/gitlab/embedded/service/omnibus-ctl* tail
root       3181  0.0  0.0   2612   604 ?        S    Jun05   0:00      |       \_ sh -c find -L /var/log/gitlab -type f -not -path '*/sasl/*' | grep -E -v '(config|lock|@|gzip|tgz|gz)' | xargs tail --
root       3184  0.0  0.0   4564   584 ?        S    Jun05   0:00      |           \_ xargs tail --follow=name --retry
root       3185  0.2  0.0   4300   584 ?        S    Jun05   6:13      |               \_ tail --follow=name --retry /var/log/gitlab/nginx/gitlab_error.log /var/log/gitlab/nginx/access.log /var/log/gi
998        5215  0.1 10.2 1134168 778140 ?      Sl   Jun05   4:26      \_ puma: cluster worker 0: 1412 [gitlab-puma-worker]
998        5217  0.1 10.2 1152600 778932 ?      Sl   Jun05   4:43      \_ puma: cluster worker 1: 1412 [gitlab-puma-worker]
998        5219  0.1 10.2 1142360 780604 ?      Sl   Jun05   4:26      \_ puma: cluster worker 2: 1412 [gitlab-puma-worker]
998        5221  0.1 10.2 1152600 779260 ?      Sl   Jun05   4:41      \_ puma: cluster worker 3: 1412 [gitlab-puma-worker]
root       2311  0.0  0.1 713108  9108 ?        Sl   Jun05   0:34 /usr/bin/containerd-shim-runc-v2 -namespace moby -id c9a392e472fdd83fa02c8c3ffd9b9d11078872d7e7a198dcb1e5a0fa6a2d9e3e -address /run/co
root       2365  0.0  0.0    212     4 ?        Ss   Jun05   0:00  \_ /usr/bin/dumb-init /entrypoint run --user=gitlab-runner --working-directory=/home/gitlab-runner
root       2400  0.1  0.4 152420 33012 ?        Ssl  Jun05   4:28      \_ gitlab-runner run --user=gitlab-runner --working-directory=/home/gitlab-runner
root       2456  0.0  0.1 711700  9408 ?        Sl   Jun05   0:28 /usr/bin/containerd-shim-runc-v2 -namespace moby -id 8c51a6f7265cafd6ec1db0f86086aa086f5493b1f913bbe0b634814e09cd0772 -address /run/co
999        2483  0.6  1.4 1546508 111420 ?      Ssl  Jun05  17:11  \_ mongod --bind_ip_all
root       3373  0.0  0.1  76760  7736 ?        Ss   Jun05   0:10 /lib/systemd/systemd --user
root       3380  0.0  0.0 191368  2552 ?        S    Jun05   0:00  \_ (sd-pam)
root     116287  0.0  0.0  92032  3448 ?        SLs  05:59   0:00  \_ /usr/bin/gpg-agent --supervised
--------------------
vmstat:
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 7  0      0 911992 386560 2464324    0    0     4    15   19   36 20  9 71  0  0
 3  0      0 908272 386560 2464336    0    0     0     0 1876 3107 16 14 71  0  0
 2  0      0 901708 386560 2464356    0    0     0     0 1479 2539 14 15 71  0  0
 2  0      0 888604 386560 2464356    0    0     0    80 1892 3200 15 15 70  0  0
 2  0      0 885012 386560 2464376    0    0     0   150 2005 4046 13 17 70  0  0
```

