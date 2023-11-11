#  Linux Command ssh-keyscan
tags: 远程



----
## 1. 简介
工具用来收集一组主机的 ssh 主机公钥 (host key), 设计目的是帮助建立和验证 `ssh_known_hosts` 文件.提供了一个小巧的接口让 shell 和 perl 文件使用.

使用了非阻塞 socket I/O 函数, 尽可能多的并行访问多个主机, 因此它的效率很高. 它可以在数十秒内采集某域中 1,000 台主机的密钥, 即使某些主机离线或不使用 ssh. 扫描的时候无须登录目标主机, 也不涉及任何加密操作.

## 2. 格式

```c
ssh-keyscan -words [-v46 ] [-p port ] [-T timeout ] [-t type ] [-f file ] [host | addrlist namelist ] [... ]
```
## 3. 参数

```bash
-4：强制使用IPv4地址；
-6：强制使用IPv6地址；
-f：从指定文件中读取“地址列表/名字列表”；
-p：指定连接远程主机的端口；
-T：指定连接尝试的超时时间；
-t：指定要创建的密钥类型；
-v：信息模式，打印调试信息。
```
## 4. 安全

如果通过建立了 ssh_known_hosts 文件, 但却没有验证里面的公钥, 用户很容易遭到中间人 攻击. 而另一方面, 如果安全模型允许这个风险, 创建 ssh_known_hosts 文件后,能够帮助检测已经发起的密钥篡改或中间人攻击.

## 5. 示例
显示 hostname 的 rsa1 主机密钥:

```bash
$ ssh-keyscan hostname
k8s2-master ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCtp4mHchzEKG17pseHL3sIruTTz3mdVUOdkSxjKecozFmCXBE6+gckswKWcY0p/gM5a47mf06fKMbN8m3F+mvDyKhzyry7z3I5UhL4NLluEWfJrUKV+K64hTt7qphHMnkBht+vDs6cSdRkD2VxWatQZ3MBAziLUKFlW8I5fVkGNgjuWyv0gaDDjZu1IqmKbJgd1hvRVuod0cM8xxlfXyx+owzJ2AnjKIzYM0YW3qvZuoU736Vpw4+XaQl0m0l6OBe531WyCH0WgO0613fgn0yQNJic0AxSv1nJmQ0C9h27F/xJ+3St18GO7T/unjARu8/MCgNWDMHm8vmeTFnSRfh5
# k8s2-master:22 SSH-2.0-OpenSSH_7.6p1 Ubuntu-4ubuntu0.3
k8s2-master ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBGCOYfqR4m29v1dhNa+NZdB/mr9LP+kuOSkvnwhNS59LnAN0rDbtdRC6nqEnhfTRZBpNULm5F8hq6oc3qDi/Nh0=
# k8s2-master:22 SSH-2.0-OpenSSH_7.6p1 Ubuntu-4ubuntu0.3
k8s2-master ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIPNoeenh2kx3GDGJ2BvZRXY8AxtJ9AvowhVdV6LBckRS
```

参考：

 - [ssh-keyscan(1) — Linux manual page](https://man7.org/linux/man-pages/man1/ssh-keyscan.1.html)
 - [ssh-keyscan(1) - Linux man page](https://linux.die.net/man/1/ssh-keyscan)
 - [ssh-keyscan - Unix, Linux Command](https://www.tutorialspoint.com/unix_commands/ssh-keyscan.htm)

