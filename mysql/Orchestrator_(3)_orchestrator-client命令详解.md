
----

 - orchestrator-client是客户端命令（`/usr/local/orchestrator/resources/bin`）
 - orchestrator是服务端命令(`/usr/local/orchestrator`)

##  1. 常用命令
在生产上部署Orchestrator，可以参考文档2。

1.Orchestrator首先需要确认本身高可用的后端数据库是用单个MySQL，MySQL复制还是本身的Raft。

2.运行发现服务（web、orchestrator-client） 

```bash
# orchestrator-client -c discover -i 10.100.97.146:3306
10.100.97.146:3306

# orchestrator-client -c clusters
10.100.97.146:3306

# orchestrator-client -c topology -i 10.100.97.146:3306
10.100.97.146:3306   [0s,ok,5.7.24-log,rw,ROW,GTID]
+ 10.100.97.147:3306 [0s,ok,5.7.24-log,rw,ROW,GTID]
+ 10.100.97.148:3306 [0s,ok,5.7.24-log,rw,ROW,GTID]
```

3.确定提升规则（某些服务器更适合被提升）

```bash
orchestrator -c register-candidate -i ${::fqdn} --promotion-rule ${promotion_rule}
```

4.如果服务器出现问题，将在Web界面上的问题下拉列表中显示。使用Downtiming则不会在问题列表里显示，并且也不会进行恢复，处于维护模式。

```bash
orchestrator -c begin-downtime -i ${::fqdn}$ --duration=5m --owner=cron --reason=continuous_downtime
```

也可以用API：

```bash
curl -s "http://my.orchestrator.service:80/api/begin-downtime/my.hostname/3306/wallace/experimenting+failover/45m"
```

5.伪GTID，如果MySQL没有开启GTID，则可以开启伪GTID实现类似GTID的功能。

6.保存元数据，元数据大部分通过参数的query来获取，比如在自的表cluster里获取集群的别名(`DetectClusterAliasQuery`)、数据中心(`DetectDataCenterQuery`)、域名(`DetectClusterDomainQuery`)等，以及复制的延迟（`pt-heartbeat`）、是否半同步(`DetectSemiSyncEnforcedQuery`)。以及可以通过正则匹配：`DataCenterPattern`、`PhysicalEnvironmentPattern`等。
7. 可以给实例打标签

##  2. 命令详解
Orchestrator不仅有Web界面来进行查看和管理，还可以通过命令行（orchestrator-client）和API（curl）来执行更多的管理命令，现在来说明几个比较常用方法。

通过help来看下有哪些可以执行的命令：

```bash
./orchestrator-client --help，命令的说明可以看手册说明。
```

orchestrator-client不需要和Orchestrator服务放一起，不需要访问后端数据库，在任意一台上都可以。

注意：因为配置了Raft，有多个Orchestrator，所以需要ORCHESTRATOR_API的环境变量，orchestrator-client会自动选择leader。如：

```bash
export ORCHESTRATOR_API="test1:3000/api test2:3000/api test3:3000/api"
```

### 2.1 查询搜索系列
1.列出所有集群：clusters

默认：

```bash
orchestrator-client -c clusters test2:3307 返回包含集群别名：clusters-alias
orchestrator-client -c clusters-alias test2:3307,test
```

2.发现指定实例：discover/async-discover

同步发现：

```bash
orchestrator-client -c discover -i test1:3307 test1:3307 异步发现：适用于批量
orchestrator-client -c async-discover -i test1:3307 :null
```

3.忘记指定对象：forget/forget-cluster

忘记指定实例：

```bash
orchestrator-client -c forget -i test1:3307 忘记指定集群：
orchestrator-client -c forget-cluster -i test
```

4.打印指定集群的拓扑：topology/topology-tabulated

普通返回：

```bash
#orchestrator-client -c topology -i test1:3307
test2:3307   [0s,ok,5.7.25-0ubuntu0.16.04.2-log,rw,ROW,>>,GTID]
+ test1:3307 [0s,ok,5.7.25-0ubuntu0.16.04.2-log,ro,ROW,>>,GTID]
+ test3:3307 [0s,ok,5.7.25-log,ro,ROW,>>,GTID]
```

列表返回：

```bash
$ orchestrator-client -c topology-tabulated -i test1:3307
test2:3307  |0s|ok|5.7.25-0ubuntu0.16.04.2-log|rw|ROW|>>,GTID
+ test1:3307|0s|ok|5.7.25-0ubuntu0.16.04.2-log|ro|ROW|>>,GTID
+ test3:3307|0s|ok|5.7.25-log                 |ro|ROW|>>,GTID
```

5.查看使用哪个API：自己会选择出leader。which-api

```bash
$ orchestrator-client -c which-api
test3:3000/api
```

也可以通过 http://192.168.163.133:3000/api/leader-check 查看。

6.调用api请求，需要和 -path 参数一起：api..-path

```bash
$ orchestrator-client -c api -path clusters
[ "test2:3307" ]
$ orchestrator-client -c api -path leader-check
"OK"
$ orchestrator-client -c api -path status | jq .
{
  "Code": "OK",
  "Message": "Application node is healthy",
  "Details": {
    "Healthy": true,
    "Hostname": "node1",
    "Token": "75367992c5222101dc4968c98489238cb06c79bcb4a22c38902b6b90c9cfd486",
    "IsActiveNode": true,
    "ActiveNode": {
      "Hostname": "192.168.211.60:10008",
      "Token": "",
      "AppVersion": "",
      "FirstSeenActive": "",
      "LastSeenActive": "",
      "ExtraInfo": "",
      "Command": "",
      "DBBackend": "",
      "LastReported": "0001-01-01T00:00:00Z"
    },
    "Error": null,
    "AvailableNodes": [
      {
        "Hostname": "node1",
        "Token": "75367992c5222101dc4968c98489238cb06c79bcb4a22c38902b6b90c9cfd486",
        "AppVersion": "3.2.3",
        "FirstSeenActive": "2020-12-29 13:53:17",
        "LastSeenActive": "2020-12-29 15:02:30",
        "ExtraInfo": "",
        "Command": "",
        "DBBackend": "127.0.0.1:3306",
        "LastReported": "0001-01-01T00:00:00Z"
      }
    ],
    "RaftLeader": "192.168.211.60:10008",
    "IsRaftLeader": true,
    "RaftLeaderURI": "http://192.168.211.60:3000",
    "RaftAdvertise": "192.168.211.60",
    "RaftHealthyMembers": [
      "192.168.211.60",
      "192.168.211.62",
      "192.168.211.61"
    ]
  }
}

```

7.搜索实例：search

```bash
#orchestrator-client -c search -i test
test2:3307
test1:3307
test3:3307
```

8.打印指定实例的主库：which-master 

```bash
#orchestrator-client -c which-master -i test1:3307
test2:3307

#orchestrator-client -c which-master -i test3:3307
test2:3307

#orchestrator-client -c which-master -i test2:3307 #自己本身是主库
:0
```

9.打印指定实例的从库：which-replicas 

```bash
#orchestrator-client -c which-replicas -i test2:3307
test1:3307
test3:3307
```

10.打印指定实例的实例名：which-instance 

```bash
#orchestrator-client -c instance -i test1:3307
test1:3307
```

11.打印指定主实例从库异常的列表：which-broken-replicas，模拟test3的复制异常：

```bash
#orchestrator-client -c which-broken-replicas -i test2:3307
test3:3307
```

12.给出一个实例或则集群别名，打印出该实例所在集群下的所有其他实例。which-cluster-instances

```bash
# orchestrator-client -c which-cluster-instances -i test
test1:3307
test2:3307
test3:3307
# orchestrator-client -c which-cluster-instances -i test1:3307
test1:3307
test2:3307
test3:3307
```
13. 给出一个实例，打印该实的集群名称：默认是`hostname:port`。which-cluster 

```bash
# orchestrator-client -c which-cluster -i test1:3307
test2:3307
# orchestrator-client -c which-cluster -i test2:3307
test2:3307
# orchestrator-client -c which-cluster -i test3:3307
test2:3307
```

14.打印出指定实例/集群名或则所有所在集群的可写实例，：which-cluster-master

指定实例：which-cluster-master

```bash
# orchestrator-client -c which-cluster-master -i test2:3307
test2:3307
# orchestrator-client -c which-cluster-master -i test
test2:3307

所有实例：all-clusters-masters，每个集群返回一个

# orchestrator-client -c all-clusters-masters
test1:3307
```

15.打印出所有实例：all-instances

```bash
# orchestrator-client -c all-instances
test2:3307
test1:3307
test3:3307
```

16.打印出集群中可以作为pt-online-schema-change操作的副本列表：which-cluster-osc-replicas 

```bash
~# orchestrator-client -c which-cluster-osc-replicas -i test
test1:3307
test3:3307
root@test1:~# orchestrator-client -c which-cluster-osc-replicas -i test2:3307
test1:3307
test3:3307
```

17.打印出集群中可以作为pt-online-schema-change可以操作的健康的副本列表：which-cluster-osc-running-replicas

```bash
# orchestrator-client -c which-cluster-osc-running-replicas -i test
test1:3307
test3:3307
# orchestrator-client -c which-cluster-osc-running-replicas -i test1:3307
test1:3307
test3:3307
```

18.打印出所有在维护（downtimed）的实例：downtimed

```bash
# orchestrator-client -c downtimed
test1:3307
test3:3307
```

19.打印出进群中主的数据中心：`dominant-dc`

```bash
# orchestrator-client -c dominant-dc
BJ
```
### 2.2 管理mysql集群
20.将集群的主提交到KV存储。`submit-masters-to-kv-stores`

```bash
# orchestrator-client -c submit-masters-to-kv-stores 
mysql/master/test:test2:3307
mysql/master/test/hostname:test2
mysql/master/test/port:3307
mysql/master/test/ipv4:192.168.163.132
mysql/master/test/ipv6:
```

21.迁移从库到另一个实例上：`relocate`

```bash
# orchestrator-client -c relocate -i test3:3307 -d test1:3307   #迁移test3:3307作为test1:3307的从库
test3:3307<test1:3307

查看
# orchestrator-client -c topology -i test2:3307
test2:3307     [0s,ok,5.7.25-0ubuntu0.16.04.2-log,rw,ROW,>>,GTID]
+ test1:3307   [0s,ok,5.7.25-0ubuntu0.16.04.2-log,ro,ROW,>>,GTID]
  + test3:3307 [0s,ok,5.7.25-log,ro,ROW,>>,GTID]
```

22.迁移一个实例的所有从库到另一个实例上：relocate-replicas

```bash
# orchestrator-client -c relocate-replicas -i test1:3307 -d test2:3307   #迁移test1:3307下的所有从库到test2:3307下，并列出被迁移的从库的实例名
test3:3307
```

23.将slave在拓扑上向上移动一级，对应web上的是在Classic Model下进行拖动：move-up

```bash
# orchestrator-client -c move-up -i test3:3307 -d test2:3307
test3:3307<test2:3307

结构从 test2:3307 -> test1:3307 -> test3:3307 
变成 
         / test1:3307
test2:3307
　　　　  \ test3:3307
```

24.将slave在拓扑上向下移动一级（移到同级的下面），对应web上的是在Classic Model下进行拖动：move-below

```bash
# orchestrator-client -c move-below -i test3:3307 -d test1:3307
test3:3307<test1:3307
 
结构从 
         / test1:3307
test2:3307
　　　　  \ test3:3307 
变成  test2:3307 -> test1:3307 -> test3:3307
```

25.将给定实例的所有从库在拓扑上向上移动一级，基于Classic Model模式：move-up-replicas

```bash
# orchestrator-client -c move-up-replicas -i test1:3307  
 test3:3307
 结构从 test2:3307 -> test1:3307 -> test3:3307 
 
 变成 
           / test1:3307
 test2:3307
　　　　    \ test3:3307 
```

26.创建主主复制，将给定实例直接和当前主库做成主主复制：make-co-master

```bash
# orchestrator-client -c make-co-master -i test1:3307
test1:3307<test2:3307
```

27.将实例转换为自己主人的主人，切换两个：take-master 

```bash
# orchestrator-client -c take-master -i test3:3307
test3:3307<test2:3307
结构从 test2:3307 -> test1:3307 -> test3:3307 变成 test2:3307 -> test3:3307 -> test1:3307
```

28. 通过GTID移动副本，move-gtid：通过orchestrator-client执行报错：

```bash
# orchestrator-client -c move-gtid -i test3:3307 -d test1:3307
parse error: Invalid numeric literal at line 1, column 9
parse error: Invalid numeric literal at line 1, column 9
parse error: Invalid numeric literal at line 1, column 9
通过orchestrator执行是没问题，需要添加--ignore-raft-setup参数：

# orchestrator -c move-gtid -i test3:3307 -d test2:3307 --ignore-raft-setup
test3:3307<test2:3307

```
29.通过GTID移动指定实例下的
所有slaves到另一个实例，move-replicas-gtid 通过orchestrator-client执行报错：

```bash
# orchestrator-client -c move-replicas-gtid -i test3:3307 -d test1:3307
jq: error (at <stdin>:1): Cannot index string with string "Key"
通过orchestrator执行是没问题，需要添加--ignore-raft-setup参数： 

# ./orchestrator -c move-replicas-gtid -i test2:3307 -d test1:3307 --ignore-raft-setup
test3:3307
```

30.将给定实例的同级slave，变更成他的slave，take-siblings

```bash
#orchestrator-client -c take-siblings -i test3:3307
test3:3307<test1:3307
结构从
          / test2:3307 
test1:3307 
          \ test3:3307
                 
变成  test1:3307 -> test3:3307 -> test2:3307　
```

　　　　　 　   
31.给指定实例打上标签，tag

```bash
# orchestrator-client -c tag -i test1:3307 --tag 'name=AAA'
test1:3307 
```

32.列出指定实例的标签，tags：

```bash
# orchestrator-client -c tags -i test1:3307
name=AAA 
```

33.列出给定实例的标签值：tag-value

```bash
# orchestrator-client -c tag-value -i test1:3307 --tag "name"
AAA
```

34.移除指定实例上的标签：untag

```bash
# orchestrator-client -c untag -i test1:3307 --tag "name=AAA"
test1:3307 
```

35.列出打过某个标签的实例，tagged：

```bash
#orchestrator-client -c tagged -t name
test3:3307
test1:3307
test2:3307
```

36.标记指定实例进入停用模式，包括时间、操作人、和原因，begin-downtime：

```bash
# orchestrator-client -c begin-downtime -i test1:3307 -duration=10m -owner=zjy -reason 'test'
test1:3307
```

37.移除指定实例的停用模式，end--downtime：

```bash
# orchestrator-client -c end-downtime -i test1:3307
test1:3307
```

38.请求指定实例上的维护锁：拓扑更改需要将锁放在最小受影响的实例上，以避免在同一个实例上发生两个不协调的操作，begin-maintenance ：

```bash
# orchestrator-client -c begin-maintenance -i test1:3307 --reason "XXX"
test1:3307
```

锁默认10分钟后过期，有参数MaintenanceExpireMinutes。

39.移除指定实例上的维护锁：end-maintenance

```bash
# orchestrator-client -c end-maintenance -i test1:3307
test1:3307
```

40.设置提升规则，恢复时可以指定一个实例进行提升：register-candidate：需要和promotion-rule一起使用

```bash
# orchestrator-client -c register-candidate -i test3:3307 --promotion-rule prefer 
test3:3307
```

提升test3:3307的权重，如果进行Failover，会成为Master。

41.指定实例执行停止复制：

普通的：stop slave：stop-replica

```bash
# orchestrator-client -c stop-replica -i test2:3307
test2:3307
```

应用完relay log，在stop slave：stop-replica-nice

```bash
# orchestrator-client -c stop-replica-nice -i test2:3307
test2:3307
```

42.指定实例执行开启复制： start-replica 

```bash
# orchestrator-client -c start-replica -i test2:3307
test2:3307
```

43.指定实例执行复制重启：restart-replica

```bash
# orchestrator-client -c restart-replica -i test2:3307
test2:3307
```

44.指定实例执行复制重置：reset-replica

```bash
# orchestrator-client -c reset-replica -i test2:3307
test2:3307
```

45.分离副本：非GTID修改binlog position，detach-replica ：

```bash
# orchestrator-client -c detach-replica -i test2:3307
```

46.恢复副本：reattach-replica 

```bash
# orchestrator-client -c reattach-replica  -i test2:3307
```

47.分离副本：注释masterhost来分离，detach-replica-master-host ：如MasterHost: //test1

```bash
# orchestrator-client -c detach-replica-master-host -i test2:3307
test2:3307
```

48.恢复副本：reattach-replica-master-host

```bash
# orchestrator-client -c reattach-replica-master-host -i test2:3307
test2:3307
```

49.跳过SQL线程的Query，如主键冲突，支持在GTID和非GTID下：skip-query 

```bash
# orchestrator-client -c skip-query -i test2:3307
test2:3307
```

50.将错误的GTID事务当做空事务应用副本的主上：gtid-errant-inject-empty「web上的fix」

```bash
# orchestrator-client -c gtid-errant-inject-empty  -i test2:3307
test2:3307 
```

51.通过RESET MASTER删除错误的GTID事务：gtid-errant-reset-master 

```bash
# orchestrator-client -c gtid-errant-reset-master  -i test2:3307
test2:3307
```

52.设置半同步相关的参数:

```bash
orchestrator-client -c $variable -i test1:3307

variable:
    enable-semi-sync-master      主上执行开启半同步
    disable-semi-sync-master      主上执行关闭半同步
    enable-semi-sync-replica       从上执行开启半同步
    disable-semi-sync-replica      从上执行关闭半同步
```

53.执行需要stop/start slave配合的SQL：restart-replica-statements

```bash
# orchestrator-client -c restart-replica-statements -i test3:3307 -query "change master to auto_position=1" | jq .[] -r 
stop slave io_thread;
stop slave sql_thread;
change master to auto_position=1;
start slave sql_thread;
start slave io_thread;

# orchestrator-client -c restart-replica-statements -i test3:3307 -query "change master to master_auto_position=1" | jq .[] -r  |  mysql -urep -p -htest3 -P3307
Enter password: 
```

54.根据复制规则检查实例是否可以从另一个实例复制(GTID和非GTID）： 非GTID，can-replicate-from： 

```bash
# orchestrator-client -c can-replicate-from -i test3:3307 -d test1:3307
test1:3307
GTID：can-replicate-from-gtid

# orchestrator-client -c can-replicate-from-gtid -i test3:3307 -d test1:3307
test1:3307 
```

55.检查指定实例是否在复制：is-replicating 

```bash
#有返回在复制
# orchestrator-client -c is-replicating -i test2:3307
test2:3307

#没有返回，不在复制
# orchestrator-client -c is-replicating -i test1:3307
```

56.检查指定实例的IO和SQL限制是否都停止： 

```bash
# orchestrator-client -c is-replicating -i test2:3307
```

57.将指定实例设置为只读，通过SET GLOBAL read_only=1，set-read-only：

```bash
# orchestrator-client -c set-read-only -i test2:3307
test2:3307
```

58.将指定实例设置为读写，通过`SET GLOBAL read_only=0，set-writeable`

```bash
# orchestrator-client -c set-writeable -i test2:3307
test2:3307
轮询指定实例的binary log，flush-binary-logs
# orchestrator-client -c flush-binary-logs -i test1:3307
test1:3307
```

60.执行自动恢复，指定一个死机的实例，`recover`：

```c
# orchestrator-client -c recover -i test2:3307
test3:3307
测试下来，该参数会让处理停机或则维护状态下的实例进行强制恢复。结构：

test1:3307 -> test2:3307 -> test3:3307(downtimed)  
当test2:3307死掉之后，此时test3:3307处于停机状态，不会进行Failover，
执行后变成

        / test2:3307
test1:3307
　　　　 \ test3:3307
```

61.优雅的进行主和指定从切换，`graceful-master-takeover`：

```bash
# orchestrator-client -c graceful-master-takeover -a test1:3307 -d test2:3307
test2:3307
结构从test1:3307 -> test2:3307 变成 test2:3307 -> test1:3307。新主指定变成读写，新从变成只读，还需要手动start slave。

注意需要配置：需要从元表里找到复制的账号和密码。

"ReplicationCredentialsQuery":"SELECT repluser, replpass from meta.cluster where anchor=1"
```

62.强行故障转移，即使orch没有发现问题，`force-master-failover`：转移之后老主独立，需要手动加入到集群。

```bash
# orchestrator-client -c force-master-failover -i test1:3307
test3:3307
```

63.强行丢弃master并指定的一个实例，`force-master-takeover`：老主(test1)独立，指定从(test2)提升为master

```bash
# orchestrator-client -c force-master-takeover -i test1:3307 -d test2:3307
test2:3307
```

64.确认集群恢复理由，在web上的Audit->Recovery->Acknowledged 按钮确认，/ack-all-recoveries 

确认指定集群：`ack-cluster-recoveries`

```bash
# orchestrator-client -c ack-cluster-recoveries  -i test2:3307 -reason=''
test1:3307
```

确认所有集群：`ack-all-recoveries` 

```bash
# orchestrator-client -c ack-all-recoveries  -reason='OOOPPP'
eason=XYZ
```

65.检查、禁止、开启orchestrator执行全局恢复：

检查：`check-global-recoveries`

```bash
# orchestrator-client -c check-global-recoveries
enabled

禁止：disable-global-recoveries
# orchestrator-client -c disable-global-recoveries
disabled

开启：enable--global-recoveries
# orchestrator-client -c enable-global-recoveries
enabled
```

66.检查分析复制拓扑中存在的问题：replication-analysis

```bash
# orchestrator-client -c replication-analysis
test1:3307 (cluster test1:3307): ErrantGTIDStructureWarning
```

### 2.3 管理orchestrator集群
67.raft检测：leader查看、健康监测、迁移leader：

查看leader节点

```bash
# orchestrator-client -c raft-leader
192.168.163.131:10008
```

健康监测

```bash
# orchestrator-client -c raft-health
healthy
```

leader 主机名

```bash
# orchestrator-client -c  raft-leader-hostname 
test1
```

指定主机选举leader

```bash
# orchestrator-client -c raft-elect-leader -hostname test3
test3
```

68.伪GTID相关参数：

```bash
match      #使用Pseudo-GTID指定一个从匹配到指定的另一个（目标）实例下
match-up   #Transport the replica one level up the hierarchy, making it child of its grandparent, using Pseudo-GTID
match-up-replicas  #Matches replicas of the given instance one level up the topology, making them siblings of given instance, using Pseudo-GTID
last-pseudo-gtid #Dump last injected Pseudo-GTID entry on a server
```

到此关于Orchestrator的使用以及命令行说明已经介绍完毕，API可以在Orchestrator上搜索学习，通过命令行和API上的操作可以更好的进行自动化开发。

参考链接：
[https://www.cnblogs.com/zhoujinyi/p/10394389.html](https://www.cnblogs.com/zhoujinyi/p/10394389.html)

