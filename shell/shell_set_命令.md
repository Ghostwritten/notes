#  shell set 命令



---
## 1. set 
会显示所有的环境变量和 Shell 函数

```bash
$ cat script.sh
set 
a=1
b=2
c=3
echo $a
echo $b
echo $c
echo $d
```

```bash
$ script.sh
 bash script.sh 
BASH=/bin/bash
BASHOPTS=cmdhist:complete_fullquote:extquote:force_fignore:hostcomplete:interactive_comments:progcomp:promptvars:sourcepath
BASH_ALIASES=()
BASH_ARGC=()
BASH_ARGV=()
BASH_CMDS=()
BASH_LINENO=([0]="0")
BASH_SOURCE=([0]="script.sh")
BASH_VERSINFO=([0]="4" [1]="3" [2]="48" [3]="1" [4]="release" [5]="x86_64-pc-linux-gnu")
BASH_VERSION='4.3.48(1)-release'
```
## 2. set -u
遇到不存在的变量就会报错，并停止执行，或者`-o nounset`

```bash
$ cat script.sh
set -u #-o nounset
a=1
b=2
c=3
echo $a
echo $b
echo $c
echo $d
```

```bash
$ bash script.sh
1
2
3
script.sh: line 9: d: unbound variable
```
## 3. set -x
用来在运行结果之前，先输出执行的那一行命令,或者`set -o xtrace`

```bash
$ cat script.sh 

set -x #set -o xtrace
a=1
b=2
c=3
echo $a
echo $b
echo $c
echo $d
```
执行结果：

```bash
$ bash script.sh
+ a=1
+ b=2
+ c=3
+ echo 1
1
+ echo 2
2
+ echo 3
3
+ echo
```
## 4. bash 的错误处理

```bash
#!/usr/bin/env bash

foo
echo bar
```

上面脚本中，foo是一个不存在的命令，执行时会报错。但是，Bash 会忽略这个错误，继续往下执行。


```bash
$ bash script.sh
script.sh:行3: foo: 未找到命令
bar
```

可以看到，Bash 只是显示有错误，并没有终止执行。

这种行为很不利于脚本安全和除错。实际开发中，如果某个命令失败，往往需要脚本停止执行，防止错误累积。这时，一般采用下面的写法。


```bash
command || exit 1
```

上面的写法表示只要command有非零返回值，脚本就会停止执行。

如果停止执行之前需要完成多个操作，就要采用下面三种写法。


```bash
# 写法一
command || { echo "command failed"; exit 1; }

# 写法二
if ! command; then echo "command failed"; exit 1; fi

# 写法三
command
if [ "$?" -ne 0 ]; then echo "command failed"; exit 1; fi
```

另外，除了停止执行，还有一种情况。如果两个命令有继承关系，只有第一个命令成功了，才能继续执行第二个命令，那么就要采用下面的写法。


```bash
command1 && command2
```
## 5. set -e
set -e从根本上解决了这个问题，它使得脚本只要发生错误，就终止执行。或者`set -o errexit`


```bash
#!/usr/bin/env bash
set -e

foo
echo bar
```

执行结果如下。


```bash
$ bash script.sh
script.sh:行4: foo: 未找到命令
```

可以看到，第4行执行失败以后，脚本就终止执行了。

set -e根据返回值来判断，一个命令是否运行失败。但是，某些命令的非零返回值可能不表示失败，或者开发者希望在命令失败的情况下，脚本继续执行下去。这时可以暂时关闭set -e，该命令执行结束后，再重新打开set -e。


```bash
set +e
command1
command2
set -e
```

上面代码中，set +e表示关闭-e选项，set -e表示重新打开-e选项。

还有一种方法是使用command || true，使得该命令即使执行失败，脚本也不会终止执行。


```bash
#!/bin/bash
set -e

foo || true
echo bar
```
## 6. set -o pipefail
set -e有一个例外情况，就是不适用于管道命令。

所谓管道命令，就是多个子命令通过管道运算符（|）组合成为一个大的命令。Bash 会把最后一个子命令的返回值，作为整个命令的返回值。也就是说，只要最后一个子命令不失败，管道命令总是会执行成功，因此它后面命令依然会执行，set -e就失效了。

请看下面这个例子。


```bash
#!/usr/bin/env bash
set -e

foo | echo a
echo bar
```

执行结果如下。


```bash
$ bash script.sh
a
script.sh:行4: foo: 未找到命令
bar
```

上面代码中，foo是一个不存在的命令，但是foo | echo a这个管道命令会执行成功，导致后面的echo bar会继续执行。

set -o pipefail用来解决这种情况，只要一个子命令失败，整个管道命令就失败，脚本就会终止执行。


```bash
#!/usr/bin/env bash
set -eo pipefail

foo | echo a
echo bar
```

运行后，结果如下。


```bash
$ bash script.sh
a
script.sh:行4: foo: 未找到命令
```

参考：

 - [阮一峰 set 命令，shopt 命令l](https://wangdoc.com/bash/set.html)
 - [Linux set Command & How to Use it {9 Examples}](https://phoenixnap.com/kb/linux-set)
 - [set - Set or unset values of shell options and positional parameters.](https://linuxcommand.org/lc3_man_pages/seth.html)
 -  [The Set Builtin](https://www.gnu.org/software/bash/manual/html_node/The-Set-Builtin.html)

