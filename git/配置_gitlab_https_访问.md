![在这里插入图片描述](https://img-blog.csdnimg.cn/3ef577bddae9457e83a2b14b91d4e04b.png)




## 1. 备份

```bash
docker exec  -ti gitlab-ce  gitlab-rake gitlab:backup:create
```
## 2. 生成SSL证书

```bash
yum install openssl openssl-devel -y
mkdir /data/gitlab/config/ssl ; cd /data/gitlab/config/ssl

### 生成证书
openssl req -new -newkey rsa:2048 -sha256 -nodes -out gitlab.demo.com.csr -keyout gitlab.demo.com.key -subj "/C=CN/ST=Shanghai/L=Shanghai/O=demo Inc./OU=Web Security/CN=gitlab.demo.com"

openssl x509 -req -days 365 -in gitlab.demo.com.csr -signkey gitlab.demo.com.key -out gitlab.demo.com.crt
```

## 3. 配置文件

```bash
$ vim /opt/gitlab/config/gitlab.rb
### 添加如下内容 ###
external_url 'https://gitlab.demo.com'
nginx['redirect_http_to_https'] = true
nginx['ssl_certificate'] = "/etc/gitlab/ssl/gitlab.demo.com.crt"
nginx['ssl_certificate_key'] = "/etc/gitlab/ssl/gitlab.demo.com.key"

nginx['proxy_set_headers'] = {
  "X-Forwarded-Proto" => "https",
  "X-Forwarded-Ssl" => "on"
}
```

## 4. 重启

```bash
$ docker exec  -ti gitlab-ce gitlab-ctl reconfigure
....
$ docker exec  -ti gitlab-ce gitlab-ctl restart    
ok: run: alertmanager: (pid 628202) 0s
ok: run: gitaly: (pid 628211) 0s
ok: run: gitlab-exporter: (pid 628229) 1s
ok: run: gitlab-kas: (pid 628249) 0s
ok: run: gitlab-workhorse: (pid 628259) 0s
ok: run: logrotate: (pid 628271) 1s
ok: run: nginx: (pid 628277) 0s
ok: run: postgres-exporter: (pid 628285) 1s
ok: run: postgresql: (pid 628295) 0s
ok: run: prometheus: (pid 628304) 0s
ok: run: puma: (pid 628320) 0s
ok: run: redis: (pid 628325) 0s
ok: run: redis-exporter: (pid 628333) 1s
ok: run: registry: (pid 628340) 0s
ok: run: sidekiq: (pid 628355) 0s
ok: run: sshd: (pid 628361) 0s

```
##  5. 访问
配置域名解析：
windows： `C:\Windows\System32\drivers\etc\hosts`

```bash
192.168.1.10 gitlab.demo.com
```

访问：`https://gitlab.demo.com`
