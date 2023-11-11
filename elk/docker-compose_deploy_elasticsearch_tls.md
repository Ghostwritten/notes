
##  1. sysctl 

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
## swap

```bash
swapoff -a
```

## hosts 配置
```bash
echo "192.168.10.71 es01 es02" >>  /etc/hosts
```

## 配置 instances.yaml

```bash
mkdir es_tls
cd es_tls
$ cat instances.yml
instances:
  - name: es01
    dns:
      - es01 
    ip:
      - 192.168.10.71

  - name: es02
    dns:
      - es02
    ip:
      - 192.168.10.71

$ cat .env 
COMPOSE_PROJECT_NAME=es 
CERTS_DIR=/usr/share/elasticsearch/config/certificates 
ELASTIC_PASSWORD=PleaseChangeMe 

$ mkdir -p /usr/share/elasticsearch/config/certificates

```

## 创建证书

```bash
$ docker-compose -f create-certs.yml run --rm create_certs
[+] Creating 2/1
 ✔ Network es_default  Created                                                                                                                                                               0.2s 
 ✔ Volume "es_certs"   Created                                                                                                                                                               0.0s 
Archive:  /certs/bundle.zip
   creating: /certs/ca/
  inflating: /certs/ca/ca.crt        
   creating: /certs/es01/
  inflating: /certs/es01/es01.crt    
  inflating: /certs/es01/es01.key    
   creating: /certs/es02/
  inflating: /certs/es02/es02.crt    
  inflating: /certs/es02/es02.key    
```

## 运行

```bash
$ docker-compose up -d
[+] Running 6/6
 ✔ Volume "es_data01"               Created                                                                                                                                                  0.0s 
 ✔ Volume "es_data02"               Created                                                                                                                                                  0.0s 
 ✔ Container kibana7                Started                                                                                                                                                  0.8s 
 ✔ Container es02                   Started                                                                                                                                                  0.9s 
 ✔ Container es01                   Healthy                                                                                                                                                 31.8s 
 ✔ Container es-wait_until_ready-1  Started  
```

## 测试

```bash
$ curl --cacert /var/lib/docker/volumes/es_certs/_data/ca/ca.crt -u elastic:PleaseChangeMe https://192.168.10.71:9200
{
  "name" : "es01",
  "cluster_name" : "docker-cluster",
  "cluster_uuid" : "6eVXcxIZT0aziRPij_Mgfw",
  "version" : {
    "number" : "7.17.6",
    "build_flavor" : "default",
    "build_type" : "docker",
    "build_hash" : "f65e9d338dc1d07b642e14a27f338990148ee5b6",
    "build_date" : "2022-08-23T11:08:48.893373482Z",
    "build_snapshot" : false,
    "lucene_version" : "8.11.1",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}

$ curl --cacert /var/lib/docker/volumes/es_certs/_data/ca/ca.crt -u elastic:PleaseChangeMe https://192.168.10.71:9200/_cluster/health?pretty
{
  "cluster_name" : "docker-cluster",
  "status" : "green",
  "timed_out" : false,
  "number_of_nodes" : 2,
  "number_of_data_nodes" : 2,
  "active_primary_shards" : 1,
  "active_shards" : 2,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 0,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,
  "active_shards_percent_as_number" : 100.0
}

$  curl --cacert /var/lib/docker/volumes/es_certs/_data/ca/ca.crt -u elastic:PleaseChangeMe https://192.168.10.71:9200/_cat/nodes?v
ip         heap.percent ram.percent cpu load_1m load_5m load_15m node.role   master name
172.29.0.2            8          98  23    0.54    1.54     1.41 cdfhilmrstw -      es01
172.29.0.4           46          98  22    0.54    1.54     1.41 cdfhilmrstw *      es02

```


参考：
- [https://www.elastic.co/guide/en/elastic-stack-get-started/7.9/get-started-docker.html](https://www.elastic.co/guide/en/elastic-stack-get-started/7.9/get-started-docker.html)
- [https://www.elastic.co/guide/en/elasticsearch/reference/7.17/configuring-tls-docker.html#_run_the_example](https://www.elastic.co/guide/en/elasticsearch/reference/7.17/configuring-tls-docker.html#_run_the_example)
