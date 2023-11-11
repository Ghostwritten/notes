#  python 模块 commands 执行命令



----
## 1. 介绍
[commands](https://docs.python.org/3/using/cmdline.html)模块是python的内置模块，他共有三个函数，使用help(commands)可以查看到

## 2. 方法

```bash
FUNCTIONS
    getoutput(cmd)
        Return output (stdout or stderr) of executing cmd in a shell.

    getstatus(file)
        Return output of "ls -ld <file>" in a string.

    getstatusoutput(cmd)
        Return (status, output) of executing cmd in a shell.
```

### 2.1 commands.getstatusoutput(cmd)
返回一个元组（status，output） 
status代表的shell命令的返回状态，如果成功的话是0；output是shell的返回的结果
用os.popen()执行命令cmd, 然后返回两个元素的元组(status, result)，其中 status为int类型，result为string类型。cmd执行的方式是{ cmd ; } 2>&1, 这样返回结果里面就会包含标准输出和标准错误.
```python
>>> import commands
>>> status, output = commands.getstatusoutput("ls")
>>> print status
0
>>> print output
atom:
bookstore
cookie.py~
```

### 2.2 commands.getstatus（file）
只返回执行的结果, 忽略返回值.返回ls -ld file执行的结果. 

```bash
commands.getstatus（file）
```
**注意：该函数已被python丢弃，不建议使用**

### 2.3 commands.getoutput(cmd)
只返回执行的结果, 忽略返回值.
```python
>>> print commands.getoutput("ls")
atom:
bookstore
cookie.py~
```

## 3 练习

```bash
#!/usr/bin/python
#coding:utf-8
import os,sys,commands

def openfile():
    grains = {}
    _open_file=65533
    try:
        getulimit=commands.getstatusoutput('source /etc/profile;ulimit -n')
    except Exception,e:
        pass
    if getulimit[0]==0:
       _open_file=int(getulimit[1])
    grains['max_open_file'] = _open_file
    print grains
    return grains
openfile()
```
输出:

```bash
$ python com1.py 
{'max_open_file': 1024}
```
参考：

 - [Command Line Python](https://learn.adafruit.com/using-python-on-windows-10/command-line-python)
 - [How to Run Your Python Scripts](https://realpython.com/run-python-scripts/)
 - [7 Commands in Python to Make Your Life Easier](https://betterprogramming.pub/7-commands-in-python-to-make-your-life-easier-d48dd0992b57)
 - [Python Commands List](https://www.interviewbit.com/blog/python-commands/)

