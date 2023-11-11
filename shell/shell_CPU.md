#  shell CPU
tags: 监控

## 1.  CPU 使用率
- `CPU.sh`
```bash
MAX=95
EMAIL=server@127.0.0.1

USE=$(grep 'cpu ' /proc/stat | awk '{usage=($2+$4)*100/($2+$4+$5)} END {print usage ""}')
USE=`printf "%.0f\n" $USE`
if [[ $USE -gt $MAX ]]; then
   echo "Percent used: $USE" | mail -s "Running out of CPU power" $EMAIL
else
   echo "all is well !"
fi
```
执行：

```bash
$ bash CPU.sh
all is well !
```

##  2. CPU 检查

```bash
#!/bin/bash
#################################################################################
# ActiveXperts Network Monitor - Shell script checks
#
# For more information about ActiveXperts Network Monitor and SSH, please
# visit the online ActiveXperts Network Monitor Shell Script Guidelines at:
#   https://www.activexperts.com/support/network-monitor/online/linux/
#################################################################################
# Script
#     cpu.sh
# Description
#     Checks CPU usage on the computer.
# Declare Parameters
#     1) nMaxCpuUsage (number) - maximum allowed CPU usage (%)
# Usage
#     cpu.sh nMaxCpuUsage
# Sample
#     cpu.sh 70
#################################################################################


nMaxCpuUsage=$1

# Validate number of arguments
if [ $# -ne 1 ] ; then
  echo "UNCERTAIN: Invalid number of arguments - Usage: cpu nMaxCpuUsage"
  exit 1
fi

# Validate numeric parameter nMaxCpuUsage
regExpNumber='^[0-9]+$'
if ! [[ $1 =~ $regExpNumber ]] ; then
  echo "UNCERTAIN: Invalid argument: nMaxCpuUsage (number expected)"
  exit 1
fi

# Check the CPU usage
nCpuLoadPercentage=`ps -A -o pcpu | tail -n+2 | paste -sd+ | bc`
nCpuLoadPercentage=$( echo "$nCpuLoadPercentage / 1" | bc )

if [ $nCpuLoadPercentage -le $nMaxCpuUsage ] ; then
  echo "SUCCESS: CPU usage is [$nCpuLoadPercentage%], minimum allowed=[$1%] DATA:$nCpuLoadPercentage"
else
  echo "ERROR: CPU usage is [$nCpuLoadPercentage%], minimum allowed=[$1%] DATA:$nCpuLoadPercentage"
fi
 
exit 0

```
执行：

```bash
$ bash nmaxcpu.sh 70
SUCCESS: CPU usage is [0%], minimum allowed=[70%] DATA:0
$ bash nmaxcpu.sh 80
SUCCESS: CPU usage is [0%], minimum allowed=[80%] DATA:0
```

更多阅读：

 - [CPU使用率达到100%怎么办？](https://blog.csdn.net/xixihahalelehehe/article/details/117926926)
 - [系统中的软中断使用率升高怎么办？](https://blog.csdn.net/xixihahalelehehe/article/details/117991825)
 - [如何快速分析出CPU的瓶颈？](https://blog.csdn.net/xixihahalelehehe/article/details/118178337)
 - [CPU使用率高却找不到应用？](https://blog.csdn.net/xixihahalelehehe/article/details/117961015)

