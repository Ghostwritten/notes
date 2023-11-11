#  python 多种运行脚本方式



您将有机会通过以下方式学习如何运行 Python 脚本：

 - 操作系统命令行或终端
 - Python 交互模式
 - 您最喜欢的 IDE 或文本编辑器
 - 系统的文件管理器，通过双击脚本的图标


##  1. 脚本与模块
在计算中，脚本一词用于指代包含订单逻辑序列的文件或批处理文件。这通常是一个简单的程序，存储在纯文本文件中。

脚本总是由某种解释器处理，它负责按顺序执行每个命令。

包含打算由用户直接执行的 Python 代码的纯文本文件通常称为脚本，这是一个非正式术语，表示顶级程序文件。

另一方面，一个包含 Python 代码的纯文本文件被称为模块。

因此，**模块**和**脚本**之间的主要区别在于**模块**是用来导入的，而**脚本**是用来直接执行的。

##  2. 什么是 Python 解释器？
Python 也是一种称为解释器的软件。解释器是运行 Python 代码和脚本所需的程序。从技术上讲，解释器是在您的程序和计算机硬件之间工作以使您的代码运行的软件层。

根据您使用的 Python 实现，解释器可以是：

 - 用C编写的程序，例如CPython，它是该语言的核心实现
 - 用Java编写的程序，例如Jython
 - 用 Python 本身编写的程序，例如PyPy
 - 在 .NET 中实现的程序，例如IronPython

无论解释器采用何种形式，您编写的代码将始终由该程序运行。因此，能够运行 Python 脚本的第一个条件是在您的系统上正确安装了解释器。

解释器能够以两种不同的方式运行 Python 代码：

 - 作为脚本或模块
 - 作为输入到交互式会话中的一段代码


##  3. 如何以交互方式运行 Python 代码

```bash
$ python3
Python 3.6.7 (default, Oct 22 2018, 11:32:17)
[GCC 8.2.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>>
>>> print('Hello World!')
Hello World!
>>> 2 + 5
7
>>> print('Welcome to Real Python!')
Welcome to Real Python!
```
要退出交互模式，您可以使用以下选项之一：

 - quit()或exit()，它们是内置函数
 - Windows 上的`Ctrl+Z`和组合键，或类 Unix 系统上的+组合键`EnterCtrlD`


##  4. 解释器如何运行 Python 脚本？

 - 按顺序处理脚本的语句
 - 将源代码编译为称为字节码的中间格式
此字节码是将代码翻译成与平台无关的低级语言。其目的是优化代码执行。所以，下次解释器运行你的代码时，它会绕过这个编译步骤。严格来说，这个代码优化只针对模块（导入文件），不针对可执行脚本。
 - 发送代码以执行
此时，称为 Python 虚拟机 (PVM) 的东西开始发挥作用。PVM 是 Python 的运行时引擎。这是一个循环遍历字节码指令以逐个运行它们的循环。PVM 不是 Python 的独立组件。它只是您在机器上安装的 Python 系统的一部分。从技术上讲，PVM 是所谓的 Python 解释器的最后一步。运行 Python 脚本的整个过程称为Python 执行模型。


##  5. 如何使用命令行运行 Python 脚本
Python 交互式会话将允许您编写很多代码行，但是一旦关闭会话，您将丢失您编写的所有内容。这就是为什么编写 Python 程序的常用方法是使用纯文本文件。按照惯例，这些文件将使用.py扩展名。（在 Windows 系统上，扩展名也可以是.pyw.）

可以使用任何纯文本编辑器创建 Python 代码文件。如果你是 Python 编程新手，可以试试[Sublime Text](https://realpython.com/products/sublime-python/)，它是一个功能强大且易于使用的编辑器，但你可以使用任何你喜欢的编辑器。

```bash
#!/usr/bin/env python3

print('Hello World!')
```
将该文件保存在您的工作目录中，并命名为`hello.py`. 准备好测试脚本后，您可以继续阅读。

```bash
$ python3 hello.py
Hello World!
```
重定向输出

```bash
$ python3 hello.py > output.txt
```
此操作将脚本的输出重定向到output.txt，而不是标准系统输出 ( stdout)。该过程通常称为流重定向，可在 Windows 和类 Unix 系统上使用。

如果output.txt不存在，则自动创建。另一方面，如果文件已经存在，那么它的内容将被新的输出替换。

最后，如果要将连续执行的输出添加到 的末尾output.txt，则必须使用两个尖括号 ( >>) 而不是一个，就像这样：

```bash
$ python3 hello.py >> output.txt
```

###  6. -m 使用选项运行模块
该`-m`选项搜索`sys.path`模块名称并将其内容运行为`__main__`：

```bash
$ python3 -m hello
Hello World!
```
###  7. 使用脚本文件名
在最新版本的 Windows 上，只需在命令提示符处输入包含代码的文件名即可运行 Python 脚本：

```bash
C:\devspace> hello.py
Hello World!
```
在类 Unix 系统上，例如 GNU/Linux，你可以实现类似的东西。您只需在第一行添加文本`#!/usr/bin/env python`，就像您对hello.py.

对于 Python，这是一个简单的注释，但对于操作系统，这一行表示必须使用什么程序来运行文件。

这一行以#!字符组合开始，通常称为hash bang或shebang，然后是解释器的路径。

有两种方法可以指定解释器的路径：

 - `#!/usr/bin/python`:写绝对路径
 - `#!/usr/bin/env python`：使用操作系统env命令，通过搜索PATH环境变量定位并执行Python

```bash
$ # Assign execution permissions
$ chmod +x hello.py
$ # Run the script by using its filename
$ ./hello.py
Hello World!
```

##  8. 如何以交互方式运行 Python 脚本

```bash
>>> import hello
Hello World!

>>> import sys
>>> for path in sys.path:
...     print(path)
...
/usr/lib/python36.zip
/usr/lib/python3.6
/usr/lib/python3.6/lib-dynload
/usr/local/lib/python3.6/dist-packages
/usr/lib/python3/dist-packages

```

##  9. 使用importlib和imp
使用`import_module()`，您可以模拟import操作，因此可以执行任何模块或脚本。看看这个例子：

```c
>>> import importlib
>>> importlib.import_module('hello')
Hello World!
<module 'hello' from '/home/username/hello.py'>
```
首次导入模块后，您将无法继续使用import它来运行它。在这种情况下，您可以使用`importlib.reload()`，这将强制解释器再次重新导入模块，就像在以下代码中一样：

```bash
>>> import hello  # First import
Hello World!
>>> import hello  # Second import, which does nothing
>>> import importlib
>>> importlib.reload(hello)
Hello World!
<module 'hello' from '/home/username/hello.py'>
```
这里要注意的重要一点是，参数reload()必须是模块对象的名称，而不是字符串：

```bash
>>> importlib.reload('hello')
Traceback (most recent call last):
    ...
TypeError: reload() argument must be a module
```
如果使用字符串作为参数，那么reload()将引发TypeError异常。

`importlib.reload()`当您修改模块并想要测试您的更改是否有效时会派上用场，而无需离开当前的交互式会话。

最后，如果您使用的是 `Python 2.x`，那么您将拥有imp一个模块，它提供了一个名为`reload(). imp.reload()`工作原理与`importlib.reload()`. 这是一个例子：

```bash
>>> import hello  # First import
Hello World!
>>> import hello  # Second import, which does nothing
>>> import imp
>>> imp.reload(hello)
Hello World!
<module 'hello' from '/home/username/hello.py'>
```
在 `Python 2.x` 中，reload()是一个内置函数。在 `2.6` 和 `2.7` 版本中，它也包含在`imp`, 以帮助过渡到 3.x。

##  10. 使用runpy.run_module()和runpy.run_path()

```python
>>> runpy.run_module(mod_name='hello')
Hello World!
{'__name__': 'hello',
    ...
'_': None}}
```
该模块使用标准import机制定位，然后在新的模块命名空间上执行。

的第一个参数`run_module()`必须是带有模块绝对名称的字符串（不带.py扩展名）。

另一方面，`runpy`还提供了`run_path()`，它允许您通过提供模块在文件系统中的位置来运行模块：

```python
>>> import runpy
>>> runpy.run_path(path_name='hello.py')
Hello World!
{'__name__': '<run_path>',
    ...
'_': None}}
```
Like run_module()，run_path()返回globals执行模块的字典。

参数必须是字符串，path_name可以引用如下：

Python 源文件的位置
已编译字节码文件的位置
中的有效条目的值sys.path，包含`__main__`模块（`__main__.py`文件）


##  11. 黑客攻击exec()

```python
>>> exec(open('hello.py').read())
'Hello World!'
```
## 12. 使用execfile()（仅限 Python 2.x）

```python
>>> execfile('hello.py')
Hello World!
```
##  13. 如何从 IDE 或文本编辑器运行 Python 脚本
在开发更大更复杂的应用程序时，建议您使用集成开发环境 (IDE) 或高级文本编辑器。

这些程序中的大多数都提供了从环境本身内部运行脚本的可能性。它们通常包含运行或构建命令，通常可从工具栏或主菜单中获得。

Python 的标准发行版包含IDLE作为默认 IDE，您可以使用它来编写、调试、修改和运行您的模块和脚本。

Eclipse-PyDev、PyCharm、Eric 和 NetBeans 等其他 IDE 也允许您从环境内部运行 Python 脚本。

[Sublime Text](https://www.sublimetext.com/)和[Visual Studio Code](https://code.visualstudio.com/docs)等高级文本编辑器也允许您运行脚本。


参考：

 - [How to Run Your Python Scripts](https://realpython.com/run-python-scripts/)

