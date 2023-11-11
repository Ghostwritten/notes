#  python pip 安装


---


## 1. 官网安装：

```bash
$ curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
$ python get-pip.py
$ easy_install 
$ wget https://pypi.python.org/pypi/ez_setup
$ cd ez_setup
$ python  ez_setup.py
#安装docker-py模块
$ pip install docker -i http://pypi.douban.com/simple --trusted-host pypi.douban.com
```

## 2. 本地安装

```bash
$ wget https://pypi.python.org/packages/source/p/pip/pip-1.5.4.tar.gz
$ tar -xzvf pip-1.5.4.tar.gz
$ cd pip-1.5.4
$ python setup.py install
```


## 3. centos7下使用yum安装pip

首先安装epel扩展源：

```bash
yum -y install epel-release
```

更新完成之后，就可安装pip：

```bash
yum -y install python-pip
pip -V 
```
pip3安装
```bash
yum -y install python3-pip
pip -V 
```
如果pip install 出现问题可以试试命令

```bash
pip install setuptools==39.2.0
```

**************************************************************

