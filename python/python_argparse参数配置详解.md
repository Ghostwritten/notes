#  python 模块 argparse参数配置





----
## 1. 简介
通过`argparse`模块，可以轻松编写用户友好的命令行界面。 它解析`sys.argv`中定义的参数。

argparse模块还会自动生成帮助和使用消息，并在用户为程序提供无效参数时发出错误。

argparse是标准模块； 我们不需要安装它。

使用`ArgumentParser`创建一个解析器，并使用`add_argument()`添加一个新参数。 参数可以是可选的，必需的或定位的。

## 2. 可选参数

```bash
#!/usr/bin/env python
import argparse
parser = argparse.ArgumentParser()
parser.add_argument('-o', '--output', action='store_true', 
    help="shows output")
args = parser.parse_args()
if args.output:
    print("This is some output")
```
执行：

```bash
[root@localhost python]# python optional_arg.py
[root@localhost python]# python optional_arg.py -o
This is some output
[root@localhost python]# python optional_arg.py -output
usage: optional_arg.py [-h] [-o]
optional_arg.py: error: argument -o/--output: ignored explicit argument 'utput'

[root@localhost python]# python optional_arg.py --output
This is some output

[root@localhost python]# python optional_arg.py --help
usage: optional_arg.py [-h] [-o]

optional arguments:
  -h, --help    show this help message and exit
  -o, --output  shows output


[root@localhost python]# python optional_arg.py -h
usage: optional_arg.py [-h] [-o]

optional arguments:
  -h, --help    show this help message and exit
  -o, --output  shows output
```

##  3. 必需参数
required_arg.py
```bash
#!/usr/bin/env python
import argparse
# required arg
parser = argparse.ArgumentParser()
parser.add_argument('--name', required=True)
args = parser.parse_args()
print(f'Hello {args.name}')
```
该示例必须指定name选项； 否则失败

```c
[root@localhost python]# python3 required_arg.py 
usage: required_arg.py [-h] --name NAME
required_arg.py: error: the following arguments are required: --name
[root@localhost python]# python3 required_arg.py --name hugo
Hello hugo
```
##  4. 位置参数
以下示例适用于位置参数。 它们是使用add_argument()创建的。
positional_arg.py

```bash
#!/usr/bin/env python

import argparse

# positional args

parser = argparse.ArgumentParser()

parser.add_argument('name')
parser.add_argument('age')

args = parser.parse_args()

print(f'{args.name} is {args.age} years old')
```
执行

```c
[root@localhost python]# python3 positional_arg.py
usage: positional_arg.py [-h] name age
positional_arg.py: error: the following arguments are required: name, age

[root@localhost python]# python3 positional_arg.py hugo 24
hugo is 24 years old
```

##  5. argparse 目标
`add_argument()`的`dest`选项为参数指定名称。 如果未给出，则从选项中推断出来。
dest.py

```bash
#!/usr/bin/env python
import argparse
import datetime
# dest gives a different name to a flag
parser = argparse.ArgumentParser()
parser.add_argument('-n', dest='now', action='store_true', help="shows now")
args = parser.parse_args()
# we can refer to the flag
# by a new name
if args.now:
    now = datetime.datetime.now()
    print(f"Now: {now}")
```
程序将now名称赋予`-n`参数。

```bash
$ dest.py -n
Now: 2019-03-22 17:37:40.406571
```

##  6. argparse 类型
type参数确定参数类型。

rand_int.py

```bash
#!/usr/bin/env python

import argparse
import random
# type determines the type of the argument
parser = argparse.ArgumentParser()
parser.add_argument('-n', type=int, required=True, 
    help="define the number of random integers")
args = parser.parse_args()
n = args.n
for i in range(n):
    print(random.randint(-100, 100))
```
程序显示从-100 到 100 的 n 个随机整数。

```bash
$ rand_int.py -n 3
92
-61
-61
```
##   7. argparse 默认
如果未指定default选项，则指定默认值。

power.py

```bash
#!/usr/bin/env python
import argparse
# required defines a mandatory argument 
# default defines a default value if not specified
parser = argparse.ArgumentParser()
parser.add_argument('-b', type=int, required=True, help="defines the base value")
parser.add_argument('-e', type=int, default=2, help="defines the exponent value")
args = parser.parse_args()
val = 1
base = args.b
exp = args.e
for i in range(exp):
    val *= base
print(val)
```
该示例计算指数。 不需要指数值； 如果未给出，则默认值为 2。

```bash
$ power.py -b 3
9
$ power.py -b 3 -e 3
27
```
##  8. argparse metavar
`metavar`选项为错误的期望值命名，并提供帮助输出。

metavar.py

```bash
#!/usr/bin/env python
import argparse
# metavar gives name to the expected value 
# in error and help outputs
parser = argparse.ArgumentParser()
parser.add_argument('-v', type=int, required=True, metavar='value', 
    help="computes cube for the given value")
args = parser.parse_args()
print(args)
val = args.v
print(val * val * val)
```
执行：

```bash
[root@localhost python]# python3 metavar.py 
usage: metavar.py [-h] -v value
metavar.py: error: the following arguments are required: -v
[root@localhost python]# python3 metavar.py -h
usage: metavar.py [-h] -v value

optional arguments:
  -h, --help  show this help message and exit
  -v value    computes cube for the given value
[root@localhost python]# python3 metavar.py -v 
usage: metavar.py [-h] -v value
metavar.py: error: argument -v: expected one argument
[root@localhost python]# python3 metavar.py -v 3
Namespace(v=3)
27
```
##  9. argparse 追加操作
append操作允许对重复选项进行分组。

appending.py

```bash
#!/usr/bin/env python
import argparse
# append action allows to group repeating
# options
parser = argparse.ArgumentParser()
parser.add_argument('-n', '--name', dest='names', action='append', 
    help="provides names to greet")
args = parser.parse_args()
names = args.names
for name in names:
    print(f'Hello {name}!')
```

该示例生成问候语，并使用n或name选项指定所有名称。 它们可以重复多次重复。

```bash
[root@localhost python]# python3 appending.py 3
usage: appending.py [-h] [-n NAMES]
appending.py: error: unrecognized arguments: 3
[root@localhost python]# python3 appending.py -n Perter -n Lucy --name Jane
Hello Perter!
Hello Lucy!
Hello Jane!
```
##  10. argparse nargs
nargs指定应使用的命令行参数的数量。

charseq.py

```bash
#!/usr/bin/env python

import argparse
import sys
# nargs sets the required number of argument values
# metavar gives name to argument values in error and help output
parser = argparse.ArgumentParser()
parser.add_argument('chars', type=str, nargs=2, metavar='c',
                    help='starting and ending character')
args = parser.parse_args()
try:
    v1 = ord(args.chars[0])
    v2 = ord(args.chars[1])
except TypeError as e:
    print('Error: arguments must be characters')
    parser.print_help()
    sys.exit(1)

if v1 > v2:
    print('first letter must precede the second in alphabet')
    parser.print_help()
    sys.exit(1)
```
该示例显示了从字符一到字符二的字符序列。 它需要两个参数。使用nargs=2，我们指定期望两个参数。

```bash
$ charseq.py e k
e f g h i j k
```
程序显示从 e 到 k 的字符序列。

可以使用`*`字符设置可变数量的参数。

var_args.py

```bash
#!/usr/bin/env python
import argparse
# * nargs expects 0 or more arguments
parser = argparse.ArgumentParser()
parser.add_argument('num', type=int, nargs='*')
args = parser.parse_args()
print(f"The sum of values is {sum(args.num)}")
```
执行

```bash
$ var_args.py 1 2 3 4 5
The sum of values is 15
```
##  11. argparse 选择
choices选项将参数限制为给定列表。
mytime.py

```bash
#!/usr/bin/env python

import argparse
import datetime
import time

# choices limits argument values to the 
# given list

parser = argparse.ArgumentParser()

parser.add_argument('--now', dest='format', choices=['std', 'iso', 'unix', 'tz'],
                    help="shows datetime in given format")

args = parser.parse_args()
fmt = args.format

if fmt == 'std':
    print(datetime.date.today())
elif fmt == 'iso':
    print(datetime.datetime.now().isoformat())
elif fmt == 'unix':
    print(time.time())
elif fmt == 'tz':
    print(datetime.datetime.now(datetime.timezone.utc))
```
执行
在示例中，now选项可以接受以下值：std，iso，unix或tz。

```bash
[root@localhost python]# python3 mytime.py
[root@localhost python]# python3 mytime.py --now
usage: mytime.py [-h] [--now {std,iso,unix,tz}]
mytime.py: error: argument --now: expected one argument
[root@localhost python]# python3 mytime.py --now std
2021-11-08
[root@localhost python]# python3 mytime.py --now iso
2021-11-08T15:06:06.640909
[root@localhost python]# python3 mytime.py --now unix
1636355170.17874
[root@localhost python]# python3 mytime.py --now tz
2021-11-08 07:06:14.381828+00:00
```
## 12. 示例
###  12.1 模仿 Linux head 命令
 它显示了文件开头的 n 行文本

```bash
[root@localhost python]# cat words.txt 
sky
top
forest
wood
lake
wood
```
head.py

```bash
#!/usr/bin/env python

import argparse
from pathlib import Path

# head command
# working with positional arguments

parser = argparse.ArgumentParser()

parser.add_argument('f', type=str, help='file name')
parser.add_argument('n', type=int, help='show n lines from the top')

args = parser.parse_args()

filename = args.f

lines = Path(filename).read_text().splitlines()

for line in lines[:args.n]:
    print(line) 
```
该示例有两个选项：f表示文件名，-n表示要显示的行数。

```bash
[root@localhost python]# python3 head.py words.txt 3
sky
top
forest
[root@localhost python]# python3 head.py
usage: head.py [-h] f n
head.py: error: the following arguments are required: f, n
```

参考：

 - [Python argparse 教程](https://geek-docs.com/python/python-tutorial/python-argparse.html#Python_argparse-5)
- [python 模块运用快速学习手册](https://ghostwritten.blog.csdn.net/article/details/112751406)
 - [python argparse参数配置](https://ghostwritten.blog.csdn.net/article/details/121199110)
 - [python click模块参数处理](https://ghostwritten.blog.csdn.net/article/details/106124675)
 - [python sys模块](https://ghostwritten.blog.csdn.net/article/details/106693564)
 - [argparse — Parser for command-line options, arguments and sub-commands](https://docs.python.org/3/library/argparse.html)
