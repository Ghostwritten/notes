#  Linux Command egrep



## 1. 特点

基本曾则：优点（兼容性强，缺点（繁琐））
拓展：优点（简单），缺点：（兼容性弱）
egrep：grep -E
##  2. 举例

```bash
 grep '^r' /etc/passwd----'^id' /etc/inittab----------'^HOSTNAME' /etc/sysconfig/network
 grep 'localhost$' /etc/hosts                      #匹配以某字符结尾的内容
grep '^root\|^daemon' /etc/passwd                   #（基本）多条件显示内容
 egrep '^root|^daemon' /etc/passwd                  #（扩展）多条件显示内容
grep -q '^192.168.4.4' /etc/hosts && echo "YES" || echo "NO"    #选项 -q 表示 quiet（静默）的意思，结合此选项可以只做检索而并不输出，通常在脚本内用来识别查找的目标是否存在，通过返回状态 $? 来判断，这样可以忽略无关的文本信息
egrep  '/sbin/nologin$' /etc/passwd  ///确认是否正确      #统计本地用户中登录Shell为“/sbin/nologin”的用户个数
egrep -c '/sbin/nologin$' /etc/passwd              #统计个数
egrep '/bin/bash$' /etc/passwd | wc -l             #统计个数
egrep '.' /etc/rc.local                            #显示非空行的内容
egrep -v '.' /etc/rc.local                         #显示空行
egrep '^$' /etc/rc.local                            #显示空行
egrep 'f+' /etc/rc.loca                         #至少出现一次
egrep 'init(ial)?' /etc/rc.local                 #匹配出现init，initab的行
egrep 'stuf*' /etc/rc.local                     #匹配stuf后面出现任意次数的行
egrep '^r.*nologin$' /etc/passwd                 #.*匹配任意多个字符的行（首，尾之间）
元字符 {} —— 限定出现的次数范围
egrep '(ab){3}' brace.txt                        #匹配ab出现3次的行
egrep '(ab){2,4}' brace.txt                      #匹配ab出现2，4次的行
egrep '(ab){3,}' brace.txt                       #匹配ab至少出现3次的行
元字符 [] —— 匹配范围内的单个字符
egrep 'ab[cd]' brace.txt
egrep '[A-Z]' brace.tx
egrep '[^ a-zA-Z]' brace.txt
单词边界匹配
egrep '\binit\b' /etc/rc.local                  #同
egrep '\<init\>' /etc/rc.local                  #同
egrep 'll\>' /etc/rc.local
egrep 'll\b' /etc/rc.local
多个条件的组合
egrep '\<IDE\>|\<CDROM\>' /var/log/dmesg         #通过dmesg启动日志查看与IDE接口、CDROM光盘相关的设备信息
egrep -i 'eth|network|bluetooth' /var/log/dmesg      #通过dmesg启动日志查看蓝牙设备、网卡设备相关的信息
利用正则表达式完成检索任务
egrep -c ".*" /etc/httpd/conf/httpd.conf      #总行数
egrep -c "#" /etc/httpd/conf/httpd.conf       #显示注释行数
egrep -c "^$" /etc/httpd/conf/httpd.con        #显示空行数
egrep -c -v '#|^$'  /etc/httpd/conf/httpd.conf  #显示除掉空行与注释行的行数

匹配MAC地址、邮箱地址、IP地址
echo $MAC01 | egrep -q '[0-9a-fA-F]{2}(:[0-9a-fA-F]{2}){5}' && echo "有效" || echo "无效"                       #检查MAC地址是否有效
匹配邮箱地址格式：用户名与域名之间以 @ 分隔 
用户名不少于3个字符，可能由字母、下划线、句点 . 、数字组成 
域名应至少有一个 . 分隔，分隔的各部分至少2个字符，可能由字母、减号、数字组成 
[0-9a-zA-Z_.]{3,}@[0-9a-zA-Z.-]{2,}(\.[0-9a-zA-Z-]{2,})+
 egrep '[0-9a-zA-Z_.]{3,}@\ [0-9a-zA-Z.-]{2,}(\.[0-9a-zA-Z-]{2,})+' mailadd.txt   #匹配有效邮箱地址
匹配主机名格式：由 . 分隔，至少包括3组字符串
每组字符串不少于2个字符，可能由字母、减号、数字、下划线组成 
主机名后必须是单词边界，主机名前不能有@符号 
^[^@][0-9a-zA-Z_-]{2,}(\.[0-9a-zA-Z_-]{2,}){2,}\>
egrep '^[^@][0-9a-zA-Z_-]{2,}(\.[0-9a-zA-Z_-]{2,}){2,}\>'  mailadd.txt     #匹配有效主机名
匹配IP地址
以 . 分隔，一共由四组十进制数构成 
每组数值的范围为0-255，字符宽度为1-3位 
前后必须是单词边界 
\<[0-9]{1,3}(\.[0-9]{1,3}){3}\>
 ifconfig | egrep '\<[0-9]{1,3}(\.[0-9]{1,3}){3}\>
```



