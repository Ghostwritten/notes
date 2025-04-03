

```bash
$ vim /etc/systemd/system/crio.service.d/http-proxy.conf
[Service]
Environment="HTTP_PROXY=http://192.168.21.101:7890" "HTTPS_PROXY=http://192.168.21.101:7890" "NO_PROXY=localhost,127.0.0.1,192.168.0.0/16,10.0.0.0/8,172.16.0.0/12,::1,bsgchina.com"

$ systemctl daemon-reload
$ systemctl restart  crio
$ systemctl show crio | grep -i environment
```

