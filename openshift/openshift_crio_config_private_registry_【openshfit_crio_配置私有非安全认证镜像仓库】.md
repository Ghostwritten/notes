```bash
$ mkdir -p /etc/crio/registries.d/
$ vi /etc/crio/registries.d/auth.json
{
        "auths": {
                "192.168.21.20": {
                        "auth": "YWRtaW46SGFyYm9yMTIzNDU="
                },
                "harbor.bsgchina.com": {
                        "auth": "YWRtaW46SGFyYm9yMTIzNDU="
                }
        }
}

$ vi /etc/containers/registries.conf

[[registry]]
location = "harbor.bsgchina.com"
insecure = true

$ systemctl restart crio
$ crictl pull harbor.bsgchina.com/demo/busybox:latest
Image is up to date for harbor.bsgchina.com/demo/busybox@sha256:e7816d54ae819cf8a5fd97a62bca7d75388c15eb5d65081bc3eb6f57728aa7bb
```

参考：

- [https://github.com/containers/image/blob/main/docs/containers-registries.conf.5.md](https://github.com/containers/image/blob/main/docs/containers-registries.conf.5.md)
