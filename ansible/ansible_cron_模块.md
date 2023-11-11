## 1. 模块方法
```bash
$ ansible-doc -s cron

 1 backup：对远程主机上的原任务计划内容修改之前做备份 
 2 cron_file：如果指定该选项，则用该文件替换远程主机上的cron.d目录下的用户的任务计划 
 3 day：日（1-31，*，*/2,……） 
 4 hour：小时（0-23，*，*/2，……）  
 5 minute：分钟（0-59，*，*/2，……） 
 6 month：月（1-12，*，*/2，……） 
 7 weekday：周（0-7，*，……）
 8 job：要执行的任务，依赖于state=present 
 9 name：该任务的描述 
10 special_time：指定什么时候执行，参数：reboot,yearly,annually,monthly,weekly,daily,hourly 
11 state：确认该任务计划是创建还是删除 
12 user：以哪个用户的身份执行
```

## 2. 使用说明

```bash
$ ansible db -m cron -a 'minute=""  hour="" day="" month="" weekday="" job="" name="（必须填写）" state=
```
**注意：**
```bash
1、定时设置指定值的写入即可，没有设置的可以不写（默认是*）
2、name必须写
3、state有两个状态：present（添加（默认值））or absent（移除）
```

## 3. 使用范例

1、添加定时任务

```bash
$ ansible db -m cron -a 'minute="*/10" job="/bin/echo hello" name="test cron job" state="present"'
$ ansible db -a "crontab -l"
```
2、移除定时任务
```bash
$ ansible db -m cron -a 'minute="*/10" job="/bin/echo hello" name="test cron job" state="absent"'
$ ansible db -a "crontab -l"
```





