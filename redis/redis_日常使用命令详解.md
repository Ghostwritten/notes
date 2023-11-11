使用KEYS 命令

```csharp
KEYS pattern
1
```

例如，

列出所有的key

```csharp
redis> keys *
1
```

列出匹配的key

```csharp
redis>keys apple*
1) apple1
2) apple2
```

修改密码

```csharp
$ config set requirepass p@ss$12E45
```

```csharp
config get timeout
client list
```

redis内存扩容

```csharp
redis-cli -h 10.253.32.194 -p 3009 -a "......"
> config set maxmemory 7g
> config rewrite   #生效
> config get maxmemory  #验证
> info memory   #验证
```
### slaveof
`SLAVEOF` 命令用于在 Redis 运行时动态地修改复制(replication)功能的行为。

 - `SLAVEOF host port` :可以将当前服务器转变为指定服务器的从属服务器(slave
   server)。如果当前服务器已经是某个主服务器(master server)的从属服务器，那么执行 SLAVEOF host port将使当前服务器停止对旧主服务器的同步，丢弃旧数据集，转而开始对新主服务器进行同步。
 - `SLAVEOF NO ONE` :将使得这个从属服务器关闭复制功能，并从从属服务器转变回主服务器，原来同步所得的数据集不会被丢弃。利用SLAVEOF NO ONE不会丢弃同步所得数据集』这个特性，可以在主服务器失败的时候，将从属服务器用作新的主服务器，从而实现无间断运行。

可用版本：

```csharp
>= 1.0.0
```

时间复杂度：

```csharp
SLAVEOF host port ，O(N)， N 为要同步的数据数量。
SLAVEOF NO ONE ， O(1) 。
```

返回值：
总是返回 OK 。

```csharp
redis> SLAVEOF 127.0.0.1 6379
OK

redis> SLAVEOF NO ONE
OK
```

