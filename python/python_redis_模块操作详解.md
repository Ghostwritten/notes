#  python 模块 redis



---
## 1. 简介
redis 是一个 Key-Value 数据库，Value 支持 string(字符串)，list(列表)，set(集合)，zset(有序集合)，hash(哈希类型)等类型。默认端口号为 `6379`
## 2. 安装 redis 模块

```python
sudo pip3 install redis
或
sudo easy_install redis
或
sudo python setup.py install
```
源码地址：[https://github.com/WoLpH/redis-py](https://github.com/WoLpH/redis-py)

测试是否安装成功：

```bash
>>> import redis
>>> r = redis.StrictRedis(host='localhost', port=6379, db=0)
>>> r.set('foo', 'bar')
True
>>> r.get('foo')
'bar'
```
## 3. 方法
### 3.1 redis.Redis
redis 提供两个类 `Redis` 和 `StrictRedis`, StrictRedis 用于实现大部分官方的命令，Redis 是 StrictRedis 的子类，用于向后兼用旧版本。

redis 取出的结果默认是字节，我们可以设定 `decode_responses=True` 改成字符串。

red1.py
```bash
import redis   # 导入redis 模块

r = redis.Redis(host='localhost', port=6379, passwd='123456',socket_connect_timeout=1, decode_responses=True)  
r.set('name', 'runoob')  # 设置 name 对应的值
print(r['name'])
print(r.get('name'))  # 取出键 name 对应的值
print(type(r.get('name')))  # 查看类型
r.keys()   # 列出所有键值
r.delete('name')   #删除key为name得值
r.dbsize()     #查看库里有多少key
r.save()   #强行把数据库保存到硬盘。保存时阻塞
r.flushdb()   #删除当前数据库的所有数据
```

### 3.2 连接池
redis-py 使用 `connection pool` 来管理对一个 redis server 的所有连接，避免每次建立、释放连接的开销。

默认，每个Redis实例都会维护一个自己的连接池。可以直接建立一个连接池，然后作为参数 Redis，这样就可以实现多个 Redis 实例共享一个连接池。

```bash
import redis    # 导入redis 模块

pool = redis.ConnectionPool(host='localhost', port=6379, decode_responses=True)
r = redis.Redis(host='localhost', port=6379, decode_responses=True)  
r.set('name', 'runoob')  # 设置 name 对应的值
print(r.get('name'))  # 取出键 name 对应的值
```

### 3.3 管道
redis默认在执行每次请求都会创建（连接池申请连接）和断开（归还连接池）一次连接操作，如果想要在一次请求中指定多个命令，则可以使用pipline实现一次请求指定多个命令，并且默认情况下一次pipline 是原子性操作。

管道（pipeline）是redis在提供单个请求中缓冲多条服务器命令的基类的子类。它通过减少服务器-客户端之间反复的TCP数据库包，从而大大提高了执行批量命令的功能。

```python
import redis
import time

pool = redis.ConnectionPool(host='localhost', port=6379, decode_responses=True)
r = redis.Redis(connection_pool=pool)

# pipe = r.pipeline(transaction=False)    # 默认的情况下，管道里执行的命令可以保证执行的原子性，执行pipe = r.pipeline(transaction=False)可以禁用这一特性。
# pipe = r.pipeline(transaction=True)
pipe = r.pipeline() # 创建一个管道

pipe.set('name', 'jack')
pipe.set('role', 'sb')
pipe.sadd('faz', 'baz')
pipe.incr('num')    # 如果num不存在则vaule为1，如果存在，则value自增1
pipe.execute()

print(r.get("name"))
print(r.get("role"))
print(r.get("num"))
```
管道的命令可以写在一起，如：

```python
pipe.set('hello', 'redis').sadd('faz', 'baz').incr('num').execute()
print(r.get("name"))
print(r.get("role"))
print(r.get("num"))
```


### 3.4 redis 基本命令 String

```bash
set(name, value, ex=None, px=None, nx=False, xx=False)
```

在 Redis 中设置值，默认，不存在则创建，存在则修改。

参数：

 - ex - 过期时间（秒）
 - px - 过期时间（毫秒）
 - nx - 如果设置为True，则只有name不存在时，当前set操作才执行
 - xx - 如果设置为True，则只有name存在时，当前set操作才执行
 
 1.ex - 过期时间（秒） 这里过期时间是3秒，3秒后p，键food的值就变成None

```bash
import redis

pool = redis.ConnectionPool(host='localhost', port=6379, decode_responses=True)
r = redis.Redis(connection_pool=pool)
r.set('food', 'mutton', ex=3)    # key是"food" value是"mutton" 将键值对存入redis缓存
print(r.get('food'))  # mutton 取出键food对应的值
```
2.px - 过期时间（豪秒） 这里过期时间是3豪秒，3毫秒后，键foo的值就变成None

```bash
import redis

pool = redis.ConnectionPool(host='localhost', port=6379, decode_responses=True)
r = redis.Redis(connection_pool=pool)
r.set('food', 'beef', px=3)
print(r.get('food'))
```

3.nx - 如果设置为True，则只有name不存在时，当前set操作才执行 （新建）

```bash
import redis

pool = redis.ConnectionPool(host='localhost', port=6379, decode_responses=True)
r = redis.Redis(connection_pool=pool)
print(r.set('fruit', 'watermelon', nx=True))    # True--不存在
# 如果键fruit不存在，那么输出是True；如果键fruit已经存在，输出是None
```

4.xx - 如果设置为True，则只有name存在时，当前set操作才执行 （修改）

```bash
print((r.set('fruit', 'watermelon', xx=True)))   # True--已经存在
# 如果键fruit已经存在，那么输出是True；如果键fruit不存在，输出是None
```
参考：

 - [python redis命令操作更多](https://www.runoob.com/w3cnote/python-redis-intro.html)
 - [How to Use Redis With Python](https://realpython.com/python-redis/)
 - [Welcome to redis-py’s documentation!](https://redis-py.readthedocs.io/en/stable/)
 - [Python操作Redis，你要的都在这了！](https://cloud.tencent.com/developer/article/1151834)

