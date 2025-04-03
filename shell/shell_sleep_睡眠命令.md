#  shell sleep 睡眠
![](https://i-blog.csdnimg.cn/blog_migrate/def4dab4f04aa7154287df08c842c093.png)




##  1. 背景
当用户在 Linux 中发出多个命令序列时，命令会立即一个接一个或同时执行（例如，tee 命令）。但是，有时需要推迟命令的执行并为系统提供足够的时间来产生预期的结果。

##  2. 简介
sleep命令将下一个命令的调用进程挂起一段指定的时间。当以下命令的执行取决于前一个命令的成功完成时，此属性很有用。

## 3. 语法

```bash
sleep [number]
```

默认情况下，系统读取之后sleep的数字作为秒数。要指定其他时间单位，请使用以下语法：

```bash
sleep [number][unit]
```

```bash
 sleep 1h 2m 0.5s
```

该`sleep`命令接受浮点数。它允许多个值相加，以计算sleep.

可用单位有：

 - `s`– 秒
 - `m`- 分钟
 - `h`- 小时
 - `d`- 天

要`sleep`在开始后和指定的等待时间结束前停止，请按`Ctrl + C`。

要查看该sleep命令的帮助，请键入：

```bash
sleep --help 
```

有关版本详细信息，请键入：

```bash
sleep --version
```

## 4. 与 wait 区别
[bash wait](https://blog.csdn.net/xixihahalelehehe/article/details/127348688) 命令是一个Shell 命令，它等待后台运行的进程完成并返回退出状态。与等待指定时间的 sleep 命令不同，`wait` 命令等待所有或特定的后台任务完成。

## 5. 实例
### 5.1 设置警报
用于`sleep`告诉系统在一定时间后播放 `mp3` 文件。

```bash
sleep 7h 30m && mplayer alarm.mp3
```
### 5.2  终端中的延迟命令
`sleep`对于强制执行两个命令之间的时间很有用,以一秒的间隔执行：

```bash
$ sleep 1 && echo "one" && sleep 1 && echo "two"
one
two
```

### 5.3 变量分配给 sleep 
可以将变量分配给`sleep`命令。
```bash
#!/bin/bash
SLEEP_INTERVAL="30"
CURRENT_TIME=$(date +"%T")
echo "Time before sleep: ${CURRENT_TIME}"
echo "Sleeping for ${SLEEP_INTERVAL} seconds"
sleep ${SLEEP_INTERVAL}
CURRENT_TIME=$(date +"%T")
echo "Time after sleep: ${CURRENT_TIME}"
```
该脚本定义了一个名为的变量`SLEEP_INTERVAL` ，其值稍后用作sleep命令的参数。此示例脚本的输出显示执行持续了 30 秒：

```bash
$ ./time_script.sh
Time before sleep: 00:01:15
Sleeping for 30 seconds
Time after sleep: 00:01:45
```

### 5.4 定义检查间隔
检查网站是否在线，如果成功 ping 一个网站，脚本就会停止，在不成功的 ping 之间引入 10 秒的延迟。


```bash
#!/bin/bash
while :
  do
    if ping -c 1 www.google.com &> /dev/null
    then
   echo "Google is online"
   break
   fi
 sleep 10
done
```

### 5.5 为操作完成留出时间
您可能正在运行一个 bash 脚本，该脚本在内部调用另外两个 bash 脚本——一个在后台运行测试，另一个打印结果。如果第二个脚本在第一个脚本完成之前执行，用于`sleep`防止第二个脚本打印错误的结果：

```bash
while kill -0 $BACK_PID ; do
    echo "Waiting for the process to end"
    sleep 1
done
```
该`kill -0 $BACK_PID`命令检查第一个脚本的进程是否仍在运行。如果是，它会打印消息并休眠 1 秒钟，然后再次检查。

### 5.6 预测延迟
用于sleep允许某些命令执行的延迟。

```bash
for (( i = 1 ; i <= 250 ; i++ )); 
    do  
    sleep 1
    <do something>
done
```



参考：
- [How to Use the Linux sleep Command with Examples](https://phoenixnap.com/kb/linux-sleep)
