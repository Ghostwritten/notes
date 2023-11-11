#  Shell 减法
tags: 数学


---

##  1. 数字相减

 - `subtraction.sh`

```bash
#!/usr/bin/env bash
printf "Enter the First Number: "
read -r a
printf "Enter the Second Number: "
read -r b
echo "$a - $b = $((a - b))"
```
执行：

```bash
$ bash subtraction.sh
Enter the First Number: 3
Enter the Second Number: 4
3 - 4 = -1

$ bash subtraction.sh
Enter the First Number: 5
Enter the Second Number: 3
5 - 3 = 2
```
---

更多阅读：

 - [Shell 加法](https://blog.csdn.net/xixihahalelehehe/article/details/105901300)

