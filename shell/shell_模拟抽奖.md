#  shell 模拟抽奖
tags: 游戏

随机滚动电影名，ctrl + c暂停

```bash
$ cat films.txt 
银翼杀手
阿凡达
黑镜
复仇联盟
荒岛余生
降临
地心引力
云图
```

```bash
#!/bin/bash

film=`cat films.txt | wc -l`
while :
do
  numfilm=$[RANDOM%$film+1]
  #filmname=`head -$numfilm films.txt | tail -1`
  filmname=`sed -n "${numfilm}p" films.txt`
  echo "$filmname"
  sleep 1
  clear
done
```

```bash
$ bash while1.sh 
地心引力
^C
```

