#  shell 上移目录

 - `up.sh`

```bash
#!/bin/bash

LEVEL=$1
for ((i = 0; i < LEVEL; i++)); do
	echo $i
	CDIR=../$CDIR
done
cd $CDIR
echo "You are in: "$PWD
sh=$(which $SHELL)
exec $sh
```
执行：

```bash
root@yourdomain:~/shell/shell-docs/scripts# pwd
/root/shell/shell-docs/scripts
root@yourdomain:~/shell/shell-docs/scripts# bash up.sh 
You are in: /root
root@yourdomain:~#

root@yourdomain:~/shell/shell-docs/scripts# bash up.sh 1
0
You are in: /root/shell/shell-docs

root@yourdomain:~/shell/shell-docs/scripts# bash up.sh 2
0
1
You are in: /root/shell
```

