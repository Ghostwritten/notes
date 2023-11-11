## 1. 介绍
定时任务 `cron job`被用于安排那些需要被周期性执行的命令。利用它，你可以配置某些命令或者脚本，让它们在某个设定的时间内周期性地运行。cron 是 Linux 或者类 Unix 系统中最为实用的工具之一。cron 服务（守护进程）在系统后台运行，并且会持续地检查 `/etc/crontab` 文件和 `/etc/cron.*/`目录。它同样也会检查 `/var/spool/cron/` 目录。

在linux平台上如果需要实现任务调度功能可以编写cron脚本来实现。
以某一频率执行任务
linux缺省会启动crond进程，crond进程不需要用户启动、关闭。
crond进程负责读取调度任务并执行，用户只需要将相应的调度脚本写入cron的调度配置文件中。
cron的调度文件有以下几个：

   1. crontab
   2. cron.d
   3. cron.daily
   4. cron.hourly
   5. cron.monthly
   6. cron.weekly 

如果用的任务不是以hourly monthly weekly方式执行，则可以将相应的crontab写入到crontab 或cron.d目录中。

## 2. 安装部署
```bash
# 安装crontab：
$ yum install crontabs

# 启动服务
$ systemctl start crond.service

# 关闭服务
$ systemctl stop crond.service

# 重启服务
$ systemctl restart crond.service

# 重新载入配置
$ systemctl reload crond.service

# 查看crontab服务状态：
$ systemctl status crond.service

# 加入开机自动启动：
$ systemctl enable crond.service
```

## 3. 命令
Crontab 选项
以下是 crontab 的有效选项:

```bash
$ crontab -r     #删除 crontab 文件。
$ crontab -ir    #删除 crontab 文件前提醒用户。
$ crontab file [-u user]   #用指定的文件替代目前的crontab。
$ crontab -[-u user]    #用标准输入替代目前的crontab.
$ crontab -1[user]    #列出用户目前的crontab.
$ crontab -e[user]    #编辑用户目前的crontab，如果文件不存在会自动创建.
$ crontab -d[user]    #删除用户目前的crontab.
$ crontab -c dir      #指定crontab的目录。
```

## 4. crontab配置

```bash
$  cat /etc/crontab
SHELL=/bin/bash
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=root

# For details see man 4 crontabs

# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name  command to be executed
```
前四行是用来配置crond任务运行的环境变量:

 - 第一行SHELL变量指定了系统要使用哪个shell，这里是bash。
 - 第二行PATH变量指定了系统执行命令的路径。
 - 第三行MAILTO变量指定了crond的任务执行信息将通过电子邮件发送给root用户，如果MAILTO变量的值为空，则表示不发送任务执行信息给用户。
 - 第四行的HOME变量指定了在执行命令或者脚本时使用的主目录。

格式：

```bash
minute   hour   day   month   week   command
```

其中：

 - minute： 表示分钟，可以是从0到59之间的任何整数。
 - hour：表示小时，可以是从0到23之间的任何整数。
 - day：表示日期，可以是从1到31之间的任何整数。
 - month：表示月份，可以是从1到12之间的任何整数。
 - week：表示星期几，可以是从0到7之间的任何整数，这里的0或7代表星期日。
 - command：要执行的命令，可以是系统命令，也可以是自己编写的脚本文件。

在以上各个字段中，还可以使用以下特殊字符：

 - 星号 (*) : 代表所有可能的值，例如month字段如果是星号，则表示在满足其它字段的制约条件后每月都执行该命令操作。
 - 逗号（,）： 可以用逗号隔开的值指定一个列表范围，例如，“1,2,5,7,8,9”
 - 中杠（-）： 可以用整数之间的中杠表示一个整数范围，例如“2-6”表示“2,3,4,5,6”
 - 正斜线（/）：可以用正斜线指定时间的间隔频率，例如“0-23/2”表示每两小时执行一次。同时正斜线可以和星号一起使用，例如*/10，如果用在minute字段，表示每十分钟执行一次。

## 5. 实战
### 5.1 一般添加定时任务
```bash
# 每1分钟执行一次command
* * * * * command

每小时的第3和第15分钟执行
3,15 * * * * command

# 在上午8点到11点的第3和第15分钟执行
3,15 8-11 * * * command

# 每隔两天的上午8点到11点的第3和第15分钟执行
3,15 8-11 */2 * * command

# 每个星期一的上午8点到11点的第3和第15分钟执行
3,15 8-11 * * 1 command

# 每晚的21:30重启smb 
30 21 * * * /etc/init.d/smb restart

# 每月1、10、22日的4 : 45重启smb 
45 4 1,10,22 * * /etc/init.d/smb restart

# 每周六、周日的1 : 10重启smb
10 1 * * 6,0 /etc/init.d/smb restart

# 每天18 : 00至23 : 00之间每隔30分钟重启smb 
0,30 18-23 * * * /etc/init.d/smb restart

# 每星期六的晚上11 : 00 pm重启smb 
0 23 * * 6 /etc/init.d/smb restart

# 每一小时重启smb 
\* */1 * * * /etc/init.d/smb restart

# 晚上11点到早上7点之间，每隔一小时重启smb 
\* 23-7/1 * * * /etc/init.d/smb restart

# 每月的4号与每周一到周三的11点重启smb 
0 11 4 * mon-wed /etc/init.d/smb restart

# 一月一号的4点重启smb 
0 4 1 jan * /etc/init.d/smb restart

# 每小时执行/etc/cron.hourly目录内的脚本
01   *   *   *   *     root run-parts /etc/cron.hourly

run-parts这个参数了，如果去掉这个参数的话，后面就可以写要运行的某个脚本名，而不是目录名了
```

### 5.2 特殊字符添加定时任务

```bash
特殊字符	含义
@reboot	  在每次启动时运行一次
@yearly	  每年运行一次,等同于 “0 0 1 1 *”.
@annually	(同 @yearly)
@monthly	每月运行一次, 等同于 “0 0 1 * *”.
@weekly	  每周运行一次, 等同于 “0 0 * * 0”.
@daily	  每天运行一次, 等同于 “0 0 * * *”.
@midnight	(同 @daily)
@hourly	  每小时运行一次, 等同于 “0 * * * *”.
```
例如
每小时运行一次 ntpdate 命令

```bash
@hourly /path/to/ntpdate
```

## 6. 注意事项
### 6.1 注意清理系统用户的邮件日志
每条任务调度执行完毕，系统都会将任务输出信息通过电子邮件的形式发送给当前系统用户，这样日积月累，日志信息会非常大，可能会影响系统的正常运行，因此，将每条任务进行重定向处理非常重要。
例如，可以在crontab文件中设置如下形式，忽略日志输出：

```bash
0 */3 * * * /usr/local/apache2/apachectl restart > /dev/null 2>&1
```

`/dev/null 2>&1`表示先将标准输出重定向到/dev/null，然后将标准错误重定向到标准输出，由于标准输出已经重定向到了/dev/null，因此标准错误也会重定向到/dev/null，这样日志输出问题就解决了。

### 6.2 生效问题
新创建的cron job，不会马上执行，至少要过2分钟才执行。如果重启cron则马上执行。
当crontab突然失效时，可以尝试/etc/init.d/crond restart解决问题。或者查看日志看某个job有没有执行/报错

```bash
$ tail -f /var/log/cron
```

### 6.3 别乱运行crontab -r
它从Crontab目录（/var/spool/cron）中删除用户的Crontab文件。删除了该用户的所有crontab都没了。


参考链接：
[https://juejin.im/post/5cdaa5f9f265da03a00ff32a](https://juejin.im/post/5cdaa5f9f265da03a00ff32a)

[https://linux.cn/article-7513-1.html](https://linux.cn/article-7513-1.html)

