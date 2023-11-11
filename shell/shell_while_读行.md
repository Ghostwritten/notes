# shell while 读行


```bash
$ cat test 
1
22
33 3
```

 - `while-read.sh`

```bash
#!/bin/bash
# while-read: read lines from a file
count=0
while read; do
	printf "%d %s\n" $REPLY
	count=$(expr $count + 1)
done <$1
```
执行：

```bash
$ bash  wile_read.sh test
1 
22 
33 3
count:3
```

