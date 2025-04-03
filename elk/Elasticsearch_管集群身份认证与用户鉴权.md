

----
## 1. 数据泄露
5700 万用户数据泄漏
1.08 亿条投注信息泄漏
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3f2b29813bc86cb8b5965d32cae939c4.png)
## 2. 原因分析

 - ES 在默认安装后，不提供任何形式的安全防护
 - 错误的配置信息导致公网可以访问 ES 集群 在 `elasticsearch.yml` 文件中， `server.host` 被错误的配置为`0.0.0.0`

## 3. 数据安全性的基本需求

 - 身份认证：鉴定用户是否合法
 - 用户鉴权：指定哪个用户可以访问哪个索引
 - 传输加密
 - 日志审计
 - ![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/2a57ef3aac1d9e4092bc946494f2febc.png)


## 4. 一些免费的方案

 - 设置 Nginx 的反向代理
 - 安装免费的 Security 插件
Search Guard - [https://search-guard.com/](https://search-guard.com/)
ReadOnly REST - [https://github.com/sscarduzio/elasticsearc...](https://github.com/sscarduzio/elasticsearch-readonlyrest-plugin)
 - X-Pack 的 Basic 版
从 ES 6.8 & ES 7.0 开始，Security 纳入 x-pack 的 Basic 版本中，免费使用一些基本的功能
[https://www.elastic.co/what-is/elastic-sta...](https://www.elastic.co/cn/what-is/elastic-stack-security)
## 5. Anthentication - 身份认证
认证体系的几种类型
 - 提供用户名和密码
 - 提供秘钥或 Kerberos 票据

Realms : X-Pack 中的认证服务

 - 内置 Realms （免费）：File / Native (用户名密码保存在 Elasticsearch)
外部 Realms （收费）：LDAP / Active Directory / PKI / SAML / Kerberos
## 6. RBAC - 用户鉴权
什么是 RBAC：Role Based Access Control， 定义一个角色，并分配一组权限。权限包括索引 级，字段级，集群级的不同的操作。然后通过将角色分配给用户，使得用户拥有这些权限
 - User：The authenticated User
 - Role：A named set of permissions
 - Permission – A set of one or more privileges against a secured
   resource
 - Privilege – A named group of 1 or more actions that user may execute
   against a secured resource 

## 7. Privilege
Cluster Privileges

 - all / monitor / manager / manage_index / manage_index_template / manage_rollup

Indices Privileges

 - all / create / create_index / delete / delete_index / index / manage/ read /writeview_index_metadata

## 8. 创建内置的用户和角色
内置的角色与用户

| 用户                     | 角色                                                                                                       |
|------------------------|----------------------------------------------------------------------------------------------------------|
| elastic                | Supper User                                                                                              |
| kibana                 | The user that is used by Kibana to connect and communicate with Elasticsearch.                           |
| logstash_system        | The user that is used by Logstash when storing monitoring information in Elasticsearch.                  |
| beats_system           | The user that the different Beats use when storing monitoring information in Elasticsearch.              |
| apm_system             | The user that the APM server uses when storing monitoring information in Elasticsearch.                  |
| Remote_monitoring_user | The user that is used by Metricbeat when collecting and storing monitoring information in Elasticsearch. |


## 9. 使用 Security API 创建用户

```bash
POST /_security/user/lsk
{
      "password": "password",
      "roles": ["admin"],
      "full_name": "Crazy Zard",
      "email":"xxxxx@gmail.com",
      "metadata": {
        "intelligence":7
      }
}
```
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/bcd51dd76cdbefda24fa2a247fc0ca51.png)


## 10. 开启并配置 X-Pack 的认证与鉴权
修改配置文件，打开认证于授权
本机测试
```bash
bin/elasticsearch -E node.name=node0 -E cluster.name=geektime -E path.data=node0_data -E http.port=9200 -E xpack.security.enabled=true -E xpack.security.transport.ssl.enabled=true
```
linux：

```bash
bin/elasticsearch -E node.name=node0 -E cluster.name=geektime -E path.data=node0_data -E network.host=192.168.211.61 -E discovery.seed_hosts=192.168.211.61:9300   -E http.port=9200 -E xpack.security.enabled=true -E xpack.security.transport.ssl.enabled=true
```

创建默认的用户和分组

```bash
$ bin/elasticsearch-password interactive
[elastic@slave1 elasticsearch-7.3.1]$ bin/elasticsearch-setup-passwords interactive
future versions of Elasticsearch will require Java 11; your Java version from [/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.282.b08-1.el7_9.x86_64/jre] does not meet this requirement
Initiating the setup of passwords for reserved users elastic,apm_system,kibana,logstash_system,beats_system,remote_monitoring_user.
You will be prompted to enter passwords as the process progresses.
Please confirm that you would like to continue [y/N]y


Enter password for [elastic]: 
Reenter password for [elastic]: 
Enter password for [apm_system]: 
Reenter password for [apm_system]: 
Enter password for [kibana]: 
Reenter password for [kibana]: 
Enter password for [logstash_system]: 
Reenter password for [logstash_system]: 
Enter password for [beats_system]: 
Reenter password for [beats_system]: 
Enter password for [remote_monitoring_user]: 
Reenter password for [remote_monitoring_user]: 
Changed password for user [apm_system]
Changed password for user [kibana]
Changed password for user [logstash_system]
Changed password for user [beats_system]
Changed password for user [remote_monitoring_user]
Changed password for user [elastic]

使用 docker 时并进入到容器中 elasticsearch-setup-passwords interactive
```

当集群开启身份认证之后，配置 Kibana
Demo

 - 创建一个 Role，配置为某个索引只读权限 、创建一个用户，把用户加入 Role

## 11. 配置 Kibana
修改 kibana.yml

```bash
server.host: "192.168.211.61"
elasticsearch.hosts: ["http://192.168.211.61:9200"]
elasticsearch.username: “kibana”
elasticsearch.password: “changeme”
```

```bash
./bin/Kibana
```
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e62a47dba43848db14eb8739440848c6.png)
## 12. 测试
### 12.1 测试超级权限读写操作
```bash
POST orders/_search
{}

#验证写权限,报错
POST orders/_bulk
{"index":{}}
{"product" : "1","price" : 18,"payment" : "master","card" : "9876543210123456","name" : "jack"}
{"index":{}}
{"product" : "2","price" : 99,"payment" : "visa","card" : "1234567890123456","name" : "bob"}
```


### 12.2 创建 new Role
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/9e849fe3c0b5b2b35f828a3def7927e6.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ae0d417d48986291c5750243d0e16ada.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6a61c021619c59b7fdd9f234acb5b296.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6e0bb87ab8e2bdf417cce2d034430771.png)


### 12.3 创建 new user
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a3eeb9053e6f1c2395dd185b5ab2a949.png)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a97c78c2ba082fd16d9eb8addce0a668.png)
创建成功
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c9d6ec0b8bfb7677ab6c893ac4a7dc0f.png)

### 12.4 测试新用户登录
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/43819e7fb8548c0cca53cb4895be0369.png)
### 12.5 测试新用户读写
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/7a4dd2fcabbee090fc93c200ad73ab30.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/df2ef697213a120a923e2873e7f397d1.png)

