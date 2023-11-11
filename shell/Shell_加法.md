#  Shell 加法



---
##  1. 两数字相加
```bash
#!/usr/bin/env bash

echo -n 'Enter the First Number: '
read -r a
echo -n 'Enter the Second Number: '
read -r b
echo "$a + $b = $((a+b))"
```
执行：

```bash
$ bash addition.sh
Enter the First Number: 1
Enter the Second Number: 2
1 + 2 = 3
```

## 2. while

```bash
#!/bin/bash

read -p "请输入一个数字：" num
NUM=${num:-100}
SUM=0
i=1

while [ $i -le $NUM ]
do
  let SUM+=i
  let i++
done

echo "$SUM"
```


```bash
$  bash sum1.sh 
请输入一个数字：10
55
```






## 3. for

```bash
#!/bin/bash
read -p "请输入一个数字：" num

NUM=${num:-100}
SUM=0
i=1

for i in `seq $NUM`
do
  SUM=$[SUM+i]
  let i++
done

echo "$SUM"
```

```bash
$ bash for1.sh
请输入一个数字：10
55
```




## 4. awk（效率更高）

```bash
[root@localhost awk]# cat awk1.sh
#!/bin/bash
sum=0
for((i=0;i<=100000;i++))
do
  ((sum+=i))
done
echo $sum
$ bash awk1.sh
5000050000
```

```bash
[root@localhost awk]# cat awk2.sh

#!/bin/bash

awk 'BEGIN{while(i++<100000)sum+=i;printf("%d\n",sum)}'
$ bash awk2.sh
5000050000
```



