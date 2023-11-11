


----
## 1. 为啥需要 HTTPS
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210311162423315.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
## 2. 配置 Elasticsearch for HTTPS

```bash
xpack.security.http.ssl.enabled: true
xpack.security.http.ssl.keystore.path: certs/elastic-certificates.p12
xpack.security.http.ssl.truststore.path: certs/elastic-certificates.p12
```
ios本机命令
```bash
# ES 启用 https
bin/elasticsearch -E node.name=node0 -E cluster.name=geektime -E path.data=node0_data -E http.port=9200 -E xpack.security.enabled=true -E xpack.security.transport.ssl.enabled=true -E xpack.security.transport.ssl.verification_mode=certificate -E xpack.security.transport.ssl.keystore.path=certs/elastic-certificates.p12 -E xpack.security.http.ssl.enabled=true -E xpack.security.http.ssl.keystore.path=certs/elastic-certificates.p12 -E xpack.security.http.ssl.truststore.path=certs/elastic-certificates.p12
```
linux命令

```bash
bin/elasticsearch -E node.name=node0 -E cluster.name=geektime -E path.data=node0_data -E network.host=192.168.211.61 -E discovery.seed_hosts=192.168.211.61:9300  -E http.port=9200  -E transport.port=9300 -E xpack.security.enabled=true -E xpack.security.transport.ssl.enabled=true -E xpack.security.transport.ssl.verification_mode=certificate  -E xpack.security.transport.ssl.keystore.path=certs/elastic-certificates.p12  -E xpack.security.transport.ssl.truststore.path=certs/elastic-certificates.p12 -E xpack.security.http.ssl.enabled=true -E xpack.security.http.ssl.keystore.path=certs/elastic-certificates.p12 -E xpack.security.http.ssl.truststore.path=certs/elastic-certificates.p12
```
浏览器访问[https://192.168.211.61:9200/_cat/nodes](https://192.168.211.61:9200/_cat/nodes)



## 3. 配置 Kibana 连接 ES HTTPS

为kibana生成pem
```bash
# 为kibana生成pem
[elastic@slave1 elasticsearch-7.3.1]$ openssl pkcs12 -in elastic-certificates.p12 -cacerts -nokeys -out elastic-ca.pem
Enter Import Password:
MAC verified OK
[elastic@slave1 elasticsearch-7.3.1]$ cp elastic-ca.pem config/certs/
[elastic@slave1 elasticsearch-7.3.1]$ ls config/certs/
elastic-ca.pem  elastic-certificates.p12

```

```bash
$ vim config/kibana.yml
elasticsearch.hosts: ["https://192.168.211.61:9200"]
elasticsearch.ssl.certificateAuthorities: [ "/home/elastic/elasticsearch-7.3.1/config/certs/elastic-ca.pem" ]
elasticsearch.ssl.verificationMode: certificate
```
执行：

```bash
[elastic@slave1 kibana-7.3.1-linux-x86_64]$ ./bin/kibana
```
测试访问ES集群
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210311165941651.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
## 4. 配置使用 HTTPS 访问 Kibana

```bash
[elastic@slave1 elasticsearch-7.3.1]$ bin/elasticsearch-certutil ca --pem
[elastic@slave1 elasticsearch-7.3.1]$ mkdir /home/elastic/kibana-7.3.1-linux-x86_64/config/certs
[elastic@slave1 elasticsearch-7.3.1] unzip elastic-stack-ca.zip 
Archive:  elastic-stack-ca.zip
   creating: ca/
  inflating: ca/ca.crt               
  inflating: ca/ca.key           
[elastic@slave1 elasticsearch-7.3.1]$ cp -r ca/* elastic-certificates.p12 /home/elastic/kibana-7.3.1-linux-x86_64/config/certs
[elastic@slave1 elasticsearch-7.3.1]$ ls /home/elastic/kibana-7.3.1-linux-x86_64/config/certs
ca.crt  ca.key  elastic-certificates.p12
[elastic@slave1 elasticsearch-7.3.1]$ cd /home/elastic/kibana-7.3.1-linux-x86_64/config/certs	
[elastic@slave1 certs]$ cd /home/elastic/kibana-7.3.1-linux-x86_64
[elastic@slave1 kibana-7.3.1-linux-x86_64]$ ls
bin  built_assets  config  data  LICENSE.txt  node  node_modules  NOTICE.txt  optimize  package.json  plugins  README.txt  src  webpackShims  x-pack
[elastic@slave1 kibana-7.3.1-linux-x86_64]$ vim config/kibana.yml 
server.ssl.enabled: true
server.ssl.certificate: config/certs/ca.crt
server.ssl.key: config/certs/ca.key

[elastic@slave1 kibana-7.3.1-linux-x86_64]$ ./bin/kibana

```
访问[https://192.168.211.61:6501](https://192.168.211.61:6501)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210311171614527.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

