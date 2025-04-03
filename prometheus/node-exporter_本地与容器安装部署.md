#  node-exporter 本地与容器安装部署
tags: exporters
<!-- catalog: ~node-exporter~ -->
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/df39fc900987dbcb3d97a35c5fb669af.png)




## 1. 简介
在Prometheus的架构设计中，Prometheus Server并不直接服务监控特定的目标，其主要任务负责数据的收集，存储并且对外提供数据查询支持。因此为了能够能够监控到某些东西，如主机的CPU使用率，我们需要使用到Exporter。Prometheus周期性的从Exporter暴露的HTTP服务地址（通常是/metrics）拉取监控样本数据。

从上面的描述中可以看出Exporter可以是一个相对开放的概念，其可以是一个独立运行的程序独立于监控目标以外，也可以是直接内置在监控目标中。只要能够向Prometheus提供标准格式的监控样本数据即可。

这里为了能够采集到主机的运行指标如CPU, 内存，磁盘等信息。我们可以使用Node Exporter。

## 2. 容器部署
第一种：监控系统资源为主，例如：cpu、mem、disk、proc
```bash
docker run -d -p 9100:9100 --restart=always -m 5G --memory-swap=5G -v "/proc:/host/proc:ro" -v "/sys:/host/sys:ro"  -v "/:/rootfs:ro"  --name node_exporter quay.io/prometheus/node-exporter   --path.procfs /host/proc  --path.sysfs /host/sys --collector.filesystem.ignored-mount-points "^/(sys|proc|dev|host|etc)($|/)"
```
node-exporter版本不同执行参数不同，还有一下情况：

```bash
docker run -d -p 9110:9100 \
  -v "/proc:/host/proc:ro" \
  -v "/sys:/host/sys:ro" \
  -v "/:/rootfs:ro" \
  --net="host" \
--name node-exporter-test \
  quay.io/prometheus/node-exporter \
    --collector.procfs /host/proc \
    --collector.sysfs /host/sys \
    --collector.filesystem.ignored-mount-points "^/(sys|proc|dev|host|etc)($|/)"
```

第二种:：监控systemd服务

```bash
docker run --name=node-exporter -d --restart=always \
  --net="host" \
  --pid="host" \
  -v "/:/host:ro,rslave" \
  -v "/run/dbus/system_bus_socket:/var/run/dbus/system_bus_socket:ro" \
  --restart=always \
  local.harbor.io/prometheus/node-exporter:v0.18.1.1 \
  --path.rootfs=/host \
  --collector.systemd --collector.systemd.unit-whitelist="(docker|keepalived|haproxy-alcordata|haproxy-mgcluster).service"
```


## 3. 本地部署
下载：
[https://github.com/prometheus/node_exporter/releases/download/v1.0.1/node_exporter-1.0.1.linux-amd64.tar.gz](https://github.com/prometheus/node_exporter/releases/download/v1.0.1/node_exporter-1.0.1.linux-amd64.tar.gz)

```bash
wget https://github.com/prometheus/node_exporter/releases/download/v1.0.1/node_exporter-1.0.1.linux-amd64.tar.gz
tar -xvf node_exporter-1.0.1.linux-amd64.tar.gz
useradd node_exporter
#mkdir  /home/node_exporter
mv node_exporter /home/node_exporter
chmod +x /home/node_exporter/node_exporter
```

配置文件
```bash
cat <<EOF> /usr/lib/systemd/system/node_exporter.service
[Unit]
Description=Node Exporter

[Service]
User=node_exporter
EnvironmentFile=/etc/sysconfig/node_exporter
ExecStart=/home/node_exporter/node_exporter

[Install]
WantedBy=multi-user.target
EOF
```

环境变量文件
```bash
cat <<EOF>  /etc/sysconfig/node_exporter
OPTIONS="--collector.textfile.directory /var/lib/node_exporter/textfile_collector"
EOF
```
启动服务
```bash
chown node_exporter:node_exporter /usr/lib/systemd/system/node_exporter.service
systemctl daemon-reload
systemctl start node_exporter.service && systemctl enable node_exporter.service
netstat -naulp | grep 9100
```
## 4. prometheus 配置 node-exporter metrics

```bash
 - job_name: 'node-exporter'
    static_configs:
    - targets: 
       - '192.168.1.193:9100'
       - '192.168.1.194:9110'
       - '192.168.1.104:9100'
```

