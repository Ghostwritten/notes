

包地址：[https://nodejs.org/en/blog/release/v14.17.6/](https://nodejs.org/en/blog/release/v14.17.6/)

----------
## 更新node版本

```bash
#第一步：清除npm缓存，执行命令：
$ npm cache clean -f 
#第二步：安装n模块：
$ npm install -g n
#n模块是专门用来管理nodejs版本d

#第三步：升级node.js到最新稳定版：
$ n stable 
#查看node版本和node安装路径
#查看node版本
#或者升级到最新版
$ n latest

$ node -v
v8.9.0
#查看node安装路径

$ which node
/usr/local/bin/nod
```


## 本地安装node tar包

```bash
tar -xvf node-v14.17.6-linux-x64.tar.xz


cat >>/etc/profile<<EOF
export PATH=$PATH:/opt/node-v14.17.6-linux-x64/bin
EOF


source /etc/profile
```
##  卸载node
centos
```bash
yum remove nodejs npm -y
```
ubuntu
```bash
 apt remove npm
 apt remove nodejs
```

## 更新npm

```bash
npm install npm@latest -g
```
## 卸载npm

```bash
npm uninstall npm -g
```


##  npm配置taobao源
源地址：https://registry.npm.taobao.org

```bash
npm config set registry https://registry.npm.taobao.org

#查看express包的配置信息是否生效
npm info express
```


