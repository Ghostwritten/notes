#  Shell 函数



---
## 1. 函数定义

```powershell
function name() {
    statements
    [return value]
}
```

对各个部分的说明：

 - function是 Shell 中的关键字，专门用来定义函数，可以省略；
 - name是函数名；
 - statements是函数要执行的代码，也就是一组语句；
 - return value表示函数的返回值，其中 return 是 Shell 关键字，专门用在函数中返回一个值；这一部分可以写也可以不写。

由{ }包围的部分称为函数体，调用一个函数，实际上就是执行函数体中的代码

## 2. 函数调用

调用 Shell 函数时可以给它传递参数，也可以不传递。如果不传递参数，直接给出函数名字即可：

```powershell
name
```

如果传递参数，那么多个参数之间以空格分隔：

```powershell
name param1 param2 param3
```

不管是哪种形式，函数名字后面都不需要带括号。

和其它编程语言不同的是，Shell 函数在定义时不能指明参数，但是在调用时却可以传递参数，并且给它传递什么参数它就接收什么参数。

Shell 也不限制定义和调用的顺序，你可以将定义放在调用的前面，也可以反过来，将定义放在调用的后面。
实例：

```powershell
#!/bin/bash
function show() {
    echo "hello , you are calling the function"
}
echo "first time call the function"
show
echo "second time call the function"
show

[root@localhost base]$ bash fun1.sh
first time call the function
hello , you are calling the function
second time call the function
hello , you are calling the function

```

## 3. 参数传递
函数可以通过位置变量传递参数。例如

```powershell
函数名 参数1 参数2 参数3 参数4
```

当函数执行时，$1 对应 参数1，其他依次类推。

实例：

```powershell
#!/bin/bash
function show() {
    echo "hello , you are calling the function  $1"
}
echo "first time call the function"
show first
echo "second time call the function"
show second

[root@localhost base]$ bash fun2.sh
first time call the function
hello , you are calling the function  first
second time call the function
hello , you are calling the function  second

```
## 4. 函数的返回值

函数中的关键字“return”可以放到函数体的任意位置，通常用于返回某些值，Shell在执行到return之后，就停止往下执行，返回到主程序的调用行，return的返回值只能是0~256之间的一个整数，返回值将保存到变量“$?”中。

```powershell
[root@localhost base]$ cat fun3.sh 
#!/bin/bash
function abc() {
    RESULT=`expr $1 \% 2`   #表示取余数
    if [ "$RESULT" != 0 ]; then
        return 0
    else
        return 1
    fi
}
echo "Please enter a number who can devide by 2"
read N
abc $N
case $? in
    0)
        echo "yes ,it is"
        ;;
    1)
        echo "no ,it isn’t"
        ;;
esac

[root@localhost base]$ bash fun3.sh 
Please enter a number who can devide by 2
2
no ,it isn’t
[root@localhost base]$ bash fun3.sh 
Please enter a number who can devide by 2
3
yes ,it is

```

## 5. 函数的载入

如果函数在另外一个文件中，我们该怎么调用它呢？

这里就有一个方法。比如 show 函数写在了function.sh里面了，我们就可以用 source 命令

```powershell
source function.sh
#or
. function.sh
show
```

## 6. 函数利用

如果在函数中使用`exit`命令，可以退出整个脚本，通常情况，函数结束之后会返回调用函数的部分继续执行。

```powershell
可以使用break语句来中断函数的执行。

declare –f 可以显示定义的函数清单

declare –F 可以只显示定义的函数名

unset –f 可以从Shell内存中删除函数

export –f 将函数输出给Shell
```

## 7. 函数变量作用域

默认情况下，变量具有全局作用域，如果想把它设置为局部作用域，可以在其前加入`local`

例如：

```powershell
local a="hello"
```

## 8. 函数的嵌套

```powershell
function first() {
    function second() {
        function third() {
            echo "------this is third"
        }
        echo "this is the second"
        third
    }
    echo "this is the first"
    second
}
 
echo "start..."
first

[root@localhost base]$ bash fun4.sh
start...
this is the first
this is the second
------this is third

```
参考：

 - [Shell入门教程：Shell函数详解](https://www.cnblogs.com/52php/p/5669675.html)
 - [Shell Scripting Tutorial Functions](https://www.shellscript.sh/functions.html)

