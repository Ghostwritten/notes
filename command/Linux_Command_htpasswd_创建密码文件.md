![在这里插入图片描述](https://img-blog.csdnimg.cn/2872c290b8524572b68bfc1fdcae12a2.png)




#  Linux Command htpasswd 创建密码文件
## 1.  简介
[htpasswd](https://httpd.apache.org/docs/2.4/programs/htpasswd.html)是Apache的Web服务器内置的工具,用于创建和更新储存用户名和用户基本认证的密码文件。

## 2. 安装
centos 7、 redhat：
```bash
yum -y install httpd-tools
```
fedora：

```bash
dnf -y install httpd-tools
```
ubuntu:

```bash
apt-get -y install httpd-tools
```
## 3. 语法

```bash
htpasswd (选项) (参数)
```
## 4. 选项
- -c: 创建一个新的密码文件
- -b: 在命令行中一并输入用户名和密码而不是根据提示输入密码
- -D: 删除指定的用户
- -n: 不更新密码文件,只将加密后的用户名密码输出到屏幕上
- -p: 不对密码进行加密,采用明文的方式
- -m: 采用MD5算法对密码进行加密(默认的加密方式)
- -d: 采用CRYPT算法对密码进行加密
- -s: 采用SHA算法对密码进行加密
- -B: 采用bcrypt算法对密码进行加密(非常安全)

## 5. 示例
1. 交互生成用户密码文件

```bash
$ htpasswd -c passwd.txt liming
New password:
Re-type new password:
Adding password for user liming
$ cat passwd.txt
liming:$apr1$57RuOboX$.bnUFsGo5Jdmfkjrv0ijs.
```
2. 以MD5加密方式生成用户密码文件

```bash
$ htpasswd -mc passwd.txt jack
New password:
Re-type new password:
Adding password for user jack
$ cat passwd.txt
jack:$apr1$LNmPUYEc$1pO2CoywBQofLOJlwN6lz1
```

3. 生成 `Bcrypt` Htpasswd 的文件

```bash
htpasswd -bBc /opt/registry/auth/htpasswd registryuser  registryuserpassword
```
文件内容

```bash
$ cat /opt/registry/auth/htpasswd
registryuser:$2y$05$XciI1wfzkUETe7XazJfc/uftBnMQfYOV1jOnbV/QOXw/SXhmLsApK
```

4. 新建一个密码文件`.passwd`并添加一个用户,不提示直接输入用户名密码

```bash
htpasswd -bc .passwd ghostwritten 123456789
```
生成内容
```bash
$ tac .passwd
ghostwritten:$apr1$8RjS08H/$KoaoCrov0U8cwaSkv5vbL1
```

5. 在原有的密码文件`.passwd`下在添加一个用户

```bash
htpasswd -b .passwd spectre 987654321
```
生成内容

```bash
$ cat .passwd
ghostwritten:$apr1$8RjS08H/$KoaoCrov0U8cwaSkv5vbL1
spectre:$apr1$OIA90sdQ$Q5AreNiGrVBmr14sXWTDX0
```
6. 更新用户的密码:有两种方式
- 第一种,直接添加相同的用户名，就会自动区更新密码：

```bash
htpasswd -b .passwd spectre abcdefg
```

```bash
$ cat .passwd
ghostwritten:$apr1$8RjS08H/$KoaoCrov0U8cwaSkv5vbL1
spectre:$apr1$KeukNWZq$V9knxEZazQvvnYQTkhLnW0
```
- 第二种,先删除需要更新密码的用户名,在添加用户：

删除

```bash
htpasswd -D .passwd spectre
```
```bash
$ cat .passwd
ghostwritten:$apr1$8RjS08H/$KoaoCrov0U8cwaSkv5vbL1
```
添加

```bash
htpasswd -b .passwd spectre 111111
```
查看

```bash
$ cat .passwd
ghostwritten:$apr1$8RjS08H/$KoaoCrov0U8cwaSkv5vbL1
spectre:$apr1$cnkYJy8N$W8DvYPDU5zsoMzROAbjif/
```
7. 不更新密码文件，只显示加密后的用户名和密码

```bash
$ htpasswd -bn spectre 222222
spectre:$apr1$Zcs4hc85$04A3bHvqhlzZJFyaqXJiT1
```
## 6. 其他
- `nginx`模块 [http_auth_basic_module](https://nginx.org/en/docs/http/ngx_http_auth_basic_module.html)中的使用,用于生成用户密码文件进行认证。
- 这是一个在线[htpasswd生成器](https://tool.oschina.net/htpasswd)


