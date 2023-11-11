## 1. key/value作用

 - 动态修改配置文件
 - 支持服务协同
 - 建立leader选举
 - 提供服务发现
 - 集成健康检查

除了提供服务发现和综合健康检查,Consul还提供了一个易于使用的键/值存储。这可以用来保存动态配置,协助服务协调,建立领导人选举,并启用其他开发人员可以想构建的任何其他内容。

有两种方法可以使用:通过HTTP API和通过CLI API。

## 2. CLI API操作key/value



### 2.1 consul kv put增加key/value

```bash
$ consul kv put redis/config/minconns 1
```
###  2.2 consul kv get 查询

```bash
###  consul kv get 查询
```
```bash
consul kv get redis/config/minconns
1
```
Consul保留额外的元数据在该字段,你可以使用-detailed标志检索详细信息:

```bash
$ consul kv get -detailed redis/config/minconns
CreateIndex      74
Flags            0
Key              redis/config/minconns
LockIndex        0
ModifyIndex      74
Session          -
Value            1
```
在web UI上可以看到用CLI API创建的key
mV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70#pic_center)
在web UI上创建一个“duan”的key：
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020082117491280.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70#pic_center)
在web UI上创建一个“duan”的key：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200821175006908.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70#pic_center)

```bash
/ # consul kv get redis/spectre
1234
```
flags用来做客户端自定义标志，consul并不使用它，你可以在你自己的程序中随便定义
设置flag值为42，想设置成什么就设置成什么．所有的键都支持设置一个64位的整型值。
```bash
/ # consul kv put -flags=42 redis/config/users/admin abcd1234
/ # consul kv get redis/config/users/admin 
abcd1234

```
### 2.3 consul kv get -recurse 列表查询

```bash
/ # consul kv get -recurse
redis/config/minconns:1
redis/config/users/admin:abcd1234
redis/spectre:1234
```
### 2.4  consul kv delete删除

```bash
/ # consul kv delete redis/config/minconns
Success! Deleted key: redis/config/minconns
/ # consul kv get -recurse
redis/config/users/admin:abcd1234
redis/spectre:1234
```
还可以使用recurse选项递归选项删除含某个前缀的所有keys:

```bash
/ # consul kv delete -recurse redis
Success! Deleted keys with prefix: redis
/ # consul kv get -recurse
```
如果要更新一个存在键的值，可以put一个新值在同样的路径上。

```bash
/ # consul kv put foo bar
Success! Data written to: foo
/ # consul kv get foo
bar
/ # consul kv put foo zip
Success! Data written to: foo
/ # consul kv get foo
zip
```
Consul可以使用Check_And_Set提供原子键更新操作。执行CAS操作时需指定-cas标志。

```bash
-modify-index=<uint>
```

```bash
/ # consul kv get -detailed foo
CreateIndex      100
Flags            0
Key              foo
LockIndex        0
ModifyIndex      102
Session          -
Value            zip
```
看到foo的索引编号ModifyIndex是102。然后使用CAS操作的方式来修改它

```bash
/ # consul kv put -cas -modify-index=102 foo bar
Success! Data written to: foo
/ # consul kv get -detailed foo
CreateIndex      100
Flags            0
Key              foo
LockIndex        0
ModifyIndex      111
Session          -
Value            bar
```
## 3 http API操作key/value
### 3.1 查看key/value
```bash
/ # curl  http://127.0.0.1:8500/v1/kv/?recurse
[
    {
        "LockIndex": 0,
        "Key": "foo",
        "Flags": 0,
        "Value": "YmFy",
        "CreateIndex": 100,
        "ModifyIndex": 111
    }
]
```
说明：

 - 使用?recurse参数来指定查看多个KV
 - 没有值--404

### 3.2 添加key/value

```bash
/ # curl -X PUT -d 'test' http://127.0.0.1:8500/v1/kv/web/config
true
```

```bash
key--->web/config
vaule----> test
```

```bash
/ # curl -X PUT -d 'test' http://127.0.1:8500/v1/kv/web/config2?flag=33
true

/ # curl -X GET http://127.0.1:8500/v1/kv/web/config2  #查看单个key/value
[
    {
        "LockIndex": 0,
        "Key": "web/config2",
        "Flags": 0,
        "Value": "dGVzdA==",
        "CreateIndex": 183,
        "ModifyIndex": 183
    }
]
/ # 

```
说明：flags--用于为任意一个KV添加一个有意义的metadata。

注意：上边的这个就是有问题的，一定要注意是flags而非flag

### 3.3 修改key/value

```bash
/ # curl -X PUT -d 'testadmin' http://127.0.0.1:8500/v1/kv/web/config
true
```
cas修改（原子性修改）

```bash
/ # curl http://127.0.0.1:8500/v1/kv/web/config
[
    {
        "LockIndex": 0,
        "Key": "web/config",
        "Flags": 0,
        "Value": "dGVzdGFkbWlu",
        "CreateIndex": 171,
        "ModifyIndex": 202
    }
]

/ # curl -X PUT -d 'testadmin12345' http://127.0.0.1:8500/v1/kv/web/config?cas=202
true


```
### 3.4 删除key/value

```bash
/ # curl -X DELETE http://127.0.0.1:8500/v1/kv/web/config
true
```
删除一定范围的KV（指定前缀范围内的KV）

```bash
/ # curl -X DELETE http://127.0.0.1:8500/v1/kv/web/config2/?recurse
true
```
说明：

 - 指定删除的KV的K的前缀（zjg）
 - 多个操作一定要有?recurse参数


