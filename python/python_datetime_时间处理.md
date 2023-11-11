#  python 模块 datetime时间处理



---
##  1. 定义的类　

```bash
datetime.date　　　　--表示日期的类。常用的属性有year, month, day
datetime.time　　　　--表示时间的类。常用的属性有hour, minute, second, microsecond
datetime.datetime　　--表示日期时间
datetime.timedelta　  --表示时间间隔，即两个时间点之间的长度
```

### 1.1 date类
date类表示日期，构造函数如下 ：

```bash
datetime.date(year, month, day);
year (1-9999)
month (1-12)
day (1-31)
date.today()　　--返回一个表示当前本地日期的date对象
date.fromtimestamp(timestamp) --根据给定的时间戮，返回一个date对象
```

```bash
#根据给定的时间戮，返回一个date对象
> print(datetime.date.fromtimestamp(time.time()))
2020-10-10
#返回一个表示当前本地日期的date对象
> print(datetime.datetime.today())
2020-10-10 15:31:14.152035
```


```bash
date.year() 　　--取给定时间的年
date.month()　　--取时间对象的月
date.day()　　--取给定时间的日
date.replace()　　--生成一个新的日期对象，用参数指定的年，月，日代替原有对象中的属性
date.timetuple()　　--返回日期对应的time.struct_time对象
date.weekday()　　--返回weekday,Monday == 0 ... Sunday == 6
date.isoweekday() --返回weekday,Monday == 1 ... Sunday == 7
date.ctime()　　　　--返回给定时间的字符串格式
```

```bash
#取时间对象的年
print(datetime.date.today().year)
#取时间对象的月
print(datetime.date.today().month)
#取时间对象的日
print(datetime.date.today().day)
#生成一个新的日期对象，用参数指定的年，月，日代替原有对象中的属性
print(datetime.date.today().replace(2010,6,12))
#返回给定时间的时间元组/对象
print(datetime.date.today().timetuple())
#返回weekday，从0开始
print(datetime.date.today().weekday())
#返回weekday,从1开始
print(datetime.date.today().isoweekday())
#返回给定时间的字符串格式
print(datetime.date.today().ctime())
```

## 2. 扩展
```python
import datetime
mtime = path.stat().st_mtime
timestamp_str = datetime.datetime.fromtimestamp(mtime).strftime('%Y-%m-%d-%H:%M')
```
示例：[添加链接描述](https://www.w3schools.com/python/python_datetime.asp)

```bash
#!/usr/bin/python

from datetime import datetime, timezone
from pathlib import Path

path = Path('foo')
path.touch()
stat_result = path.stat()
modified = datetime.fromtimestamp(stat_result.st_mtime, tz=timezone.utc)
```

输出：

```bash
$ python3 test.py 
modified 2021-01-17 09:04:37.146791+00:00
```

参考：

 - [geeksforgeeks Python datetime module](https://www.geeksforgeeks.org/python-datetime-module/)
 - [datetime — Basic date and time types](https://docs.python.org/3/library/datetime.html)
 - [w3schools Python Datetime](https://www.w3schools.com/python/python_datetime.asp)

