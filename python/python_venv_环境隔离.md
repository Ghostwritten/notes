


## 1. 介绍
`virtualenv` 用来创建隔离的Python环境。

处理python环境的多版本和模块依赖，以及相应的权限是一个很常见的问题。比如，你有个应用使用的是LibFoo V1.0，但另一个应用却要用到LibFoo V2.0。 如何处理呢？如果把所有模块都安装到 /usr/lib/python2.7/site-packages (或是你本机python默认的模块安装目录)，那你极有可能无意中升级一些不该升级的模块。

更普遍的是，就算你成功安装了某个应用，那么接下来又会怎样？只要它开始运行了，那么只要其所依赖的模块发生任何改动，亦或升级，都可能打断该应用。这还没完，要是你无法在 site-packages 目录下安装模块呢？比如共享主机。上述这几种场合都适用 virtualenv 。它会创建一个拥有独立安装目录的python环境，该隔离环境不会与其他virtualenv环境共享模块（可选择是否访问全局库目录）

## 2. 安装
### 2.1 第一种方式（python>3.4）
如果你正在使用Python3，虚拟环境已经成为内置模块，可以直接通过如下命令来创建它：

```bash
$ python3.8 -m venv [虚拟环境名字]
```
使用这个命令来让Python运行venv包，它会创建一个名为venv的虚拟环境。
**注意**：这个命令不一定能够执行成功，比如译者在Ubuntu16.04环境下执行，提示需要先安装对应的依赖。`sudo apt-get install python3-venv`
### 2.2 第二种方式（python<3.4）
如果你使用的Python版本低于3.4（包括2.7版本），则不会默认支持虚拟环境。 对于这些版本的Python，在创建虚拟环境之前，需要下载并安装称为`virtualenv`的第三方工具。 一旦安装了virtualenv，你可以使用以下命令创建一个虚拟环境：

```bash
$ pip install virtualenv
$ pip install virtualenv==dev #安装最新
$ pip3 install virtualenv
virtualenv [虚拟环境名字]
```
### 2.3 第三种

```bash
$ easy_install virtualenv
```
 [更多方式安装，请点击](https://virtualenv.pypa.io/en/latest/installation.html)
## 3. 进出虚拟环境
```bash
$ source /path/to/[虚拟环境名字]/bin/activate  #进
$ deactivate   #出
```
参考：
- [https://virtualenv-chinese-docs.readthedocs.io/en/latest/#id29](https://virtualenv-chinese-docs.readthedocs.io/en/latest/#id29)
