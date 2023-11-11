

---

参考链接：
[https://www.runoob.com/nodejs/nodejs-install-setup.html](https://www.runoob.com/nodejs/nodejs-install-setup.html)

---
## nodejs tar包安装
选择[nodejs](https://nodejs.org/en/download//)
```bash
cd /usr/local
wget https://nodejs.org/dist/v17.4.0/node-v17.4.0-linux-x64.tar.gz
tar -zxvf node-v17.4.0-linux-x64.tar.gz
ln -s /usr/local/node-v17.4.0-linux-x64/bin/node /usr/local/bin/
ln -s /usr/local/node-v17.4.0-linux-x64/bin/npm /usr/local/bin/
node -v
v17.4.0
npm -v
8.3.1

```

##  nodejs 源码编译安装

> 条件：必须安装 python 3.6以上，[python 3.10.0源码编译安装](https://ghostwritten.blog.csdn.net/article/details/122587523)

###  1. 安装编译依赖工具

```bash
$ yum install gcc openssl-devel gcc-c++ compat-gcc-34 compat-gcc-34-c++
```
###  2. 下载node版本
[版本列表](https://nodejs.org/dist/)

```bash
wget https://nodejs.org/dist/v17.4.0/node-v17.4.0.tar.gz
tar -zxvf node-v17.4.0.tar.gz
cd node-v17.4.0
```
### 3. 安装

```bash
./configure  --enable-optimizations --prefix=/usr/local/node
make && make install

echo export PATH=/usr/local/node/bin:$PATH >> /etc/profile
```


>注意：
> [“Can‘t connect to HTTPS URL because the SSL module is not available.“](https://www.codeleading.com/article/69475219011/)



