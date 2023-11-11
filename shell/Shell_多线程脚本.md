
#  Shell 多线程脚本


---
## 1. 千台主机ping是否连通
```bash
#!/bin/sh
#文本分割函数：将文本$1按份数$2进行分割
SplitFile()
{
    linenum=`wc -l $1 |awk '{print $1}'`
    if [[ $linenum -le $2 ]]
    then
        echo "The lines of this file is less then $2, Are you kidding me..."
        exit
    fi
    Split=`expr $linenum / $2`
    Num1=1
    FileNum=1
    test -d SplitFile || mkdir -p SplitFile
    rm -rf SplitFile/*
    while [ $Num1 -lt $linenum ]
    do
        Num2=`expr $Num1 + $Split`
        sed -n "${Num1}, ${Num2}p " $1 > SplitFile/$1-$FileNum
        Num1=`expr $Num2 + 1`
        FileNum=`expr $FileNum + 1`
    done
}
 
#Define some variables
SPLIT_NUM=${1:-10} #参数1表示分割成多少份即,开启多少个线程，默认10个
FILE=${2:-iplist}  #参数2表示分割的对象，默认iplist文件
 
#分割文件
SplitFile $FILE $SPLIT_NUM
 
#循环遍历临时IP文件
for iplist in $(ls ./SplitFile/*)
do
    #循环ping测试临时IP文件中的ip（丢后台）
    cat $iplist | while read ip
    do
        ping -c 4 -w 4 $ip >/dev/null && echo $ip | tee -ai okip.log #ping 可达的IP则写入日志
    done &     #在while循环后面加上&符号，让这个嵌套循环在后台执行
done
```
执行

```bash
$ bash ping.sh 3 ip_list
```

## 2. 网站健康状态检查
抓出中国博客联盟失联站点

```bash
#!/bin/bash
#Author:ZhangGe
#Date：2014-08-21
#Desc：Check the site of ZGboke Alliance.
#取出网站数据
data=`/usr/bin/mysql  -uroot -p123456 -e "use zgboke;select web_url from dir_websites where web_status='3';" -N -B | awk '{print $1}'`
if [ -z "$data" ];then
        echo "Faild to connect database!"
        exit 1
fi
test -f result.log && rm -f result.log
function delay {
        sleep 3
}
tmp_fifofile=/tmp/$$.fifo
mkfifo  $tmp_fifofile
exec 6<>$tmp_fifofile
rm  $tmp_fifofile
#定义并发线程数，需根据vps配置进行调整。
thread=100
for  ((i=0 ;i<$thread;i++ ))
do
        echo
done>&6
#开始多线程循环检测
for url in $data
do
        read -u6
        {
        #curl抓取网站http状态码
        code=`curl -o /dev/null --retry 3 --retry-max-time 8 -s -w %{http_code} $url`
        echo "$code ---> $url">>result.log
        #判断子线程是否执行成功，并输出结果
        delay &&  {
                echo  "$code ---> $url"
        }  ||  {
                echo  "Check thread error!"
        }
        echo  >& 6
}&
done
#等待所有线程执行完毕
wait
exec 6>&-
#找出非200返回码的站点
echo List of exception website:
cat result.log | grep -v 200
exit 0
```

参考：

 - [张戈分享一个入门级可控多线程shell脚本方案](https://zhang.ge/5071.html/comment-page-1/?unapproved=21898&moderation-hash=da6b959d1fa3767137c9f74d72acc85b#comment-21898)

