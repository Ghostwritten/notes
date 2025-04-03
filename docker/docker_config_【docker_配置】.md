```bash
$ cat /etc/docker/daemon.json 
{
   "exec-opts": ["native.cgroupdriver=systemd"],
   "data-root": "/data/docker",
   "live-restore": true,
   "insecure-registries" : ["registry.dev.com"],
   "dns": ["8.8.8.8"],
   "log-driver": "json-file",
   "log-opts": {
     "max-size":  "100m",
     "max-file": "5"
    },
   "registry-mirrors": [
    "https://hub-mirror.c.163.com",
    "https://mirror.baidubce.com"
  ]
 }
```

参考：

- [Docker daemon configuration overview](https://docs.docker.com/config/daemon/)
- [https://docs.docker.com/config/containers/live-restore/#enable-live-restore](https://docs.docker.com/config/containers/live-restore/#enable-live-restore)
