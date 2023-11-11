

![在这里插入图片描述](https://img-blog.csdnimg.cn/ca517cb396224d7a9628b7adc962024f.png)


## 1. 简介
这是一款`go`编写的MySQL高可用性和复制拓扑管理工具，支持复制拓扑结构的调整，自动故障转移和手动主从切换等。后端数据库用MySQL或SQLite存储元数据，并提供Web界面展示MySQL复制的拓扑关系及状态，通过Web可更改MySQL实例的复制关系和部分配置信息，同时也提供命令行和api接口，方便运维管理。

相对比MHA来看最重要的是解决了管理节点的单点问题，其通过raft协议保证本身的高可用。GitHub的一部分管理也在用该工具进行管理。

## 2. 特点

 - 自动发现MySQL的复制拓扑，并且在web上展示。
 - 提供了GUI，CLI，API等接口来检查复制拓扑的状态以及做一些调整的操作
 - 重构复制关系，可以在web进行拖图来进行复制关系变更。
 - 支持自动的master failover，当复制结构的server挂掉以后(不管手动还是自动的)，能够重新形成复制的拓扑结构
 - 检测主异常，并可以自动或手动恢复，通过Hooks进行自定义脚本。
 - 支持命令行和web界面管理复制

## 3. 安装mysql集群
环境

```bash
centos 7
192.168.211.60  master
192.168.211.61 slave
192.168.211.62 slave
```
[**centos本地部署mysql主从同步之gtid方法请点击**](https://ghostwritten.blog.csdn.net/article/details/111826719)

创建orchestrator的后台管理端数据库权限配置（主库执行）

```bash
CREATE DATABASE IF NOT EXISTS orchestrator;

GRANT ALL PRIVILEGES ON `orchestrator`.* TO 'orchestrator'@'%' IDENTIFIED BY '222222'; 
```
3台orch管理的数据库集群上，权限及数据库执行（3台数据库执行）

```bash
GRANT SUPER, PROCESS, REPLICATION SLAVE, RELOAD ON *.* TO 'orchestrator'@'%' IDENTIFIED BY '222222';
GRANT SELECT ON mysql.slave_master_info TO 'orchestrator'@'%';
```

创建测试库并授权

```bash
CREATE DATABASE IF NOT EXISTS meta;
GRANT SELECT ON meta.* TO 'orchestrator'@'%';
```
## 4. 安装Orchestrator(三个节点部署)
**Orchestrator下载地址**
[https://github.com/openark/orchestrator/releases](https://github.com/openark/orchestrator/releases)
```bash
$ yum -y install jq
$ rpm ivh orchestrator-3.2.3-1.x86_64.rpm
准备中...                          ################################# [100%]
正在升级/安装...
   1:orchestrator-1:3.2.3-1           ################################# [100%]
```

```bash
$ rpm -ql orchestrator
/etc/systemd/system/orchestrator.service
/usr/local/orchestrator/orchestrator
/usr/local/orchestrator/orchestrator-sample-sqlite.conf.json
/usr/local/orchestrator/orchestrator-sample.conf.json
/usr/local/orchestrator/resources/bin/orchestrator-client
.....
```
## 5. 配置Orchestrator
```bash
$ cd /usr/local/orchestrator/
cat orchestrator.conf.json 
{
  "Debug": true,
  "EnableSyslog": false,
  "ListenAddress": ":3000",
  "MySQLTopologyUser": "root",
  "MySQLTopologyPassword": "Admin@2018",
  "MySQLTopologyCredentialsConfigFile": "",
  "MySQLTopologySSLPrivateKeyFile": "",
  "MySQLTopologySSLCertFile": "",
  "MySQLTopologySSLCAFile": "",
  "MySQLTopologySSLSkipVerify": true,
  "MySQLTopologyUseMutualTLS": false,
  "MySQLOrchestratorHost": "127.0.0.1",
  "MySQLOrchestratorPort": 3306,
  "MySQLOrchestratorDatabase": "orchestrator",
  "MySQLOrchestratorUser": "orchestrator",
  "MySQLOrchestratorPassword": "Admin@2018",
  "MySQLOrchestratorCredentialsConfigFile": "",
  "MySQLOrchestratorSSLPrivateKeyFile": "",
  "MySQLOrchestratorSSLCertFile": "",
  "MySQLOrchestratorSSLCAFile": "",
  "MySQLOrchestratorSSLSkipVerify": true,
  "MySQLOrchestratorUseMutualTLS": false,
  "MySQLConnectTimeoutSeconds": 1,
  "MySQLTopologyReadTimeoutSeconds": 3,
  "MySQLDiscoveryReadTimeoutSeconds": 3,
  "DefaultInstancePort": 3306,
  "DiscoverByShowSlaveHosts": true,
  "InstancePollSeconds": 3,
  "UnseenInstanceForgetHours": 240,
  "SnapshotTopologiesIntervalHours": 0,
  "InstanceBulkOperationsWaitTimeoutSeconds": 10,
  "HostnameResolveMethod": "default",
  "MySQLHostnameResolveMethod": "@@hostname",
  "SkipBinlogServerUnresolveCheck": true,
  "SkipMaxScaleCheck":true,
  "ExpiryHostnameResolvesMinutes": 60,
  "RejectHostnameResolvePattern": "",
  "ReasonableReplicationLagSeconds": 10,
  "ProblemIgnoreHostnameFilters": [],
  "VerifyReplicationFilters": false,
  "ReasonableMaintenanceReplicationLagSeconds": 20,
  "CandidateInstanceExpireMinutes": 1440,
  "AuditLogFile": "",
  "AuditToSyslog": false,
  "RemoveTextFromHostnameDisplay": ":3306",
  "ReadOnly": false,
  "AuthenticationMethod": "",
  "HTTPAuthUser": "",
  "HTTPAuthPassword": "",
  "AuthUserHeader": "",
  "PowerAuthUsers": [
    "*"
  ],
  "ClusterNameToAlias": {
    "127.0.0.1": "test suite"
  },
  "SlaveLagQuery": "",
  "DetectClusterAliasQuery":  "SELECT cluster_name FROM meta.cluster WHERE cluster_name = left(@@hostname,4) ",
  "DetectClusterDomainQuery": "SELECT cluster_domain FROM meta.cluster WHERE cluster_name = left(@@hostname,4) ",
  "DetectInstanceAliasQuery": "SELECT @@hostname as instance_alias",
  "DetectPromotionRuleQuery": "",
  "DetectDataCenterQuery": "SELECT data_center FROM meta.cluster WHERE cluster_name = left(@@hostname,4) ",
  "PhysicalEnvironmentPattern": "",
  "PromotionIgnoreHostnameFilters": [],
  "DetachLostReplicasAfterMasterFailover": true,
  "DetectSemiSyncEnforcedQuery": "SELECT 0 AS semisync FROM DUAL WHERE NOT EXISTS (SELECT 1 FROM performance_schema.global_variables WHERE VARIABLE_NAME = 'rpl_semi_sync_master_wait_no_slave' AND VARIABLE_VALUE = 'ON') UNION SELECT 1 FROM DUAL WHERE EXISTS (SELECT 1 FROM performance_schema.global_variables WHERE VARIABLE_NAME = 'rpl_semi_sync_master_wait_no_slave' AND VARIABLE_VALUE = 'ON')",
  "ServeAgentsHttp": false,
  "AgentsServerPort": ":3001",
  "AgentsUseSSL": false,
  "AgentsUseMutualTLS": false,
  "AgentSSLSkipVerify": false,
  "AgentSSLPrivateKeyFile": "",
  "AgentSSLCertFile": "",
  "AgentSSLCAFile": "",
  "AgentSSLValidOUs": [],
  "UseSSL": false,
  "UseMutualTLS": false,
  "SSLSkipVerify": false,
  "SSLPrivateKeyFile": "",
  "SSLCertFile": "",
  "SSLCAFile": "",
  "SSLValidOUs": [],
  "URLPrefix": "",
  "StatusEndpoint": "/api/status",
  "StatusSimpleHealth": true,
  "StatusOUVerify": false,
  "AgentPollMinutes": 60,
  "UnseenAgentForgetHours": 6,
  "StaleSeedFailMinutes": 60,
  "SeedAcceptableBytesDiff": 8192,
  "AutoPseudoGTID":true,
  "PseudoGTIDPattern": "drop view if exists `meta`.`_pseudo_gtid_hint__asc:",
  "PseudoGTIDPatternIsFixedSubstring": true,
  "PseudoGTIDMonotonicHint": "asc:",
  "DetectPseudoGTIDQuery": "select count(*) as pseudo_gtid_exists from meta.pseudo_gtid_status where anchor = 1 and time_generated > now() - interval 2 hour",
  "BinlogEventsChunkSize": 10000,
  "SkipBinlogEventsContaining": [],
  "ReduceReplicationAnalysisCount": true,
  "FailureDetectionPeriodBlockMinutes": 60,
  "RecoveryPeriodBlockSeconds": 31,
  "RecoveryIgnoreHostnameFilters": [],
  "RecoverMasterClusterFilters": ["*"],
  "RecoverIntermediateMasterClusterFilters": ["*"],
  "OnFailureDetectionProcesses": [
    "echo '②  Detected {failureType} on {failureCluster}. Affected replicas: {countSlaves}' >> /tmp/recovery.log"
  ],
  "PreGracefulTakeoverProcesses": [
    "echo '①   Planned takeover about to take place on {failureCluster}. Master will switch to read_only' >> /tmp/recovery.log"
  ],
  "PreFailoverProcesses": [
    "echo '③  Will recover from {failureType} on {failureCluster}' >> /tmp/recovery.log"
  ],
  "PostMasterFailoverProcesses": [
    "echo '④  Recovered from {failureType} on {failureCluster}. Failed: {failedHost}:{failedPort}; Promoted: {successorHost}:{successorPort}' >> /tmp/recovery.log"
  ],
  "PostFailoverProcesses": [
    "echo '⑤  (for all types) Recovered from {failureType} on {failureCluster}. Failed: {failedHost}:{failedPort}; Successor: {successorHost}:{successorPort}' >> /tmp/recovery.log"
  ],
  "PostUnsuccessfulFailoverProcesses": [
    "echo '⑧  >> /tmp/recovery.log'"
  ],
  "PostIntermediateMasterFailoverProcesses": [
    "echo '⑥ Recovered from {failureType} on {failureCluster}. Failed: {failedHost}:{failedPort}; Successor: {successorHost}:{successorPort}' >> /tmp/recovery.log"
  ],
  "PostGracefulTakeoverProcesses": [
    "echo '⑦ Planned takeover complete' >> /tmp/recovery.log"
  ],
  "CoMasterRecoveryMustPromoteOtherCoMaster": true,
  "DetachLostSlavesAfterMasterFailover": true,
  "ApplyMySQLPromotionAfterMasterFailover": true,
  "PreventCrossDataCenterMasterFailover": false,
  "MasterFailoverDetachSlaveMasterHost": false,
  "MasterFailoverLostInstancesDowntimeMinutes": 0,
  "PostponeSlaveRecoveryOnLagMinutes": 0,
  "OSCIgnoreHostnameFilters": [],
  "GraphiteAddr": "",
  "GraphitePath": "",
  "GraphiteConvertHostnameDotsToUnderscores": true,

  "RaftEnabled": true,
  "BackendDB": "mysql",
  "RaftBind": "192.168.211.60",
  "RaftDataDir": "/var/lib/orchestrator",
  "DefaultRaftPort": 10008,
  "RaftNodes": [
    "192.168.211.60",
    "192.168.211.61",
    "192.168.211.62"
    ],
 "ConsulAddress": "",
 "ConsulAclToken": ""
}

```
## 6. 启动测试
```bash
$ cd /usr/local/orchestrator
$ ./orchestrator http
2020-12-28 11:32:59 DEBUG Connected to orchestrator backend: orchestrator:?@tcp(127.0.0.1:3306)/orchestrator?timeout=1s
2020-12-28 11:32:59 DEBUG Orchestrator pool SetMaxOpenConns: 128
2020-12-28 11:32:59 DEBUG Initializing orchestrator
2020-12-28 11:32:59 INFO Connecting to backend 127.0.0.1:3306: maxConnections: 128, maxIdleConns: 32
2020-12-28 11:32:59 INFO Starting Discovery
2020-12-28 11:32:59 INFO Registering endpoints
2020-12-28 11:32:59 INFO continuous discovery: setting up
2020-12-28 11:32:59 INFO continuous discovery: starting
2020-12-28 11:32:59 DEBUG Queue.startMonitoring(DEFAULT)
2020-12-28 11:32:59 INFO Starting HTTP listener on :3000
2020-12-28 11:33:01 DEBUG Waiting for 15 seconds to pass before running failure detection/recovery
2020-12-28 11:33:02 DEBUG Waiting for 15 seconds to pass before running failure detection/recovery
2020-12-28 11:33:03 DEBUG Waiting for 15 seconds to pass before running failure detection/recovery
2020-12-28 11:33:04 DEBUG Waiting for 15 seconds to pass before running failure detection/recovery
2020-12-28 11:33:05 DEBUG Waiting for 15 seconds to pass before running failure detection/recovery
2020-12-28 11:33:06 DEBUG Waiting for 15 seconds to pass before running failure detection/recovery
2020-12-28 11:33:07 DEBUG Waiting for 15 seconds to pass before running failure detection/recovery
2020-12-28 11:33:08 DEBUG Waiting for 15 seconds to pass before running failure detection/recovery
2020-12-28 11:33:09 DEBUG Waiting for 15 seconds to pass before running failure detection/recovery
2020-12-28 11:33:10 DEBUG Waiting for 15 seconds to pass before running failure detection/recovery
[martini] Started GET / for 192.168.211.1:53587
[martini] Completed 302 Found in 974.579?s
[martini] Started GET /web/clusters for 192.168.211.1:53587
[martini] Completed 200 OK in 1.788032ms

```
点击发现服务
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201228114438278.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
加入mysql集群
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201228114228915.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

查看配置
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201227210138836.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

实例说明

```bash
Instance Alias ：实例别名
Last seen       :  最后检测时间
Self coordinates ：自身的binlog位点信息
Num replicas ：有几个从库
Server ID    ： MySQL server_id
Server UUID ：    MySQL UUID
Version ：    版本
Read only ： 是否只读
Has binary logs ：是否开启binlog
Binlog format    ：binlog 模式
Logs slave updates ：是否开启log_slave_updates
GTID supported ：是否支持GTID
GTID based replication ：是否是基于GTID的复制
GTID mode    ：复制是否开启了GTID
Executed GTID set ：复制中执行过的GTID列表
Uptime ：启动时间
Allow TLS ：是否开启TLS
Cluster ：集群别名
Audit ：审计实例
Agent ：Agent实例

其中Begin Downtime 会将实例标记为已停用，此时如果发生Failover，该实例不会参与。
```
任意改变主从的拓扑结构：可以直接在图上拖动变更复制，会自动恢复拓扑关系： 
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201227210751686.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
3.主库挂了之后自动Failover，如：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201227210811679.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
图中显示，当主挂掉之后，拓扑结构里自动剔除该主节点，选择一个最合适的从库提升成主库，并修复复制拓扑。在Failover过程当中，可以查看/tmp/recovery.log文件（配置文件里定死），里面包含了在Failover过程中Hooks执行的外部脚本，类似MHA的master_ip_failover_script参数。可以通过外部脚本进行相应的如：VIP切换、Proxy修改、DNS修改、中间件修改、LVS修改等等，具体的执行脚本可以根据自己的实际情况编写。

4.Orchestrator高可用。因为在一开始就已经部署了3台，通过配置文件里的Raft参数进行通信。只要有2个节点的Orchestrator正常，就不会影响使用，如果出现2个节点的Orchestrator异常，则Failover会失败。2个节点异常的图如下：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201227211332187.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
图中的各个节点全部显示灰色，此时Raft算法失效，导致Orch的Failover功能失败。相对比MHA的Manager的单点，Orchestrator通过Raft算法解决了本身的高可用性以及解决网络隔离问题，特别是跨数据中心网络异常。这里说明下Raft，通过共识算法：Orchestrator节点能够选择具有仲裁的领导者（leader）。如有3个orch节点，其中一个可以成为leader（3节点仲裁大小为2，5节点仲裁大小为3）。只允许leader进行修改，每个MySQL拓扑服务器将由三个不同的orchestrator节点独立访问，在正常情况下，三个节点将看到或多或少相同的拓扑图，但他们每个都会独立分析写入其自己的专用后端数据库服务器：

 - ① 所有更改都必须通过leader。
 - ② 在启用raft模式上禁止使用orchestrator客户端。
 - ③在启用raft模式上使用orchestrator-client，orchestrator-client可以安装在没有orchestrator上的服务器。
 - ④单个orchestrator节点的故障不会影响orchestrator的可用性。在3节点设置上，最多一个服务器可能会失败。在5节点设置上，2个节点可能会失败。
 - ⑤ Orchestrator节点异常关闭，然后再启动。它将重新加入Raft组，并接收遗漏的任何事件,只要有足够的Raft记录。
 - ⑥ 要加入比日志保留允许的更长/更远的orchestrator节点或者数据库完全为空的节点，需要从另一个活动节点克隆后端DB。
关于Raft更多的信息见：[https://github.com/github/orchestrator/blob/master/docs/raft.md](https://github.com/github/orchestrator/blob/master/docs/raft.md)
Orchestrator的高可用有2种方式，第一种就是上面说的通过Raft（推荐），另一种是通过后端数据库的同步。详细信息见文档。文档里详细比较了两种高可用性部署方法。两种方法的图如下：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201227212417471.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
到这里，Orchestrator的基本功能已经实现，包括主动Failover、修改拓扑结构以及Web上的可视化操作。

## 7.Web上各个按钮的功能说明

 - ①：Home下的status：查看orch的状态：包括运行时间、版本、后端数据库以及各个Raft节点的状态。
 - ②：Cluster下的dashboard：查看orch下的所有被管理的MySQL实例。
 - ③：Cluster下的Failure analysis：查看故障分析以及包括记录的故障类型列表。
 - ④：Cluster下的Discover：用来发现被管理的MySQL实例。
 - ⑤：Audit下的Failure detection：故障检测信息，包含历史信息。
 - ⑥：Audit下的Recovery：故障恢复信息以及故障确认。
 - ⑦：Audit下的Agent：是一个在MySQL主机上运行并与orchestrator通信的服务，能够向orch提供操作系统，文件系统和LVM信息，以及调用某些命令和脚本。
 - ⑧：导航栏里的图标，对应左边导航栏的图标
![5.Web上各个按钮的功能说明](https://img-blog.csdnimg.cn/20201227214813389.png)

第1行：集群别名的查看修改。
第2行：pools。
第3行：Compact display，紧凑展示
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201227214851840.png)
第4行：Pool indicator，池指示器
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201227214910924.png)
第5行：Colorize DC，每个数据中心用不同颜色展示
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201227214926427.png)
第6行：Anonymize，匿名集群中的主机名
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201227214949446.png)
注意：左边导航栏里的图标
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201227215005744.png)
表示实例的概括：实例名、别名、故障检测和恢复等信息。

⑧：导航栏里的图标
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201227215803986.png)
表示是否禁止全局恢复。禁止掉的话不会进行Failover。

⑨：导航栏里的图标
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201227215820594.png)
,表示是否开启刷新页面（默认60一次）。

⑩：导航栏里的图标
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020122721584393.png)
，表示MySQL实例迁移模式。

Smart mode：自动选择迁移模式，让Orch自己选择迁移模式。
Classic mode：经典迁移模式，通过binlog和position进行迁移。
GTID mode：GTID迁移模式。
Pseudo GTID mode：伪GTID迁移模式。
到此，Orchestrator的基本测试和Web说明已经介绍完毕。和MHA比已经有很大的体验提升，不仅在Web进行部分参数的设置修改，还可以改变复制拓扑，最重要的是解决MHA Manager单点的问题。还有什么理由不替换MHA呢？:)

## 8. 工作流程说明
Orchestrator实现了自动`Failover`，现在来看看自动Failover的大致流程是怎么样的。

1.检测流程

 - ① orchestrator利用复制拓扑，先检查主本身，并观察其slaves。
 - ② 如果orchestrator本身连不上主，可以连上该主的从，则通过从去检测，若在从上也看不到主（IOThread）「2次检查」，判断Master宕机。

**该检测方法比较合理，当从都连不上主了，则复制肯定有出问题，故会进行切换。所以在生产中非常可靠。**

检测发生故障后并不都会进行自动恢复，比如：禁止全局恢复、设置了shutdown time、上次恢复离本次恢复时间在RecoveryPeriodBlockSeconds设置的时间内、失败类型不被认为值得恢复等。检测与恢复无关，但始终启用。 每次检测都会执行OnFailureDetectionProcesses Hooks。

配置故障检测：

```bash
{
  "FailureDetectionPeriodBlockMinutes": 60,
}

Hooks相关参数：
{
  "OnFailureDetectionProcesses": [
    "echo 'Detected {failureType} on {failureCluster}. Affected replicas: {countReplicas}' >> /tmp/recovery.log"
  ],
}

MySQL复制相关调整：
slave_net_timeout
MASTER_CONNECT_RETRY
```
2. 恢复流程

恢复的实例需要支持：GTID、伪GTID、开启Binlog。恢复的配置如下：

```bash
{
  "RecoveryPeriodBlockSeconds": 3600,
  "RecoveryIgnoreHostnameFilters": [],
  "RecoverMasterClusterFilters": [
    "thiscluster",
    "thatcluster"
  ],
  "RecoverMasterClusterFilters": ["*"],
  "RecoverIntermediateMasterClusterFilters": [
    "*"
  ],
}

{
  "ApplyMySQLPromotionAfterMasterFailover": true,
  "PreventCrossDataCenterMasterFailover": false,
  "FailMasterPromotionIfSQLThreadNotUpToDate": true,
  "MasterFailoverLostInstancesDowntimeMinutes": 10,
  "DetachLostReplicasAfterMasterFailover": true,
}

Hooks：
{
  "PreGracefulTakeoverProcesses": [
    "echo 'Planned takeover about to take place on {failureCluster}. Master will switch to read_only' >> /tmp/recovery.log"
  ],
  "PreFailoverProcesses": [
    "echo 'Will recover from {failureType} on {failureCluster}' >> /tmp/recovery.log"
  ],
  "PostFailoverProcesses": [
    "echo '(for all types) Recovered from {failureType} on {failureCluster}. Failed: {failedHost}:{failedPort}; Successor: {successorHost}:{successorPort}' >> /tmp/recovery.log"
  ],
  "PostUnsuccessfulFailoverProcesses": [],
  "PostMasterFailoverProcesses": [
    "echo 'Recovered from {failureType} on {failureCluster}. Failed: {failedHost}:    {failedPort}; Promoted: {successorHost}:{successorPort}' >> /tmp/recovery.log"
  ],
  "PostIntermediateMasterFailoverProcesses": [],
  "PostGracefulTakeoverProcesses": [
    "echo 'Planned takeover complete' >> /tmp/recovery.log"
  ],
}
```
具体的参数含义请参考「MySQL高可用复制管理工具:Orchestrator介绍」。在执行故障检测和恢复的时候都可以执行外部自定义脚本（hooks），来配合使用（VIP、Proxy、DNS）。

可以恢复中继主库（DeadIntermediateMaster）和主库：

中继主库：恢复会找其同级的节点进行做主从。匹配副本按照哪些实例具有log-slave-updates、实例是否延迟、它们是否具有复制过滤器、哪些版本的MySQL等等

主库：恢复可以指定提升特定的从库「提升规则」（register-candidate），提升的从库不一定是最新的，而是选择最合适的，设置完提升规则之后，有效期为1个小时。

提升规则选项有：

```bash
prefer     --比较喜欢
neutral    --中立（默认）
prefer_not --比较不喜欢
must_not   --拒绝
```
恢复支持的类型有：自动恢复、优雅的恢复、手动恢复、手动强制恢复，恢复的时候也可以执行相应的Hooks参数。具体的恢复流程可以看恢复流程的说明。关于恢复的配置可以官方说明。






参考链接“
- [https://cloud.tencent.com/developer/article/1418017](https://cloud.tencent.com/developer/article/1418017)


相关阅读：
- [**Orchestrator (3) orchestrator-client命令详解**](https://ghostwritten.blog.csdn.net/article/details/111881197)
- [**Orchestrator (2) 配置参数详解**](https://ghostwritten.blog.csdn.net/article/details/111881286)
- [**Orchestrator  (1) 高可用管理详解**](https://ghostwritten.blog.csdn.net/article/details/106099648)
- [**centos本地部署mysql主从同步之gtid方法**](https://ghostwritten.blog.csdn.net/article/details/111826719)
