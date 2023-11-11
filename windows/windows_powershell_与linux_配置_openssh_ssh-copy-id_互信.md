

![在这里插入图片描述](https://img-blog.csdnimg.cn/ef4ae4ad5af9431399fd7b637ba8489b.png)


##  第一步

```bash
PS C:\Users\Christopher> ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (C:\Users\Christopher/.ssh/id_rsa):
Created directory 'C:\Users\Christopher/.ssh'.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in C:\Users\Christopher/.ssh/id_rsa.
Your public key has been saved in C:\Users\Christopher/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:/mjkrJOQbRzCAwlSPYVBNcuxntm/Ms5/MMC15dCRrMc christopher@Christopher-Win10-VM-01
The key's randomart image is:
+---[RSA 2048]----+
|oo.+o==    o.o   |
|. o +. =  o =    |
|   o .+. . B     |
|    +..+o o E    |
|     *+.S. .     |
|    o +...o      |
|     o =. .o     |
|      o.*o ..    |
|      .=+++.     |
+----[SHA256]-----+
PS C:\Users\Christopher>
Copy SSH Key to Remote L
```
##  第二步
格式：
```bash
type $env:USERPROFILE\.ssh\id_rsa.pub | ssh root@192.168.30.31 "cat >> .ssh/authorized_keys"
```
示例
```bash
PS C:\Users\Christopher> type $env:USERPROFILE\.ssh\id_rsa.pub | ssh root@192.168.30.31 "cat >> .ssh/authorized_keys"
The authenticity of host '192.168.30.31 (192.168.30.31)' can't be established.
ECDSA key fingerprint is SHA256:mTD0/WNCVZ/p/PFSkNDmLJtzIGb5eD7qj6erOQkomjM.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.30.31' (ECDSA) to the list of known hosts.
christopher@192.168.30.31's password:
PS C:\Users\Christopher>
```
##  第三步

```bash
PS C:\Users\Christopher> ssh 192.168.30.31
Last login: Sat May 23 12:44:51 2020 from 192.168.10.139
[christopher@linux ~]$ who
```

