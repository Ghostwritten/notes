```bash
root@spectre:~# ssh-keygen 
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:bIcQJwbB5RqKcXGJWha9qfJpGm3YyLxkBmLlb9Hdue0 root@k8s2-master
The key's randomart image is:
+---[RSA 2048]----+
|  o=+== .        |
|  +o+o +         |
|.+...oo          |
|.+o.oo.o... .    |
|+..o.. .S..o     |
|B=. . .. .  o    |
|+O+. o     . .   |
|+o= .       .    |
|.+           E   |
+----[SHA256]-----+
'
root@spectre:~# sshpass -p 12345mtr. ssh-copy-id -f root@192.168.211.51
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/root/.ssh/id_rsa.pub"

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'root@192.168.211.51'"
and check to make sure that only the key(s) you wanted were added.
```

