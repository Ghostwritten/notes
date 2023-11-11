# VMware vCenter Server 7.0.3 安装




[部署完EXSI 7.0.3](https://blog.csdn.net/xixihahalelehehe/article/details/130313339)，你要在EXSI创建一台 windows10 虚拟机，在虚拟机启动 vcenter。
## 1. 安装 vcenter

###  1.1 第一阶段
双击
![在这里插入图片描述](https://img-blog.csdnimg.cn/59fa7655ae464d639d644e8320d2cfdb.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/597d266233ee4241863bc98d8a5d4448.png)


![在这里插入图片描述](https://img-blog.csdnimg.cn/2b7cf8e9de264e908fd6d29baee38159.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/267e4a243fd04449b34e0909181ebc08.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/c7fd4be6790d4ca5aa4033226d8c9b54.png)
- 192.168.21.91
- 443
- root
- 2023bsg.
![在这里插入图片描述](https://img-blog.csdnimg.cn/1d5c796acdb84c57997f24969a36f28d.png)
用户密码：root/2023bsg.
![在这里插入图片描述](https://img-blog.csdnimg.cn/226617e81d744096b490b3ea8e9153f6.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/0ad6f931450745fea9599e448d13a088.png)
- 虚拟机名字：vMware vCenter Server01
- root密码：2023Bsg.

![在这里插入图片描述](https://img-blog.csdnimg.cn/dab429f2323f40038929bb69c336a0ac.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/6f9d09709a0d47558b6e7d44368d5e0e.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/aa817f9e10dc47b38b40bb88875d9af0.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/b6a53d18eebb4f12a38d41874344d284.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/4392f36b834647ddafce1442e029ae41.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/ad3ac009b2124ea6b66ba516bea3732c.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/2a3a08760df04b988ae0e25f29cb2a90.png)
### 1.2  第二阶段
![在这里插入图片描述](https://img-blog.csdnimg.cn/4e9b59e4f6e84874a944e6dee3931d33.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/fef488a6aa94433c9b79c711e1f9bbc6.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/cfb083a10e4d4f579e1819540a61a2db.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/dee54062bac4440b814527cc76cbbdbe.png)
域名：bsg.shanghai
![在这里插入图片描述](https://img-blog.csdnimg.cn/30f09f63fce3478790e98982ad11618d.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/22065c7f69b943c4b7451ddd9b0045d2.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/7e9780e0c8fc4c068deeee9149bde368.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/cf8e1a730a3b4343bb2010d8693cc55a.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/d2f7a95391ac4283821ab6902f798672.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/8029fbfc2c0e431b90c2267265f44f02.png)

下载证书
https://192.168.21.100/certs/download.zip

解压
![在这里插入图片描述](https://img-blog.csdnimg.cn/2903c11fc75842188c7700f2e2de0318.png)
google 浏览器点击“`settting`”，选择 “`privacy and security`”, 再点击“`security`”
![在这里插入图片描述](https://img-blog.csdnimg.cn/a7ba9ea1e351469a9842b499c659829c.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/57275ae9fa7148b6ae0f7062c341dff0.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/b37513051eb649a1b4e467157cdb3a46.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/f49f47614a364349a789d173732006de.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/4ff4fad992e74e228caf077a08e171ee.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/394d8b60d5694e85b5741cfb3d8dda3e.png)
倒入证书成功后，重启浏览器。

访问：`https://192.168.21.100`，小锁上锁成功。


![在这里插入图片描述](https://img-blog.csdnimg.cn/c13b116eae91483db372af98d38ba446.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/1197160055754b7cb982855b2d7b33f0.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/87903c501bef48a989ee371915fa92f2.png)
bsg.shanghai\administrator
2023Bsg.

![在这里插入图片描述](https://img-blog.csdnimg.cn/e3a4f1def51e4d99b30107c0bd57ae08.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/df899393416e4005978b471306f3513d.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/12d21318aa1d4e60955bc9c9796ff0af.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/eb491a2f90df47b5907a25868dd32a30.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/2873043896d54e77b860f946960adb7d.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/0ece280f999c4e3c9215aeaa9604b4fb.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/158a0ade0046484d9adef333b08633a7.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/860a73bfef084b98a614cdbebfce99d6.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/8b2837ddf551484da639aec3077b7d22.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/fdbd667c4e8343ce87ccaf33baefee0f.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/121855e5171445e0881c4d09858f19b8.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/34c52c1162d34c5aad3b7d60fb9d630e.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/23cb80c5a3cc41e3ae20d069320cde5a.png)


## 2. exsi 查看 vcenter


![在这里插入图片描述](https://img-blog.csdnimg.cn/01736867536d44a5a09c18b3b0e4f45c.png)

vcenter 部署结束。
##  3. 部署 DNS server
新建一台 centos 虚拟机做为 DNS server。
###  3.1 安装  unbound
在 CentOS 中，您可以使用 yum 包管理器来安装 unbound：

```bash
sudo yum install unbound
```

安装完成后，您可以使用以下命令启动 unbound 服务，并将其设置为开机启动：

```bash
sudo systemctl start unbound
sudo systemctl enable unbound
```

### 3.2 配置 unbound
一旦 unbound 安装完成，您需要对其进行配置，以便它可以解析您的域名。以下是基本的 unbound 配置示例：
- 打开 unbound 的配置文件 `/etc/unbound/unbound.conf`。

```bash
server:
    interface: 0.0.0.0
    access-control: 0.0.0.0/0 allow
    do-ip4: yes
    do-ip6: no
    do-udp: yes
    do-tcp: yes
    prefetch: yes
    num-threads: 2
    hide-identity: yes
    hide-version: yes
    harden-glue: yes
    harden-dnssec-stripped: yes
    use-caps-for-id: yes

......
# tanzu demo
    local-zone: "bsg.demo." static
    local-data: "tanzu01.bsg.demo A 192.168.21.100"
    local-data-ptr: "192.168.21.100 tanzu01.bsg.demo"
    local-data: "tanzu01-avi.bsg.demo A 192.168.22.5"
    local-data-ptr: "192.168.22.5 tanzu01-avi.bsg.demo"
.....
```

这个配置文件中，您需要修改以下内容：

- `interface`：指定 unbound 监听的 IP 地址。
- `access-control`：指定哪些 IP 地址可以访问 unbound。
- forward-zone：指定您要解析的域名以及使用的权威 DNS 服务器的 IP 地址。
将上面的配置保存到 /etc/unbound/unbound.conf 文件中，并使用以下命令重新加载配置：

```bash
sudo systemctl reload unbound
```

### 3.3 vcenter 配置域名访问

![在这里插入图片描述](https://img-blog.csdnimg.cn/e0f8105f571048a694dba01a75810521.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/c0475d141b73443c9f93dd50d8ccc8f0.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/f28cba9c9e744b3f9b60be9271725617.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/9c68848c15b6488abf217011674bbc6f.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/0642ae27d4834857bae9f9b61806b241.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/6519e2d4d6894dfea4044aa459734c27.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/cd6afe48843d4c4ca7a636fc9bf09b5e.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/f49024f32d5744ebbb4b219665710f41.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/513ae4d41edd4925a2d8560754ac084f.png)

```bash
❯ nslookup tanzu01.bsg.demo 192.168.21.2
Server:         192.168.21.2
Address:        192.168.21.2#53

Name:   tanzu01.bsg.demo
Address: 192.168.21.100
```
测试域名登陆 vcenter

![在这里插入图片描述](https://img-blog.csdnimg.cn/b1599b80ce1c4c54aa3915989706d2c0.png)
登陆
用户：`bsg.shanghai\administrator`
密码： `2023Bsg.`
![在这里插入图片描述](https://img-blog.csdnimg.cn/65ed1b613d85407a800071df4bf8ae6d.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/6c6ec7f7cc804ef385efd20df0b29dba.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/6f7d76acc2e44688a6c2318c85a7c24f.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/45441ebe25ac42e7908e53b4ec09fbbd.png)

