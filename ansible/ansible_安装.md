
![在这里插入图片描述](https://img-blog.csdnimg.cn/a10a572dbdd34ff2aac547d582c2412b.png)




## 1. 简介
[Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html) 是简单的开源 IT 引擎，可自动执行应用程序部署、内部服务编排、云配置和许多其他 IT 工具。

Ansible 易于部署，因为它不使用任何代理或自定义安全基础设施。

Ansible 使用 playbook 来描述自动化作业，而 playbook 使用非常简单的语言即YAML（这是一种人类可读的数据序列化语言，通常用于配置文件，但可以用于存储数据的许多应用程序）这非常容易供人类理解、阅读和书写。因此，优势在于即使是 IT 基础设施支持人员也可以阅读和理解剧本，并在需要时进行调试（YAML——它是人类可读的形式）。

Ansible 专为多层部署而设计。Ansible 不会一次管理一个系统，它通过描述所有相互关联的系统来建模 IT 基础设施。Ansible 是完全无代理的，这意味着 Ansible 通过 ssh（默认情况下）连接您的节点来工作。但是，如果您想要其他连接方法（如 Kerberos），Ansible 会为您提供该选项。

连接到您的节点后，Ansible 会推送称为“Ansible 模块”的小程序。Ansible 在您的节点上运行该模块，并在完成后将其删除。Ansible 在简单的文本文件中管理您的库存（这些是主机文件）。Ansible 使用 hosts 文件，可以在其中对主机进行分组，并可以控制 playbook 中特定组的操作。

## 2. 安装

### 2.1 yum 安装

以下 `centos` 或 `redhat` 安装

配置 yum 源：

```bash
cat <<eof>>/etc/yum.repos.d/my.repo
[epel]
name=epel
baseurl=http://mirrors.aliyun.com/epel/7Server/x86_64/
enable=1
gpgcheck=0
eof
```

```bash
yum list ansible --showduplicates | sort -r
yum -y install ansible
```

### 2.2 Python 安装
ansible 依赖 `Python 2.7`，或 `Python 3.5 - 3.11` 版本。

- [ansible 官方安装教程](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html)
- [编译安装 python 3.xx](https://blog.csdn.net/xixihahalelehehe/article/details/122587523)

编译安装 python 3.xx
```bash
yum -y install gcc openssl-devel gcc-c++ compat-gcc-34 compat-gcc-34-c++ libffi-devel
wget https://www.python.org/ftp/python/3.10.0/Python-3.10.0.tgz
tar zxf Python-3.10.0.tgz
cd Python-3.10.0
./configure --enable-optimizations --with-ssl --prefix=/usr/local/python-3.10.0
make && make install
```
声明命令路径
```bash
$ vim /etc/profile.d/python.sh
export PATH=/usr/local/python-3.10.0/bin:$PATH
alias py='/usr/local/python-3.10.0/bin/python3.10'
$ source /etc/profile.d/python.sh
```
建立软连接
```bash
ln -s /usr/local/python-3.10.0/bin/python3.10 /usr/bin/python3.10
ln -s /usr/local/python-3.10.0/bin/pip3.10 /usr/bin/pip3.10
```
或者通过 yum 安装 python3

```bash
sudo yum -y install python3 python3-pip
sudo pip3 install ansible
```

测试安装是否成功
```bash
 python3.10 -V
Python 3.10.0

$ pip3.10 -V
pip 21.2.3 from /usr/local/python-3.10.0/lib/python3.10/site-packages/pip (python 3.10)

$ python3.10 -m pip -V
pip 21.2.3 from /usr/local/python-3.10.0/lib/python3.10/site-packages/pip (python 3.10)

# 安装 ansible
$ python3.10 -m pip install --user ansible
Collecting ansible
  Using cached ansible-7.1.0-py3-none-any.whl (42.4 MB)
Collecting ansible-core~=2.14.1
  Using cached ansible_core-2.14.1-py3-none-any.whl (2.2 MB)
Collecting PyYAML>=5.1
  Downloading PyYAML-6.0-cp310-cp310-manylinux_2_5_x86_64.manylinux1_x86_64.manylinux_2_12_x86_64.manylinux2010_x86_64.whl (682 kB)
     |████████████████████████████████| 682 kB 563 kB/s
Collecting packaging
  Downloading packaging-23.0-py3-none-any.whl (42 kB)
     |████████████████████████████████| 42 kB 1.8 MB/s
Collecting resolvelib<0.9.0,>=0.5.3
  Downloading resolvelib-0.8.1-py2.py3-none-any.whl (16 kB)
Collecting cryptography
  Downloading cryptography-39.0.0-cp36-abi3-manylinux_2_28_x86_64.whl (4.2 MB)
     |████████████████████████████████| 4.2 MB 16.2 MB/s
Collecting jinja2>=3.0.0
  Using cached Jinja2-3.1.2-py3-none-any.whl (133 kB)
Collecting MarkupSafe>=2.0
  Downloading MarkupSafe-2.1.1-cp310-cp310-manylinux_2_17_x86_64.manylinux2014_x86_64.whl (25 kB)
Collecting cffi>=1.12
  Downloading cffi-1.15.1-cp310-cp310-manylinux_2_17_x86_64.manylinux2014_x86_64.whl (441 kB)
     |████████████████████████████████| 441 kB 14.4 MB/s
Collecting pycparser
  Downloading pycparser-2.21-py2.py3-none-any.whl (118 kB)
     |████████████████████████████████| 118 kB 15.6 MB/s
Installing collected packages: pycparser, MarkupSafe, cffi, resolvelib, PyYAML, packaging, jinja2, cryptography, ansible-core, ansible
  WARNING: The scripts ansible, ansible-config, ansible-connection, ansible-console, ansible-doc, ansible-galaxy, ansible-inventory, ansible-playbook, ansible-pull and ansible-vault are installed in '/root/.local/bin' which is not on PATH.
  Consider adding this directory to PATH or, if you prefer to suppress this warning, use --no-warn-script-location.
  WARNING: The script ansible-community is installed in '/root/.local/bin' which is not on PATH.
  Consider adding this directory to PATH or, if you prefer to suppress this warning, use --no-warn-script-location.
Successfully installed MarkupSafe-2.1.1 PyYAML-6.0 ansible-7.1.0 ansible-core-2.14.1 cffi-1.15.1 cryptography-39.0.0 jinja2-3.1.2 packaging-23.0 pycparser-2.21 resolvelib-0.8.1
WARNING: Running pip as the 'root' user can result in broken permissions and conflicting behaviour with the system package manager. It is recommended to use a virtual environment instead: https://pip.pypa.io/warnings/venv
WARNING: You are using pip version 21.2.3; however, version 22.3.1 is available.
You should consider upgrading via the '/usr/local/python-3.10.0/bin/python3.10 -m pip install --upgrade pip' command.

# 查看 ansible 版本第一种方式
$ python3.10 -m pip show ansible
Name: ansible
Version: 7.1.0
Summary: Radically simple IT automation
Home-page: https://ansible.com/
Author: Ansible, Inc.
Author-email: info@ansible.com
License: GPLv3+
Location: /root/.local/lib/python3.10/site-packages
Requires: ansible-core
Required-by:

# # 查看 ansible 版本第二种方式
$ ansible --version
ansible [core 2.14.1]
  config file = None
  configured module search path = ['/root/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /root/.local/lib/python3.10/site-packages/ansible
  ansible collection location = /root/.ansible/collections:/usr/share/ansible/collections
  executable location = /root/.local/bin/ansible
  python version = 3.10.0 (default, Jan 11 2023, 17:07:52) [GCC 8.5.0 20210514 (Red Hat 8.5.0-4)] (/usr/local/python-3.10.0/bin/python3.10)
  jinja version = 3.1.2
  libyaml = True
```



## 3. 帮助

```bash
$ ansible-doc -h
Usage: ansible-doc [options] [module...]
Options:
 -a, --all Show documentation for all modules
 -h, --help show this help message and exit
 -l, --list List available modules
 -M MODULE_PATH, --module-path=MODULE_PATH
 specify path(s) to module library (default=None)
 -s, --snippet Show playbook snippet for specified module(s)
 -v, --verbose verbose mode (-vvv for more, -vvvv to enable
 connection debugging)
 --version show program's version number and exit
```
其中"-l"选项⽤于列出ansible的模块，通常结合grep来筛选。例如找出和yum相关的可⽤模块。

```bash
$ ansible-doc -l | grep yum
yum Manages packages with the `yum' package manager
yum_repository Add or remove YUM repositories
```
再使⽤"-s"选项可以获取指定模块的使⽤帮助。例如，获取yum模块的使⽤语法。

```bash
ansible-doc -s yum
- name: Manages packages with the `yum' package manager
 action: yum
 conf_file # The remote yum configuration file to use for the transaction.
 disable_gpg_check # Whether to disable the GPG checking of signatures of packages being
 installed. Has an effect only if state is `present' or `latest'.
 disablerepo # `Repoid' of repositories to disable for the install/update operation.
 These repos will not persist beyond the transaction. When specifying
 multiple repos, separate them with a ",".
 enablerepo # `Repoid' of repositories to enable for the install/update operation.
 These repos will not persist beyond the transaction. When specifying
 multiple repos, separate them with a ",".
 exclude # Package name(s) to exclude when state=present, or latest
 installroot # Specifies an alternative installroot, relative to which all packages
 will be installed.
 list # Package name to run the equivalent of yum list <package> against.
 name= # Package name, or package specifier with version, like `name-1.0'.
 When using state=latest, this can be '*' which means run: yum -y update.
 You can also pass a url or a local path to a rpm file(using state=present).
 To operate on several packages this can accept a comma separated list of
 packages or (as of 2.0) a list of packages.
 skip_broken # Resolve depsolve problems by removing packages that are causing problems
 from the trans- action.
 state # Whether to install (`present' or `installed', `latest'), or remove
 (`absent' or `removed') a package.
 update_cache # Force yum to check if cache is out of date and redownload if needed. Has
 an effect only if state is `present' or `latest'.
 validate_certs # This only applies if using a https url as the source of the rpm.
 e.g. for localinstall. If set to `no', the SSL certificates will not be
 validated. This should only set to `no' used on personally controlled
 sites using self-signed certificates as it avoids verifying the source
 site. Prior to 2.1 the code worked as if this was set to `yes'.
```

