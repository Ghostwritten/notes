![](https://i-blog.csdnimg.cn/blog_migrate/398189f865b2d6cc413e99a583e3f046.jpeg#pic_center)



## 1. 简介



Pyenv 是一个用于管理和切换多个 Python 版本的工具。它允许开发人员在同一台计算机上同时安装和使用多个不同的 Python 版本，而无需对系统进行全局更改。Pyenv 提供了一种简单的方法来切换 Python 版本，并且对于开发不同项目或在不同环境中使用不同的 Python 版本非常有用。

以下是 Pyenv 的一些主要特点和功能：

- Python 版本管理：Pyenv 允许您方便地安装和管理多个 Python 版本。您可以选择安装官方的 Python 版本，也可以使用其他第三方实现的 Python 版本，如 PyPy。

- 版本切换：通过 Pyenv，您可以轻松地在不同的 Python 版本之间进行切换。这对于测试和验证代码在不同 Python 版本下的运行行为非常有用。

- 虚拟环境支持：Pyenv 可以与 Python 的虚拟环境管理工具（如 virtualenv 和 pyvenv）结合使用，为每个 Python 版本创建独立的虚拟环境。这使得在不同的项目中使用不同的 Python 版本和依赖项变得更加方便。

- 插件系统：Pyenv 提供了一个插件系统，允许用户扩展其功能。例如，有一些插件可用于方便地安装特定的 Python 版本或提供其他附加功能。

- 简单易用：Pyenv 的命令行界面非常简单和直观，使得安装、管理和切换 Python 版本变得容易上手

## 2. 安装
### 2.1 brew 安装  pyenv
```bash
# 更新 brew 保证下载到新版本的 pyenv
$ brew update

# 安装依赖
$ brew install openssl readline sqlite3 xz zlib

# 安装
$ brew install pyenv
# 卸载
$ brew uninstall pyenv
```

### 2.2 脚本安装
 安装依赖
```bash
$ brew install openssl readline sqlite3 xz zlib
```
安装构建依赖项后，您就可以安装pyenv本身了。我建议使用 [pyenv-installer 项目](https://github.com/pyenv/pyenv-installer)：

```bash
curl https://pyenv.run | bash
```
输出：

```bash
 curl https://pyenv.run | bash
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   270  100   270    0     0     70      0  0:00:03  0:00:03 --:--:--    70
Cloning into '/Users/zongxun/.pyenv'...
remote: Enumerating objects: 1185, done.
remote: Counting objects: 100% (1185/1185), done.
remote: Compressing objects: 100% (675/675), done.
remote: Total 1185 (delta 692), reused 657 (delta 377), pack-reused 0
Receiving objects: 100% (1185/1185), 589.16 KiB | 479.00 KiB/s, done.
Resolving deltas: 100% (692/692), done.
Cloning into '/Users/zongxun/.pyenv/plugins/pyenv-doctor'...
remote: Enumerating objects: 11, done.
remote: Counting objects: 100% (11/11), done.
remote: Compressing objects: 100% (9/9), done.
remote: Total 11 (delta 1), reused 5 (delta 0), pack-reused 0
Receiving objects: 100% (11/11), 38.72 KiB | 535.00 KiB/s, done.
Resolving deltas: 100% (1/1), done.
Cloning into '/Users/zongxun/.pyenv/plugins/pyenv-update'...
remote: Enumerating objects: 10, done.
remote: Counting objects: 100% (10/10), done.
remote: Compressing objects: 100% (6/6), done.
remote: Total 10 (delta 1), reused 5 (delta 0), pack-reused 0
Receiving objects: 100% (10/10), done.
Resolving deltas: 100% (1/1), done.
Cloning into '/Users/zongxun/.pyenv/plugins/pyenv-virtualenv'...
remote: Enumerating objects: 63, done.
remote: Counting objects: 100% (63/63), done.
remote: Compressing objects: 100% (56/56), done.
remote: Total 63 (delta 11), reused 27 (delta 0), pack-reused 0
Receiving objects: 100% (63/63), 40.25 KiB | 502.00 KiB/s, done.
Resolving deltas: 100% (11/11), done.

WARNING: seems you still have not added 'pyenv' to the load path.

# Load pyenv automatically by appending
# the following to
# ~/.bash_profile if it exists, otherwise ~/.profile (for login shells)
# and ~/.bashrc (for interactive shells) :

export PYENV_ROOT="$HOME/.pyenv"
[[ -d $PYENV_ROOT/bin ]] && export PATH="$PYENV_ROOT/bin:$PATH"
eval "$(pyenv init -)"

# Restart your shell for the changes to take effect.

# Load pyenv-virtualenv automatically by adding
# the following to ~/.bashrc:

eval "$(pyenv virtualenv-init -)"

```

这将安装pyenv以及一些有用的插件：

- pyenv：实际的pyenv应用程序
- pyenv-virtualenv：pyenv 和虚拟环境的插件
- pyenv-update: 更新插件pyenv
- pyenv-doctor：用于验证是否安装了pyenv和构建依赖项的插件
- pyenv-which-ext：自动查找系统命令的插件

> 注意：上述命令与下载[pyenv-installer脚本](https://github.com/pyenv/pyenv-installer/blob/master/bin/pyenv-installer)并运行它的效果相同本地。因此，如果您想确切地了解正在运行的内容，可以自行查看该文件。或者，如果您确实不想运行脚本，可以查看[手动安装说明](https://github.com/pyenv/pyenv#basic-github-checkout)。

~/.zshrc 添加以上内容：

```bash
$ vim ~/.zshrc 
export PYENV_ROOT="$HOME/.pyenv"
[[ -d $PYENV_ROOT/bin ]] && export PATH="$PYENV_ROOT/bin:$PATH"
eval "$(pyenv init -)"
```

然后

```bash
exec "$SHELL" # Or just restart your terminal
```



## 3. pyenv 安装 Python

查看可安装列表

```bash
pyenv install --list 
```

可用的CPython 3.6 到 3.8

```bash
pyenv install --list | grep " 3\.[678]"
```
查看所有 Jython 版本

```bash
$ pyenv install --list | grep "jython"
  jython-dev
  jython-2.5.0
  jython-2.5-dev
  jython-2.5.1
  jython-2.5.2
  jython-2.5.3
  jython-2.5.4-rc1
  jython-2.7.0
  jython-2.7.1
  jython-2.7.2
```

安装 3.12.1

```bash
pyenv install 3.12.1
```
安装 3.12.1并显示安装过程

```bash
pyenv install -v 3.12.1
```
最后正常输出如下显示安装成功：

```bash
......
Successfully installed pip-23.2.1
/var/folders/0_/3h6j4mkd2156rcrkqzj3b9600000gn/T/python-build.20231218163320.47316 ~
~
Installed Python-3.12.1 to /Users/zongxun/.pyenv/versions/3.12.1
```


在终端安装也许会卡住，例如：

```bash
$ pyenv install -v 3.12.1
python-build: use openssl@3 from homebrew
python-build: use readline from homebrew
/var/folders/0_/3h6j4mkd2156rcrkqzj3b9600000gn/T/python-build.20231218161915.46244 ~
Downloading Python-3.12.1.tar.xz...
-> https://www.python.org/ftp/python/3.12.1/Python-3.12.1.tar.xz
```
可以先浏览器下载下来，再将包放到安装的默认位置(`~/.pyenv/versions/`),再执行安装`pyenv install -v 3.12.1`

已安装的位置
```bash
$ ls ~/.pyenv/versions/
3.12.1
```

## 4. 卸载 python

您的所有版本都将位于此处。这很方便，因为删除这些版本很简单：

```bash
$ rm -rf ~/.pyenv/versions/3.12.1
```

当然pyenv还提供了卸载特定Python版本的命令：

```bash
$ pyenv uninstall 3.12.1
```


## 5. 管理 python

检查您有哪些可用的 Python 版本：

```bash
$ pyenv versions
* system (set by /Users/zongxun/.pyenv/version)
  3.10.13
  3.12.1
```
* 表示 system Python 版本当前处于活动状态。您还会注意到，这是由根 pyenv 目录中的文件设置的。这意味着，默认情况下，您仍在使用系统 Python：

```bash
$ python3 -V
Python 3.9.6
$  which python3
/Users/zongxun/.pyenv/shims/python3
```

如果您想使用版本 `3.10.13`，则可以使用`global` 命令：

```bash
$ pyenv global 3.10.13
$ python -V
Python 3.10.13
```
python 版本从 3.9.6 切换为 3.10.13了。

测试命令
```bash
python -m test
```

这将启动大量内部 Python 测试来验证您的安装。您可以轻松地观看测试通过。

如果您想返回到默认的 Python 系统版本，您可以运行以下命令：

```bash
$ pyenv global system
$ python -V
Python 3.9.6
```

参考：
- [Python 官网](https://www.python.org/?spm=a2c6h.12873639.article-detail.7.67222d7eLnmPbc)
-  [官方 pyenv Github](https://github.com/pyenv/pyenv) 下载
- [https://realpython.com/intro-to-pyenv/](https://realpython.com/intro-to-pyenv/)


