# shell 石头剪刀布
tags: 数学

compare.sh 
```powershell
#!/bin/bash

#1.列出计算机随即选择的可能.
#2.列出人人为选择的方式.
#3.表达计算机的选择结果与人的做出比较.
#4.布置虚拟现实环境.

game=(石头 剪刀 布)

num=$[RANDOM%3]

echo "请根据下列提示选择您的出拳手势"

echo "1.石头"

echo "2.剪刀"

echo "3.布"

read -p "请选择1-3:" num2

echo  "计算机选择：${game[$num]}"

case $num2 in
  1)
    if [[ $num -eq 0 ]];then
       echo "平局"
    elif [[ $num -eq 1 ]];then
       echo "你赢啦"
    else
       echo "计算机赢啦"
    fi
    ;;

  2)
    if [[ $num -eq 0 ]];then
       echo "计算机赢啦"
    elif [[ $num -eq 1 ]];then
       echo "平局"
    else 
       echo "你赢啦"
    fi
    ;;

  3)
    if [[ $num -eq 0 ]];then
       echo "你赢啦"
    elif [[ $num -eq 1 ]];then
       echo "计算机赢啦"
    else
       echo "平局"
    fi
    ;;
  
  *)
     echo "必须输入1-3的数字"
esac
```
执行：

```powershell
$ bash compare2.sh 
请根据下列提示选择您的出拳手势
1.石头
2.剪刀
3.布
请选择1-3:1
 计算机选择：剪刀
你赢啦
```

