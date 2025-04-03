

----
## 1. 为啥要加密通讯

 - 加密数据 - 避免数据抓包，敏感信息泄露
 - 验证身份 - 避免 Imposter Node
Data/Cluster State

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/115b5912d4e63064fd90dbdb3d120545.png)
## 2. 为节点创建证书
TLS

 - TLS 协议要求 Trusted Certificate Authority（CA）签发的 X.509 的证书

证书认证的不同级别

 - Certificate – 节点加入需要使用相同 CA 签发的证书
 - Full Verification – 节点加入集群需要相同 CA 签发的证书，还需要验证 Host name 或 IP 地址
 - No Verification – 任何节点都可以加入，开发环境中用于诊断目的

## 3. 生成节点证书

 - bin/elasticsearch-certutil ca
![4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)](https://i-blog.csdnimg.cn/blog_migrate/a165f519a2a73b45988718c3cb35717e.png)


 - bin/elasticsearch-certutil cert –ca elastic-stack-ca.p12

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0e6765e77002dbc09c180cd7f50606e6.png)

## 4. 配置节点间通讯
```bash
[elastic@slave1 elasticsearch-7.3.1]$ mkdir config/certs
[elastic@slave1 elasticsearch-7.3.1]$ cp elastic-certificates.p12 config/certs
[elastic@slave1 elasticsearch-7.3.1]$ ls config/certs
elastic-certificates.p12
```


 - [https://www.elastic.co/guide/en/elasticsea...](https://www.elastic.co/guide/en/elasticsearch/reference/7.1/configuring-tls.html)

配置文件添加加密参数

```bash
[elastic@slave1 elasticsearch-7.3.1]$ tail config/elasticsearch.yml
xpack.security.transport.ssl.enabled: true
xpack.security.transport.ssl.verification_mode: certificate 
xpack.security.transport.ssl.keystore.path: certs/elastic-certificates.p12 
xpack.security.transport.ssl.truststore.path: certs/elastic-certificates.p12
```
或者命令-E 形式
[Elasticsearch本地安装与简单配置详情请点击](https://ghostwritten.blog.csdn.net/article/details/109385145)
第一个节点：

```bash
bin/elasticsearch -E node.name=node0 -E cluster.name=geektime -E path.data=node0_data -E network.host=192.168.211.61 -E discovery.seed_hosts=192.168.211.61:9300,192.168.211.62:9300  -E http.port=9200  -E transport.port=9300 -E xpack.security.enabled=true -E xpack.security.transport.ssl.enabled=true -E xpack.security.transport.ssl.verification_mode=certificate  -E xpack.security.transport.ssl.keystore.path=certs/elastic-certificates.p12  -E xpack.security.transport.ssl.truststore.path=certs/elastic-certificates.p12
```


第二个节点
```bash
bin/elasticsearch -E node.name=node1 -E cluster.name=geektime -E path.data=node1_data -E network.host=192.168.211.62 -E discovery.seed_hosts=192.168.211.61:9300,192.168.211.62:9300  -E http.port=9200  -E transport.port=9300 -E xpack.security.enabled=true -E xpack.security.transport.ssl.enabled=true -E xpack.security.transport.ssl.verification_mode=certificate  -E xpack.security.transport.ssl.keystore.path=certs/elastic-certificates.p12  -E xpack.security.transport.ssl.truststore.path=certs/elastic-certificates.p12

```
登录测试验证成功
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/cf68bd288efc1a9251afe6d3731654ce.png)



