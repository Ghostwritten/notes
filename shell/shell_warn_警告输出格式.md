# shell warn 警告输出格式
## 1. die
在linux shell中执行命令后加上die命令，执行过程中如果出错会报出相应的原因与行号。如`cat /usr/home/test.log || die $?`，如果文件不存在，则会报出相应的错误。

 - test1.sh

```powershell
#!/bin/bash

warn () {
  echo "$@" >&2
}


die () {
  status="$1"
  shift
  warn "$@"
  exit "$status"
}

cat /usr/home/test.log || die $?
[root@dockermake ~]# bash test1.sh 
cat: /usr/home/test.log: No such file or directory
```


