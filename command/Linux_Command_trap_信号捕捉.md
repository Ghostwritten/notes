#  Linux Command trap 信号捕捉



---

##  1. 简介
信号捕捉 trap，它用于捕获指定的信号并执行预定义的命令。比如，按`Ctrl+C`会使脚本终止执行，实际上系统发送了SIGINT信号给脚本进程，SIGINT信号的默认处理方式就是退出程序。如果要在`Ctrl +C`不退出程序，那么就得使用trap命令来指定一下SIGINT的处理方式了。trap命令不仅仅处理Linux信号，还能对脚本退出（EXIT）、调试（DEBUG）、错误（ERR）、返回（RETURN）等情况指定处理方式。

##  2. 语法
它用于捕获指定的信号并执行预定义的命令。

```bash
trap [-lp] [[arg] sigspec ...]
```

 - `arg`：是shell命令或者自定义函数或者脚本
 - `sigspec`：可以是信号名或者数值。信号名的大小写不敏感，SIG这个前缀也是可选的。以下的命令效果一样

实例：

```bash
trap "echo trap int" 2
trap "echo trap int" int 
trap "echo trap int" Int
trap "echo trap int" INT 
trap "echo trap int" SIGINT
```
也可以同时写多个信号量

```bash
trap 'echo "Press ctrl+c"' 2 3
```

##  3. 信号介绍
通过`trap -l`查看类似kill -l的输出查看一共有多少信号量

```bash
$ trap -l
 1) SIGHUP       2) SIGINT       3) SIGQUIT      4) SIGILL       5) SIGTRAP
 6) SIGABRT      7) SIGBUS       8) SIGFPE       9) SIGKILL     10) SIGUSR1
11) SIGSEGV     12) SIGUSR2     13) SIGPIPE     14) SIGALRM     15) SIGTERM
16) SIGSTKFLT   17) SIGCHLD     18) SIGCONT     19) SIGSTOP     20) SIGTSTP
21) SIGTTIN     22) SIGTTOU     23) SIGURG      24) SIGXCPU     25) SIGXFSZ
26) SIGVTALRM   27) SIGPROF     28) SIGWINCH    29) SIGIO       30) SIGPWR
31) SIGSYS      34) SIGRTMIN    35) SIGRTMIN+1  36) SIGRTMIN+2  37) SIGRTMIN+3
38) SIGRTMIN+4  39) SIGRTMIN+5  40) SIGRTMIN+6  41) SIGRTMIN+7  42) SIGRTMIN+8
43) SIGRTMIN+9  44) SIGRTMIN+10 45) SIGRTMIN+11 46) SIGRTMIN+12 47) SIGRTMIN+13
48) SIGRTMIN+14 49) SIGRTMIN+15 50) SIGRTMAX-14 51) SIGRTMAX-13 52) SIGRTMAX-12
53) SIGRTMAX-11 54) SIGRTMAX-10 55) SIGRTMAX-9  56) SIGRTMAX-8  57) SIGRTMAX-7
58) SIGRTMAX-6  59) SIGRTMAX-5  60) SIGRTMAX-4  61) SIGRTMAX-3  62) SIGRTMAX-2
63) SIGRTMAX-1  64) SIGRTMAX
```

trap -l 等价于执行 kill -l，发送信号请查看 kill 命令。

```bash
1) SIGHUP 本信号在用户终端连接(正常或非正常)结束时发出, 通常是在终端的控制进程结束时, 通知同一session内的各个作业, 这时它们与控制终端不再关联. 　　
2) SIGINT 程序终止(interrupt)信号, 在用户键入INTR字符(通常是Ctrl-C)时发出 　　
3) SIGQUIT 和SIGINT类似, 但由QUIT字符(通常是Ctrl-/)来控制. 进程在因收到SIGQUIT退出时会产生core文件, 在这个意义上类似于一个程序错误信号. 　　
4) SIGILL 执行了非法指令. 通常是因为可执行文件本身出现错误, 或者试图执行数据段. 堆栈溢出时也有可能产生这个信号. 　　
5) SIGTRAP 由断点指令或其它trap指令产生. 由debugger使用. 　　
6) SIGABRT 程序自己发现错误并调用abort时产生. 　　
7) SIGIOT 在PDP-11上由iot指令产生, 在其它机器上和SIGABRT一样. 　　
8) SIGBUS 非法地址, 包括内存地址对齐(alignment)出错. eg: 访问一个四个字长的整数, 但其地址不是4的倍数. 　　
9) SIGFPE 在发生致命的算术运算错误时发出. 不仅包括浮点运算错误, 还包括溢出及除数为0等其它所有的算术的错误. 　　
10) SIGKILL 用来立即结束程序的运行. 本信号不能被阻塞, 处理和忽略. 　　
11) SIGUSR1 留给用户使用 　　
12) SIGSEGV 试图访问未分配给自己的内存, 或试图往没有写权限的内存地址写数据. 　　
13) SIGUSR2 留给用户使用 　　
14) SIGPIPE Broken pipe 　　
15) SIGALRM 时钟定时信号, 计算的是实际的时间或时钟时间. alarm函数使用该信号. 　　
16) SIGTERM 程序结束(terminate)信号, 与SIGKILL不同的是该信号可以被阻塞和处理. 通常用来要求程序自己正常退出. shell命令kill缺省产生这个信号. 　　
17) SIGCHLD 子进程结束时, 父进程会收到这个信号. 　　
18) SIGCONT 让一个停止(stopped)的进程继续执行. 本信号不能被阻塞. 可以用一个handler来让程序在由stopped状态变为继续执行时完成特定的工作. 例如, 重新显示提示符 　　
19) SIGSTOP 停止(stopped)进程的执行. 注l意它和terminate以及interrupt的区别: 该进程还未结束, 只是暂停执行. 本信号不能被阻塞, 处理或忽略. 　　
20) SIGTSTP 停止进程的运行, 但该信号可以被处理和忽略. 用户键入SUSP字符时(通常是Ctrl-Z)发出这个信号 　　
21) SIGTTIN 当后台作业要从用户终端读数据时, 该作业中的所有进程会收到SIGTTIN信号. 缺省时这些进程会停止执行. 　　
22) SIGTTOU 类似于SIGTTIN, 但在写终端(或修改终端模式)时收到. 　　
23) SIGURG 有紧急数据或out-of-band数据到达socket时产生. 　　
24) SIGXCPU 超过CPU时间资源限制. 这个限制可以由getrlimit/setrlimit来读取/改变 　　
25) SIGXFSZ 超过文件大小资源限制. 　　
26) SIGVTALRM 虚拟时钟信号. 类似于SIGALRM, 但是计算的是该进程占用的CPU时间. 　　
27) SIGPROF 类似于SIGALRM/SIGVTALRM, 但包括该进程用的CPU时间以及系统调用的时间. 　　
28) SIGWINCH 窗口大小改变时发出. 　　
29) SIGIO 文件描述符准备就绪, 可以开始进行输入/输出操作. 　　
30) SIGPWR Power failure
```
## 4. 常用信号

 - `Ctrl + c`: 程序终止信号,也可以指定为2或者int
 - `EXIT`：在shell退出前执行trap设置的命令，也可以指定为0
 - `RETURN`：在函数返回时，或者`.`和`source`执行其他脚本返回时，执行trap设置的命令
 - `DEBUG`：在任何命令执行前执行trap设置的命令，但对于函数仅在函数的第一条命令前执行一次


##  5. 命令

```bash
$ trap "脚本或命令" signal-list
当脚本收到signal-list清单内列出的信号时,trap命令执行双引号中的命令，而不会执行原操作

$ trap signal-list
如果没有指定命令部分，那么就将信号处理复原。比如 trap INT 就表明恢复Ctrl+C退出

$ trap "" signal-list
忽略信号signals，可以多个，比如 trap "" INT 表明忽略SIGINT信号，按Ctrl+C也不能使脚本或者命令退出。

$ trap "-" signal-list
恢复原信号的操作

$ trap -p
当前的trap设置打印出来

$ trap “commands” EXIT
脚本退出时执行的命令

$ trap : 2
恢复信号
```

##  6. 实例

###  6.1 ctrl + c 
`Ctrl + c`: 程序终止信号,也可以指定为2或者int

```bash
$ trap "echo hello" 2
$ ^Chello

$ trap "echo apple" 2
$
$ ^Capple

$ trap "echo help" int
$ ^Chelp

$ trap -p
trap -- 'echo trap 2' SIGINT
trap -- '' SIGTSTP
trap -- '' SIGTTIN
trap -- '' SIGTTOU
```

### 6.2 信号屏蔽和恢复

```bash
trap "" 2   #信号屏蔽(忽略)
trap : 2    #恢复信号
trap "-" 2  #恢复信号
```

```bash
$ trap 'echo "Press ctrl+c"' int
$ trap "" int      #信号屏蔽
$ ls    #ctrl + c不会有动作反应
1.sh  2  3  c  dir1  dir3

$ trap : 2   #恢复信号
$ ls^C   #ctrl + c有动作反应
$

```
忽略多个信号

```bash
$ trap '' 1 2 3 15
```


###  6.3 debug
DEBUG：在任何命令执行前执行trap设置的命令，但对于函数仅在函数的第一条命令前执行一次

```bash
#!/bin/bash
hi() {
    echo "hi shuge"
}
trap "echo this is trap" DEBUG
hi
hello() {
    echo "hello shuge"
}
hello
```
执行：
$ bash trap_debug.sh 

```bash
this is trap       # 在函数执行前执行trap中的命令
hi shuge
this is trap       # 在函数执行前执行trap中的命令
hello shuge
```

###  6.4  exit

```bash
#!/bin/bash
# test trap command
trap "echo Goodbye." EXIT
#trap "echo Goodbye." 0 # 与EXIT一个效果

echo This is a test script

count=1
while [ $count -le 10 ]
do
  echo "Loop $count"
  sleep 1
  count=$[ $count + 1 ]
done

echo The end.   
```
执行：

```bash
$ bash trap_exit.sh 
This is a test script
Loop 1
Loop 2
Loop 3
Loop 4
Loop 5
Loop 6
Loop 7
Loop 8
Loop 9
Loop 10
The end.
Goodbye. #脚本退出前执行trap中指定的命令
```


### 6.5  return

```bash
#!/bin/bash
hi() {
    trap "echo this is trap return" return
    echo "hi shuge"
    echo "hi shuge2"
}

hi
hello() {
    trap "echo this is trap return" return
    echo "hello shuge"
}
hello
```
执行：

```bash
$ bash trap_return.sh 
hi shuge
hi shuge2
this is trap return
hello shuge
this is trap return
```

###  6.6  SIGINT

 - trap_ SIGINT.sh

```bash
#!/bin/bash
# test trap command

trap "echo 'Sorry! I have trapped Ctrl-C'" SIGINT

count=1
while [ $count -le 5 ]
do
  echo "Loop $count"
  sleep 1
  count=$[ $count + 1 ]
done


trap "echo 'Sorry! The trap has been modified.'" SIGINT

count=1
while [ $count -le 5 ]
do
  echo "Loop $count"
  sleep 1
  count=$[ $count + 1 ]
done

echo The end.
```
执行：

```bash
$ bash  trap_ SIGINT.sh
Loop 1
Loop 2
Loop 3
^CSorry! I have trapped Ctrl-C
Loop 4
Loop 5
Loop 1
Loop 2
Loop 3
^CSorry! The trap has been modified.
Loop 4
Loop 5
The end.
```

##  7. 综合
###  7.1 信号设置与恢复
 - trap_test_1.sh

```bash
#!/bin/bash

trap "echo 'Sorry!I have trapped Ctrl+C'" 2
echo "This is a test script~"

count=1

while [ $count -le 3 ]; do
    echo "Loop #$count"
    sleep 2
    count=$(($count + 1))
done

echo "This is the end of the script~"
trap - 2 ##恢复
echo "I just removed the trap"
```
执行：

```bash
$ bash trap_test_1.sh
This is a test script~
Loop #1
Loop #2
^CSorry!I have trapped Ctrl+C
Loop #3
This is the end of the script~
I just removed the trap
```

### 7.2 信号屏蔽
 - trap_test_2.sh

```bash
#!/bin/bash
trap 'echo "Press ctrl+c"' int
trap -p
for ((i = 0; i <= 3; i++)); do
    sleep 1
    echo $i
done
trap '' int
trap -p
for ((i = 4; i <= 6; i++)); do
    sleep 1
    echo $i
done
trap '-' int
trap -p
for ((i = 7; i <= 9; i++)); do
    sleep 1
    echo $i
done
```

执行：

```bash
$ bash trap_test_2.sh
trap -- 'echo "Press ctrl+c"' SIGINT
1
^CPress ctrl+c	# 输出信号对应的命令
2
3
trap -- '' SIGINT
4
^C5	#屏蔽信号
6
7
^C	#恢复信号，终端进程
```

###  7.3 条件动作
执行脚本时，脚本完成后，将新建的文件全部删除掉

```bash
#!/bin/bash

trap "find /tmp -type f -name 'trap_test*' | xargs rm -f && exit "  0

new_file(){
    touch /tmp/trap_test_$(date +%F-%N-%M-%S)
    sleep 2
    touch /tmp/trap_test_$(date +%F-%N-%M-%S)
    ls -l /tmp/trap_test*
}

new_file
```
执行：

```bash
$ bash trap_rm.sh
-rw-r--r-- 1 root root 0 Jun 28 10:22 /tmp/trap_test_2022-06-28-899109106-22-51
-rw-r--r-- 1 root root 0 Jun 28 10:22 /tmp/trap_test_2022-06-28-916718446-22-53
```

参考：

 - [The Bash Trap Command](https://www.linuxjournal.com/content/bash-trap-command)
 - [phoenixnap Bash trap Command Explained](https://phoenixnap.com/kb/bash-trap-command)
 - [linuxhint Bash trap command](https://linuxhint.com/bash_trap_command/)
 - [Using Bash traps in your scripts](https://opensource.com/article/20/6/bash-trap)

