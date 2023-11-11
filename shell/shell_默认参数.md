#  shell 默认参数
脚本
```bash
#!/bin/bash
v=${1:-'1.0.0'}
h=${2:-'test demo'}
echo ${v}
echo ${h}
```
输出：

```bash
[root@master ~]# bash test.sh 
1.0.0
test demo
[root@master ~]# bash test.sh  abc 123
abc
123
```

