

```bash
$ apt install resolvconf
$ resolvconf -u
$ ls /etc/resolvconf/resolv.conf.d/
base head tail
```

`head`提示不可编辑，所以修改`base`和`tail`两个文件
#如果没有这个文件的手动创建

```bash
$ vim /etc/resolvconf/resolv.conf.d/base  
nameserver 8.8.8.8 

$ reboot
$ cat /etc/resolv.conf
...........
nameserver 8.8.8.8 
```


