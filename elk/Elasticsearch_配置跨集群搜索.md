## 1. 水平扩展的痛点
单集群 - 当水平扩展时，节点数不能无限增加

 - 当集群的 meta 信息（节点，索引，集群状态）过多，会导致更新压力变大，单个 Active Master会成为性能瓶颈，导致整个集群无法正常工作

早起版本，通过 Tribe Node 可以实现多集群访问的需求，但是还存在一定的问题

 - Tribe Node 会以 Client Node 的方式加入集群。集群中 Master 节点的任务变更需要 Tribe Node的回应才能继续
 - Tribe Node 不保存 Cluster State 信息，一旦重启，初始化很慢 当多个集群存在索引重名的情况下，只能设置一种Perfer 规则


## 2. 跨集群搜索 - Cross Cluster Search
早期 `Tribe Node` 的方案存在一定的问题，现已被 Deprecated ES5.3 引入跨集群搜索的功能（Cross Cluster Search），推荐使用允许任何节点扮演 `federated` 节点，以轻量的方式，将搜索请求进行代理不需要以 `Client Node` 的形式加入其它集群

## 3. 创建集群
[本地安装elasticsearch](https://blog.csdn.net/xixihahalelehehe/article/details/109385145)


/启动3个集群

```bash
bin/elasticsearch -E node.name=cluster0node -E cluster.name=cluster0 -E path.data=cluster0_data -E discovery.type=single-node -E http.port=9200 -E transport.port=9300
bin/elasticsearch -E node.name=cluster1node -E cluster.name=cluster1 -E path.data=cluster1_data -E discovery.type=single-node -E http.port=9201 -E transport.port=9301
bin/elasticsearch -E node.name=cluster2node -E cluster.name=cluster2 -E path.data=cluster2_data -E discovery.type=single-node -E http.port=9202 -E transport.port=9302
```


//在每个集群上设置动态的设置

```bash
PUT _cluster/settings
{
  "persistent": {
    "cluster": {
      "remote": {
        "cluster0": {
          "seeds": [
            "127.0.0.1:9300"
          ],
          "transport.ping_schedule": "30s"
        },
        "cluster1": {
          "seeds": [
            "127.0.0.1:9301"
          ],
          "transport.compress": true,
          "skip_unavailable": true
        },
        "cluster2": {
          "seeds": [
            "127.0.0.1:9302"
          ]
        }
      }
    }
  }
}
```

使用CURL命令put执行：

```bash
curl -XPUT "http://localhost:9200/_cluster/settings" -H 'Content-Type: application/json' -d'
{"persistent":{"cluster":{"remote":{"cluster0":{"seeds":["127.0.0.1:9300"],"transport.ping_schedule":"30s"},"cluster1":{"seeds":["127.0.0.1:9301"],"transport.compress":true,"skip_unavailable":true},"cluster2":{"seeds":["127.0.0.1:9302"]}}}}}'

curl -XPUT "http://localhost:9201/_cluster/settings" -H 'Content-Type: application/json' -d'
{"persistent":{"cluster":{"remote":{"cluster0":{"seeds":["127.0.0.1:9300"],"transport.ping_schedule":"30s"},"cluster1":{"seeds":["127.0.0.1:9301"],"transport.compress":true,"skip_unavailable":true},"cluster2":{"seeds":["127.0.0.1:9302"]}}}}}'

curl -XPUT "http://localhost:9202/_cluster/settings" -H 'Content-Type: application/json' -d'
{"persistent":{"cluster":{"remote":{"cluster0":{"seeds":["127.0.0.1:9300"],"transport.ping_schedule":"30s"},"cluster1":{"seeds":["127.0.0.1:9301"],"transport.compress":true,"skip_unavailable":true},"cluster2":{"seeds":["127.0.0.1:9302"]}}}}}'
```


#创建测试数据

```bash
curl -XPOST "http://localhost:9200/users/_doc" -H 'Content-Type: application/json' -d'
{"name":"user1","age":10}'

curl -XPOST "http://localhost:9201/users/_doc" -H 'Content-Type: application/json' -d'
{"name":"user2","age":20}'

curl -XPOST "http://localhost:9202/users/_doc" -H 'Content-Type: application/json' -d'
{"name":"user3","age":30}'
```
查看三个集群的索引

```bash
[root@slave1 ~]# curl http://localhost:9200/_cat/indices
yellow open users RoX-JDKuQ7-L9WFC7N5b9A 1 1 1 0 3.6kb 3.6kb
[root@slave1 ~]# curl http://localhost:9201/_cat/indices
yellow open users 0Nz5rbMkQ6CtFirpNMk6fw 1 1 1 0 3.6kb 3.6kb
[root@slave1 ~]# curl http://localhost:9202/_cat/indices
yellow open users 0IduI24KSO2DM-A0kWSX6g 1 1 1 0 3.6kb 3.6kb
```

## 4. 查询

```bash
GET /users,cluster1:users,cluster2:users/_search
{
  "query": {
    "range": {
      "age": {
        "gte": 20,
        "lte": 40
      }
    }
  }
}
```



