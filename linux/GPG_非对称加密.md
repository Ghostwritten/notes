非对称加密/解密文件时，发送方（UserA）以接收方（UserB）的公钥加密文件，接收方以自己的私钥解密
1.接收方UserB创建自己的公钥、私钥对，执行gpg --gen-key操作，根据提示选择并创建密钥：
2.[UserB@pc207 ~]$ gpg --gen-key
1.请选择您要使用的密钥种类：
2.(1) RSA and RSA (default)
3.(2) DSA and Elgamal
4.(3) DSA (仅用于签名)
5.(4) RSA (仅用于签名)
6.您的选择？                                             //直接回车默认(1)
7.RSA 密钥长度应在 1024 位与 4096 位之间。
8.您想要用多大的密钥尺寸？(2048)                             //接受默认2048位
9.您所要求的密钥尺寸是 2048 位
10.请设定这把密钥的有效期限。
11.0 = 密钥永不过期
12.<n> = 密钥在 n 天后过期
13.<n>w = 密钥在 n 周后过期
14.<n>m = 密钥在 n 月后过期
15.<n>y = 密钥在 n 年后过期
16.密钥的有效期限是？(0)                                     //接受默认永不过期
17.密钥永远不会过期
18.以上正确吗？(y/n)y                                         //输入y确
2.接收方UserB导出自己的公钥文件；用户的公钥、私钥信息分别保存在pubring.gpg和secring.gpg文件内

```csharp
[UserB@pc207 ~]$ gpg --list-keys                         //查看公钥环
/home/UserB/.gnupg/pubring.gpg
[UserB@pc207 ~]$ gpg --list-secret-keys
/home/UserB/.gnupg/secring.gpg                         //查看私钥环
```

使用gpg命令结合--import选项将其中的公钥文本导出，传给发送方UserA：

```csharp
[UserB@pc207 ~]$ gpg -a --export UserB > /tmp/UserB.pub
[UserB@pc207 ~]$ ftp 192.168.4.7
Name (192.168.4.7:UserB): ftp
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> cd pub
250 Directory successfully changed.
ftp> lcd /tmp/
Local directory now /tmp
ftp> put UserB.pub                          //通过FTP将公钥传给发送方主机
local: UserB.pub remote: UserB.pub
227 Entering Passive Mode (192,168,4,6,59,39).
150 Ok to send data.
226 Transfer complete.
1719 bytes sent in 0.000127 secs (13535.43 Kbytes/sec)
ftp> quit
221 Goodbye.
```

3.发送方UserA导入接收方的公钥信息
使用gpg命令结合--import选项导入发送方的公钥信息，以便在加密文件时指定接收人来调用对应的公钥。

```csharp
[UserA@svr7 ~]$ gpg --import /var/ftp/pub/UserB.pub
[UserA@svr7 ~]$ echo "I love you ." > tosend.txt
[UserA@svr7 ~]$ gpg -e -r UserB tosend.txt
[UserA@svr7 ~]$ exit
logout
[root@svr7 ~]# cp /home/UserA/tosend.txt.gpg /var/ftp/tosend.txt.gpg
```

4.接收方UserB收取加密文件，以自己的私钥解密文件

```csharp
[UserB@pc207 ~]$ wget ftp://192.168.4.7/tosend.txt.gpg
[UserB@pc207 ~]$ gpg -d tosend.txt.gpg > tosend.txt
[UserB@pc207 ~]$ cat tosend.txt                      //获得解密后的文件内容
I love you .
***************************************
```

7.使用GPG实现软件包的完整性校验，检查软件包签名
1.在pc207上，作者UserB为软件包创建分离式签名，将软件包、签名文件、公钥文件一起发布给其他用户下载，。

```csharp
[UserB@pc207 ~]$ tar zcf tools-1.2.3.tar.gz /etc/hosts      //建立测试软件包
[UserB@pc207 ~]$ gpg -b tools-1.2.3.tar.gz                  //创建分离式数字签名
[UserB@pc207 ~]$ ls -lh tools-1.2.3.tar.gz* UserB.pub
-rw-rw-r--. 1 UserB UserB 170 8月  17 21:18 tools-1.2.3.tar.gz
-rw-rw-r--. 1 UserB UserB 287 8月  17 21:22 tools-1.2.3.tar.gz.sig
-rw-rw-r--. 1 UserB UserB 1.7K 8月  17 21:26 UserB.pub
[root@pc207 ~]# yum -y install vsftpd
[root@pc207 ~]# cp /home/UserB/tools-1.2.3.tar.gz* /var/ftp/
[root@pc207 ~]# cp /home/UserB/UserB.pub /var/ftp/
[root@pc207 ~]# service vsftpd start
```

2.在svr7上，下载软件包并验证官方签名。下载主机pc207发布的UserB的软件包、签名、公钥，导入UserB的公钥后即可验证软件包的完整性。

```csharp
[root@svr7 ~]# wget ftp://192.168.4.207/tools-1.2.3*
[root@svr7 ~]# wget ftp://192.168.4.207/UserB.pub
[root@svr7 ~]# gpg --import UserB.pub                      //导入作者的公钥信息
[root@svr7 ~]# gpg --verify tools-1.2.3.tar.gz.sig tools-1.2.3.tar.gz
```

