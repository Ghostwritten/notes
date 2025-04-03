# VMware vCenter Server 7.0.3 安装




[部署完EXSI 7.0.3](https://blog.csdn.net/xixihahalelehehe/article/details/130313339)，你要在EXSI创建一台 windows10 虚拟机，在虚拟机启动 vcenter。
## 1. 安装 vcenter

###  1.1 第一阶段
双击
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/10d467123300f495374bc63b78f38930.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e8873f96aed9752bdf1cd4a3a0493758.png)


![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f9af6c21249a7a4e1a06d68891e4a7c2.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ea099997b1a6d31cb55e881c85d94f66.png)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/574b74034c2cf4519d1254070b6b8559.png)
- 192.168.21.91
- 443
- root
- 2023bsg.
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/29409d7d5325adfcf06ed4f71234b33d.png)
用户密码：root/2023bsg.
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/179fa5cabbc6d9b70e4c5d19e4b358e8.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6b70ee739d5b15b4af23b085ab66f6be.png)
- 虚拟机名字：vMware vCenter Server01
- root密码：2023Bsg.

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e4cc6f06b2ce663e510a293133c2263a.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/66b1b6c948d370b77511c9638378373f.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/1d4bfc32a4d7ae3f0ede261a9ad7663d.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d34ac5a49a27018fb4ccceae704a598a.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/2da1775733f771920229fad5d7dc6e99.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/701de9822824ff0a92ac0e7034d47385.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c674792a6eabb8b0ecf3dc57e5a16585.png)
### 1.2  第二阶段
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/7962730a16c238d8fe489b16ff8e9122.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/742aa9d99bbb51ac36b77e8cfe234c2a.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b4cce5bb47d537cfec85ba0758c354c9.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b455caffe26ab230feadc9e60296ad95.png)
域名：bsg.shanghai
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/2c7f74787fbb8f0c4387470e1e4c46df.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/7b3a4669d5ef010bb24041185ee59a91.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/aa0ad23426e6daf659d9bce7afbb4965.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4b9697b0a02acdc0e03eaacc2ff6b69f.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/81bae270cb3666a5129e46cfc99d631b.png)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c639226411cc7b1848214250e52c4e7a.png)

下载证书
https://192.168.21.100/certs/download.zip

解压
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0548487f298ec8ae7b3136956f072cca.png)
google 浏览器点击“`settting`”，选择 “`privacy and security`”, 再点击“`security`”
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/2888c6c2a446889493a4850ea53ed4f6.png)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0cf0d0f3207b72867d2f216ab4aaef5f.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/792f63b9ad2172a418d4841c56fa1cf0.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/386b2dc283b26f859e212aa41efa579c.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e74efa693782d7f61c8b79045b9c5f73.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/05f5951d61cef3ae2bc379dd33b97e70.png)
倒入证书成功后，重启浏览器。

访问：`https://192.168.21.100`，小锁上锁成功。


![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c088e1dbecf013b4a165786eac2c7dcb.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/5eb1753db15e05c06ba1f4edcf67af91.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/cbaaa92edcbe27d50b19702cbc8ab619.png)
bsg.shanghai\administrator
2023Bsg.

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/cdd659d01c3916d201f953c2729e6138.png)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/cac5ec99c018ed55f89f8f4c5cf4e3ab.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/1b12b7acb35595d983292df567ebe295.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/73197f0b190e7df9ccb28f95bcf21ba6.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e76b01081de2c1f0a327e650e50aa5d6.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b2f085b1dbeeb826225d6bac2275cc7d.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c3682e799c6617bd719b9d87a39823de.png)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/adbdb18b993917d4ec2e5e2b33775acc.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e39bc5dfb2125135f99f165ebc7528fa.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/eeb60b36b2593a1dabfca1f6cbf76c0a.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a3b334fb4789bb7b629e770e09e6dced.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/9c365626d5e21a1a287683495f7f25e7.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8bc35cfaf8e729996e375689d8a4b37c.png)


## 2. exsi 查看 vcenter


![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/5cee80e98505f81e0463326c09a0f728.png)

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

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/5ff0b9c75161c7a788430c6188937804.png)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6f6207063491c89083a45077776c7d95.png)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6a1bf2fd8a432ad34e41c626f5e3192a.png)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/19d03abffa4a4275935ff1bdbb40b5a7.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/cbf0405d09d16dff9de0d51dda491de5.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/aea8bc04d005fe5668c41be7743619b8.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/267ca6507f7b76cfc94d8756a3d019ff.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6f6ae4d7b0fa57a70a6ee08dc143fbe7.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/5eae9acae1b1a9cc05fd526583e8b053.png)

```bash
❯ nslookup tanzu01.bsg.demo 192.168.21.2
Server:         192.168.21.2
Address:        192.168.21.2#53

Name:   tanzu01.bsg.demo
Address: 192.168.21.100
```
测试域名登陆 vcenter

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/99cebe3a84d982bbb245f9e6cad919dc.png)
登陆
用户：`bsg.shanghai\administrator`
密码： `2023Bsg.`
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c9a423366308f78f3d6015ec348c314f.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/87909e2374e6783fd2b09c1eaf3b91f0.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/2da31f1791b83a3a973f50355cc15f83.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/858bf199a6d8473349827cc10316a4be.png)

