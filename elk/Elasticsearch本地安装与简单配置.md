
## 1.部署架构
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201030144739754.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70#pic_center)
## 2. 安装java
安装java


```bash
$ yum install -y java-1.8.0-openjdk.x86_64

$ rpm -ql java-1.8.0-openjdk.x86_64
/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.262.b10-0.el7_8.x86_64/jre/bin/policytool

$ vim /etc/profile
export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.262.b10-0.el7_8.x86_64/jre
export PATH=$JAVA_HOME/bin:$PATH 
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar 

$ source /etc/profile
```
配置最大文件打开数
```bash
echo "* soft nofile 65535"  >> /etc/security/limits.conf
echo "* hard nofile 65535"  >> /etc/security/limits.conf
```

```bash
$ vim /etc/sysctl.conf 
vm.max_map_count=655360
保存后，执行：
$ sysctl -p
vm.max_map_count = 655360
```
注销之后重新登录才能生效。

运行es，需安装并配置JDK
设置$JAVA_HOME
各个版本对jdk版本的依赖

ES 7.x 不需要本地 JDK 环境支持：
ES 5，安装需要 JDK 8 以上
ES 6.5，安装需要 JDK 11 以上
ES 7.2.1，内置了 JDK 12
## 3. elasticsearch下载
elastic全家桶：[https://www.elastic.co/cn/downloads/](https://www.elastic.co/cn/downloads/)
[elasticsearch下载](https://www.elastic.co/cn/downloads/elasticsearch)

安装方式：

 - 二进制运行
 - docker运行
 - helm chart for kubernetes
 - puppet module
## 4. elasticsearch目录结构如下
 - bin ：脚本文件，包括 ES 启动 & 安装插件等等
 - config ： elasticsearch.yml（ES 配置文件）、jvm.options（JVM 配置文件）、日志配置文件等等
 - JDK ： 内置的 JDK，JAVA_VERSION="12.0.1"
 - lib ： 类库
 - logs ： 日志文件
 - modules ： ES 所有模块，包括 X-pack 等
 - plugins ： ES 已经安装的插件。默认没有插件
 - data ： ES 启动的时候，会有该目录，用来存储文档数据。该目录可以设置
## 5. JVM配置

```bash
vim ./config/jvm.options
-Xms1g
-Xmx1g
```
7.1版本默认是1G
建议：
Xmx与Xms设置成一样
Xmx不要超过机器内存得50%
不要超过30G,原因参考
[https://www.elastic.co/cn/blog/a-heap-of-trouble](https://www.elastic.co/cn/blog/a-heap-of-trouble)
[https://www.elastic.co/guide/cn/elasticsearch/guide/current/heap-sizing.html](https://www.elastic.co/guide/cn/elasticsearch/guide/current/heap-sizing.html)

## 6. elasticsearch部署

```bash
groupadd elastic
useradd -g elastic elastic
tar -xvf elasticsearch-7.2.0-linux-x86_64.tar.gz 
useradd elastic
chown -R elastic:elastic  elasticsearch-7.2.0
chmod -R 775 elasticsearch-7.2.0
su - elastic
cd elasticsearch-7.2.0
./bin/elasticsearch 
```
## 7. elasticsearch安装插件

```bash
 ./bin/elasticsearch-plugin install analysis-icu
 ./bin/elasticsearch-plugin list
```
## 8. elasticsearch部署多个实例

```bash
./bin/elasticsearch -E node.name=node1 -E cluster.name=cluster1 -E path.data=node1_data -d
./bin/elasticsearch -E node.name=node2 -E cluster.name=cluster1 -E path.data=node2_data -d 
./bin/elasticsearch -E node.name=node3 -E cluster.name=cluster1 -E path.data=node3_data -d
ps -ef |grep elastic
```

```bash
$ curl http://localhost:9200
{
  "name" : "node1",
  "cluster_name" : "cluster1",
  "cluster_uuid" : "Ug5W-8dDQS-y5Gq-Uo1rOQ",
  "version" : {
    "number" : "7.2.0",
    "build_flavor" : "default",
    "build_type" : "tar",
    "build_hash" : "508c38a",
    "build_date" : "2019-06-20T15:54:18.811730Z",
    "build_snapshot" : false,
    "lucene_version" : "8.0.0",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}

$ curl http://localhost:9200/_cat/nodes?v
ip        heap.percent ram.percent cpu load_1m load_5m load_15m node.role master name
127.0.0.1           14          97  22    0.05    0.40     0.32 mdi       *      node1
127.0.0.1            7          97  19    0.05    0.40     0.32 mdi       -      node2
127.0.0.1           18          97  12    0.05    0.40     0.32 mdi       -      node3

```

如果用ip:9200访问效果：

```bash
./bin/elasticsearch -E node.name=node1 -E cluster.name=cluster1 -E path.data=node1_data -E network.host=192.168.211.60 -E http.port=9200 -E transport.port=9300 -E discovery.seed_hosts=["192.168.211.60:9300","192.168.211.60:9301","192.168.211.60:9302"], -E cluster.initial_master_nodes=["192.168.211.60:9300","192.168.211.60:9301","192.168.211.60:9302"] -d

./bin/elasticsearch -E node.name=node2 -E cluster.name=cluster1 -E path.data=node2_data -E network.host=192.168.211.60 -E http.port=9201 -E transport.port=9301 -E discovery.seed_hosts=["192.168.211.60:9300","192.168.211.60:9301","192.168.211.60:9302"], -E cluster.initial_master_nodes=["192.168.211.60:9300","192.168.211.60:9301","192.168.211.60:9302"] -d

./bin/elasticsearch -E node.name=node3 -E cluster.name=cluster1 -E path.data=node3_data -E network.host=192.168.211.60 -E http.port=9202 -E transport.port=9302 -E discovery.seed_hosts=["192.168.211.60:9300","192.168.211.60:9301","192.168.211.60:9302"], -E cluster.initial_master_nodes=["192.168.211.60:9300","192.168.211.60:9301","192.168.211.60:9302"] -d
```

## 9. kibana下载部署
[kibana下载](https://www.elastic.co/cn/downloads/kibana)
安装方式：

 - 二进制运行
 - docker运行
 - helm chart for kubernetes
 - puppet module

```bash
$ tar -xvf kibana-7.2.0-linux-x86_64.tar.gz
$ cd kibana-7.2.0
$ vim config/kibana.yml
server.port: 5601
server.host: "192.168.211.60"
elasticsearch.hosts: ["http://192.168.211.60:9200"]

$ nohup ./bin/kibana --allow-root & > /dev/null 2>&1

如果用elastic用户运行
$ chmod 775 -R kibana-7.2.0-linux-x86_64
$ chown -R elastic:elastic kibana-7.2.0-linux-x86_64
$ su - elastic
$ nohup ./bin/kibana & > /dev/null 2>&1
```

访问：`192.168.211.60：5601`
可以导入样板数据

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020103016245988.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70#pic_center)
查看样板dashboard效果
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201030162657209.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70#pic_center)

kibanna查用得工具：dev tools
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201030164354129.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70#pic_center)

## 10. kibana安装插件
格式
```bash
bin/kibana-plugin install <package name or URL>
```

```bash
$ bin/kibana-plugin install x-pack
$ bin/kibana-plugin install https://artifacts.elastic.co/downloads/packs/x-pack/x-pack-6.0.0.zip
$ bin/kibana-plugin install file:///some/local/path/x-pack.zip -d path/to/directory
```

Kibana 服务需要有 optimize 目录的写权限。如果您使用 sudo 或者 su 安装插件，您需要确保这些命令使用 kibana 用户执行。这个用户已经默认为您添加了，它用于包的安装。

```bash
$ sudo -u kibana bin/kibana-plugin install x-pack
```

如果插件使用了不同的用户安装且服务又没有运行起来，您就需要修改这些文件的所属用户：

```bash
$ chown -R kibana:kibana /path/to/kibana/optimize
```

## 11 logstash下载与部署
### 11.1 java安装

```bash
$ yum install -y java-1.8.0-openjdk.x86_64

$ rpm -ql java-1.8.0-openjdk.x86_64
/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.262.b10-0.el7_8.x86_64/jre/bin/policytool

$ vim /etc/profile
export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.262.b10-0.el7_8.x86_64/jre
export PATH=$JAVA_HOME/bin:$PATH 
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar 

$ source /etc/profile
```

### 11.2  logstash下载安装
[logstash 官方下载](https://www.elastic.co/cn/downloads/past-releases#logstash)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201031183608252.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70#pic_center)
解压

```bash
tar xvf  logstash-7.2.0.tar.gz  /
```

### 11.3 下载并配置导入数据信息
下载测试数据信息地址：[https://grouplens.org/datasets/movielens/](https://grouplens.org/datasets/movielens/)
随意下载一个数据源解压后我获取一个movies.csv存放/logstash-7.2.0/data/路径下
设置logstash配置文件`logstash.conf`

```bash
input {
  file {
    path => "/logstash-7.2.0/data/movies.csv"   #修改
    start_position => "beginning"
    sincedb_path => "/dev/null"
  }
}
filter {
  csv {
    separator => ","
    columns => ["id","content","genre"]
  }

  mutate {
    split => { "genre" => "|" }
    remove_field => ["path", "host","@timestamp","message"]
  }

  mutate {

    split => ["content", "("]
    add_field => { "title" => "%{[content][0]}"}
    add_field => { "year" => "%{[content][1]}"}
  }

  mutate {
    convert => {
      "year" => "integer"
    }
    strip => ["title"]
    remove_field => ["path", "host","@timestamp","message","content"]
  }

}
output {
   elasticsearch {
     hosts => "http://localhost:9200"
     index => "movies"
     document_id => "%{id}"
   }
  stdout {}
}
```

### 11.4 数据导入
```bash
/logstash-7.2.0/bin/logstash -f /logstash-7.2.0/config/logstash.conf
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201031182736491.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70#pic_center)

kibanna查看movie 索引
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201031183317522.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70#pic_center)
#### 注意：
**问题1**
```bash
OpenJDK 64-Bit Server VM warning: Option UseConcMarkSweepGC was deprecated in version 9.0 and will likely be removed in a future release.
```
因jdk版本兼容问题，使用jdk 1.8.0没有，但使用过高的11版本有该问题，但该类问题并不影响正常使用logstash。

**问题2：**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201031181845498.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201031181902345.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70#pic_center)
解决方法：
**要分清logstash的配置文件.xxx.conf与设置文件logstash.yml。**



