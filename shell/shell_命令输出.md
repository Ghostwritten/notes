# shell 命令输出

process.sh 

```bash
#!/usr/bin/env bash
echo "Hello $USER"
echo "Hey i am $USER and will be telling you about the current processes"
echo "Running processes List"
ps
```
执行：

```bash
$ bash process.sh 
Hello root
Hey i am root and will be telling you about the current processes
Running processes List
   PID TTY          TIME CMD
  6898 pts/0    00:00:00 bash
 29530 pts/0    00:00:00 bash
 29531 pts/0    00:00:00 ps
```

