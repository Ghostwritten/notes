![](https://i-blog.csdnimg.cn/blog_migrate/3498a6b0ebf024b8f8cc05b3f13c8eda.png)



## 1.sysctl

```bash
[root@github es_tls]# cat /etc/sysctl.conf 
# sysctl settings are defined through files in
# /usr/lib/sysctl.d/, /run/sysctl.d/, and /etc/sysctl.d/.
#
# Vendors settings live in /usr/lib/sysctl.d/.
# To override a whole file, create a new file with the same in
# /etc/sysctl.d/ and put new settings there. To override
# only specific settings, add a file with a lexically later
# name in /etc/sysctl.d/ and put new settings there.
#
# For more information, see sysctl.conf(5) and sysctl.d(5).

net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1

net.netfilter.nf_conntrack_max = 262144
net.nf_conntrack_max = 262144


fs.aio-max-nr = 1065535
kernel.pid_max = 600000
net.ipv4.tcp_max_syn_backlog = 30000
net.core.somaxconn = 65535
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_timestamps = 1
net.ipv4.ip_forward = 1
net.ipv4.ip_local_reserved_ports = 30000-32767
net.ipv4.ip_local_port_range = 1024 65000
net.core.netdev_max_backlog = 300000
net.ipv4.tcp_rmem = 4096 87380 134217728
net.ipv4.tcp_wmem = 4096 87380 134217728
net.ipv4.tcp_sack = 0
net.ipv4.tcp_fin_timeout = 20
net.ipv6.conf.default.forwarding = 1
net.ipv6.conf.all.forwarding = 1
net.ipv6.route.max_size = 2147483647
net.ipv6.conf.all.accept_redirects = 0
net.ipv6.conf.default.accept_redirects = 0
vm.swappiness = 0
vm.max_map_count = 262144
fs.inotify.max_user_watches=1048576


```

## 2. swap

```bash
swapoff -a
```

## 3. hosts

```bash
echo "10.253.219.1 es01 es02 es03 kib01" >>  /etc/hosts
```

## 4. 配置 instances.yaml

```bash
$ vim instances.yaml
instances:
  - name: es01
    dns:
      - es01 
    ip:
      - 10.253.219.1

  - name: es02
    dns:
      - es02
    ip:
      - 10.253.219.1

  - name: es03
    dns:
      - es03
    ip:
      - 10.253.219.1

  - name: 'kib01'
    dns:
      - kib01
    ip:
      - 10.253.219.1


$ cat .env
COMPOSE_PROJECT_NAME=es 
CERTS_DIR=/usr/share/elasticsearch/config/certificates 
VERSION=7.17.6

$ mkdir -p /usr/share/elasticsearch/config/certificates
```

## 5. 创建证书

```bash
$ sudo docker-compose -f create-certs.yml run --rm create_certs
[+] Creating 1/0
 ✔ Volume "es_certs"  Created                                                                                                                                           0.0s 
Archive:  /certs/bundle.zip
   creating: /certs/ca/
  inflating: /certs/ca/ca.crt        
   creating: /certs/es01/
  inflating: /certs/es01/es01.crt    
  inflating: /certs/es01/es01.key    
   creating: /certs/es02/
  inflating: /certs/es02/es02.crt    
  inflating: /certs/es02/es02.key    
   creating: /certs/es03/
  inflating: /certs/es03/es03.crt    
  inflating: /certs/es03/es03.key    
   creating: /certs/kib01/
  inflating: /certs/kib01/kib01.crt  
  inflating: /certs/kib01/kib01.key  
$ sudo docker volume ls|grep es
local               es_certs

$ sudo ls /apps/data/docker/volumes/es_certs/_data
bundle.zip  ca	es01  es02  es03  kib01


```

## 6. 部署

```bash
$ sudo  docker-compose up -d
[+] Running 7/7
 ✔ Volume "es_data03"  Created                                                                                                                                          0.0s 
 ✔ Volume "es_data01"  Created                                                                                                                                          0.0s 
 ✔ Volume "es_data02"  Created                                                                                                                                          0.0s 
 ✔ Container es02      Started                                                                                                                                         10.8s 
 ✔ Container es01      Healthy                                                                                                                                         43.9s 
 ✔ Container es03      Started                                                                                                                                         10.8s 
 ✔ Container kib01     Started                                                                                                                                         40.6s 
$ sudo  docker-compose ps
NAME                IMAGE                                                  COMMAND                  SERVICE             CREATED             STATUS                    PORTS
es01                docker.elastic.co/elasticsearch/elasticsearch:7.17.6   "/bin/tini -- /usr/l…"   es01                53 seconds ago      Up 42 seconds (healthy)   0.0.0.0:9200->9200/tcp, 9300/tcp
es02                docker.elastic.co/elasticsearch/elasticsearch:7.17.6   "/bin/tini -- /usr/l…"   es02                53 seconds ago      Up 42 seconds             9200/tcp, 9300/tcp
es03                docker.elastic.co/elasticsearch/elasticsearch:7.17.6   "/bin/tini -- /usr/l…"   es03                53 seconds ago      Up 42 seconds             9200/tcp, 9300/tcp
kib01               docker.elastic.co/kibana/kibana:7.17.6                 "/bin/tini -- /usr/l…"   kib01               46 seconds ago      Up 6 seconds              0.0.0.0:5601->5601/tcp


$ sudo docker exec es01 /bin/bash -c "bin/elasticsearch-setup-passwords auto --batch --url https://es01:9200"
Changed password for user apm_system
PASSWORD apm_system = 6Je1ftTgEv7DrFzhNMDf

Changed password for user kibana_system
PASSWORD kibana_system = ZYbQBQXHHPYJvq6r2RVM

Changed password for user kibana
PASSWORD kibana = ZYbQBQXHHPYJvq6r2RVM

Changed password for user logstash_system
PASSWORD logstash_system = e7a3mdoDvEstm74ym4SK

Changed password for user beats_system
PASSWORD beats_system = SkNQ624TS29y5EAD9bXP

Changed password for user remote_monitoring_user
PASSWORD remote_monitoring_user = D965EMle8bVNHh17dl2K

Changed password for user elastic
PASSWORD elastic = fQkkGPlAaQld61gRr8GK

```
测试

```bash
$ sudo curl --cacert /apps/data/docker/volumes/es_certs/_data/ca/ca.crt -u elastic:fQkkGPlAaQld61gRr8GK  https://10.253.219.1:9200/_cat/nodes?v
ip         heap.percent ram.percent cpu load_1m load_5m load_15m node.role   master name
172.22.0.2            6          76  19   19.73   19.07    19.65 cdfhilmrstw -      es01
172.22.0.4           14          76  19   19.73   19.07    19.65 cdfhilmrstw -      es02
172.22.0.3           12          76  19   19.73   19.07    19.65 cdfhilmrstw *      es03

```

## 7. 修改 kibanna 密码
修改 `docker-compose.yaml`

```bash
......
      ELASTICSEARCH_PASSWORD: ZYbQBQXHHPYJvq6r2RVM
.....
```

重启

```bash
$ sudo docker-compose stop
[+] Stopping 4/4
 ✔ Container es02   Stopped                                                                                                                                             1.8s 
 ✔ Container kib01  Stopped                                                                                                                                             2.0s 
 ✔ Container es03   Stopped                                                                                                                                             1.8s 
 ✔ Container es01   Stopped   

$ sudo docker-compose up -d
[+] Running 4/4
 ✔ Container es03   Started                                                                                                                                             6.6s 
 ✔ Container es01   Healthy                                                                                                                                            38.1s 
 ✔ Container es02   Started                                                                                                                                             6.6s 
 ✔ Container kib01  Started 

$ sudo docker-compose ps
NAME                IMAGE                                                  COMMAND                  SERVICE             CREATED              STATUS                        PORTS
es01                docker.elastic.co/elasticsearch/elasticsearch:7.17.6   "/bin/tini -- /usr/l…"   es01                About an hour ago    Up About a minute (healthy)   0.0.0.0:9200->9200/tcp, 9300/tcp
es02                docker.elastic.co/elasticsearch/elasticsearch:7.17.6   "/bin/tini -- /usr/l…"   es02                About an hour ago    Up About a minute             9200/tcp, 9300/tcp
es03                docker.elastic.co/elasticsearch/elasticsearch:7.17.6   "/bin/tini -- /usr/l…"   es03                About an hour ago    Up About a minute             9200/tcp, 9300/tcp
kib01               docker.elastic.co/kibana/kibana:7.17.6                 "/bin/tini -- /usr/l…"   kib01               About a minute ago   Up 30 seconds                 0.0.0.0:5601->5601/tcp
                                                      
```

```bash
sudo curl --cacert /apps/data/docker/volumes/es_certs/_data/ca/ca.crt -u  kibana_system:ZYbQBQXHHPYJvq6r2RVM  https://10.253.219.1:5601
```

## 8. 清理

```bash
docker-compose stop
docker-compose rm
```
清理容器卷

```bash
$  sudo cat /etc/systemd/system/docker.service.d/docker-options.conf
[Service]
Environment="DOCKER_OPTS= --insecure-registry=0.0.0.0/0  --data-root=/apps/data/docker --log-opt max-size=50m --log-opt max-file=5 --live-restore=true --pidfile=/apps/run/docker/docker.pid --iptables=true"

$  sudo ls /apps/data/docker/volumes/es_certs/_data/
bundle.zip  ca	es01  es02  es03  kib01

$ sudo docker volume ls|grep es
local               es_certs
local               es_data01
local               es_data02
local               es_data03

sudo docker volume rm es_certs
sudo docker volume rm es_data01
sudo docker volume rm es_data02
sudo docker volume rm es_data03

$ sudo ls /apps/data/docker/volumes/es_certs/_data/
ls: cannot access /apps/data/docker/volumes/es_certs/_data/: No such file or directory
```

参考：

 - [Running the Elastic Stack on Docker](https://www.elastic.co/guide/en/elastic-stack-get-started/7.9/get-started-docker.html)
 - [Encrypting communications in an Elasticsearch Docker Containere](https://www.elastic.co/guide/en/elasticsearch/reference/7.17/configuring-tls-docker.html#_run_the_example)
