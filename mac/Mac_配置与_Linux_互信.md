![在这里插入图片描述](https://img-blog.csdnimg.cn/625d4a59799947aa950e0eaa6ad19e56.png)



- [Mac 如何配置国内镜像源](https://blog.csdn.net/xixihahalelehehe/article/details/129151854)
## 1. 安装 sh-copy-id

```bash
sudo curl -L https://raw.githubusercontent.com/beautifulcode/ssh-copy-id-for-OSX/master/install.sh | sh

或者
brew install ssh-copy-id
```

## 2. 生成登录公钥私钥对
```bash
$ ssh-keygen -o -t rsa -C "1zoxun1@gmail.com" -b 4096
Generating public/private rsa key pair.
Enter file in which to save the key (/Users/zongxun/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /Users/zongxun/.ssh/id_rsa
Your public key has been saved in /Users/zongxun/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:ewAVKPAGTO/lI+lE2VwUDThKV2xoSY8JINGqTGDleMY 1zoxun1@gmail.com
The key's randomart image is:
+---[RSA 4096]----+
|o=*+...OO=       |
|.o*+o=O*o .      |
|oo E*=Bo.        |
|..o+.+ .         |
|+   = o S        |
|.. o . . o       |
|    .   . .      |
|         .       |
|                 |
+----[SHA256]-----+
➜  ~ cat /Users/zongxun/.ssh/id_rsa.pub
```
## 3. 拷贝公钥到远程主机
```bash
  ~ cat /Users/zongxun/.ssh/id_rsa.pub
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQCtWnPklKrtkQ9tWm68M+YOLgJBOr4NFI8u4KVGEY+SZS2WjYOjc1A/4at8ccal2xyd0ajH8ytiXgW5q1ngCYgM3QBOoZ6Wenq6FEWwRR9o7Txwf+0d2Y1/goRfYEJ+S940ErrIyb5KCEJFOvnu5DwoJTULolel1+gxYWAyJdhnCCJbB9KHuMTwcVDRpkz4tGQHMgyq8JthpJcUTqeIG/jwmEioVapm9llcOJIFCmaGkO7HOSdH5CczMqT6WndXWx2TtOlUxXggPBqkKi3yqm21XPz1Ent5Lfgl0PNfamGHU6Znm3RViZl8Q8A3yoZlLS8o7ca4P8gT9VIcYwxPzs7sdQvr6wur65YTJFTpE95lUUGZNbCTj/2ZBvX54Yiosd5kQ9U/6zh66LLilagBOx7OtRHex9LIxrRuUWDqBewjvZSERY5/ZylOYQ6KX+xC1GgkOhF2giOqtQxS7Q4ORF9czIdvmSeI0TQKQIsxcIaebiwaI2FLzE+FNDdO8imHV9rPNN+wuu2T9/LkemquQAnhDlGXQzszB/Nvzv0BDjHv+TsbOvFVUdB6VRevlz+c6tSvhJXDY/fluAYnl6yBK12X34PFhUta8K+QVxq/TAOmE9ZJZosp96Pa3abHxW2I7BbVajon94V/dU66d+osbuTXuQ/rjIaWy55nZOzi9QVlkw== 1zoxun1@gmail.com
➜  ~ ssh-copy-id root@192.168.10.29
/opt/homebrew/opt/ssh-copy-id/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/Users/zongxun/.ssh/id_rsa.pub"
/opt/homebrew/opt/ssh-copy-id/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/opt/homebrew/opt/ssh-copy-id/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
root@192.168.10.29's password:

Number of key(s) added:        1

Now try logging into the machine, with:   "ssh 'root@192.168.10.29'"
and check to make sure that only the key(s) you wanted were added.
```
## 4. 访问远程主机
```bash
➜  ~ ssh root@192.168.10.29
Activate the web console with: systemctl enable --now cockpit.socket

Last login: Thu Mar  2 14:56:56 2023 from 172.7.245.44
[root@kind2 ~]#
```

