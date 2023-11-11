#  python pip 解决下载速度慢



----
## 1. 国内源：
新版ubuntu要求使用https源，要注意。

```bash
清华：https://pypi.tuna.tsinghua.edu.cn/simple

阿里云：http://mirrors.aliyun.com/pypi/simple/

中国科技大学 https://pypi.mirrors.ustc.edu.cn/simple/

华中理工大学：http://pypi.hustunique.com/

山东理工大学：http://pypi.sdutlinux.org/ 

豆瓣：http://pypi.douban.com/simple/
```
## 2. 临时使用
语法：

```bash
pip install -i [源] [package]
```

示例：

```bash
pip install -i http://mirrors.aliyun.com/pypi/simple/ scapy
```
## 3. 永久生效
### 3.1. linux

```bash
$ vim ~/.pip/pip.conf 
[global]
index-url = https://pypi.tuna.tsinghua.edu.cn/simple
[install]
trusted-host=mirrors.aliyun.com
```

```bash
$ pip install scapy
```
### 3.2. Windows：

在 windows 命令提示符（控制台）中，输入 `%APPDATA%`，进入此目录
在该目录下新建名为 pip 的文件夹，然后在其中新建文件 pip.ini。
`"C:\Users\Administrator\AppData\Roaming\pip\pip.ini`"）

```bash
[global]
index-url = http://pypi.douban.com/simple
[install]
trusted-host=pypi.douban.com
```

