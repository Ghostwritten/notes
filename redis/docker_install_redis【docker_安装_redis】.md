
# 1. 创建 Redis 配置文件
在本地目录 /data/redis/config 下创建一个 redis.conf 配置文件，用于指定 Redis 配置项（包括用户密码）。
你可以使用以下命令创建该目录：

```bash
$ mkdir -p /data/redis/config
```

生成强密码

```bash
$ openssl rand -base64 16
KRzjQRNHlVEdv94DZtsN5Q==
```

编辑配置文件
在 /data/redis/config/redis.conf 中添加以下内容，启用密码认证并设置用户：

```bash
$ vim /data/redis/config/redis.conf
# 设置密码
requirepass KRzjQRNHlVEdv94DZtsN5Q==
```

# 2. 运行

你可以使用以下命令，通过 Podman 运行一个 Redis 容器，并将数据持久化到本地目录 /data/redis，同时将容器的 Redis 端口映射到本地端口 10102：

```bash
podman run -d \
  --name redis-server \
  -p 10102:6379 \
  -v /data/redis:/data \
  -v /data/redis/config/redis.conf:/usr/local/etc/redis/redis.conf:Z \
  redis:7.4.1 \
  redis-server /usr/local/etc/redis/redis.conf --appendonly yes --dir /data

```

解释：

 • -d: 以后台模式运行容器。
 • --name redis-server: 给容器命名为 redis-server。
 • -p 10102:6379: 将容器内的 Redis 端口 6379 映射到本地的 10102 端口。
 • -v /data/redis:/data: 将本地目录 /data/redis 挂载到容器内的 /data 目录，用于持久化 Redis 数据。
 • redis:latest: 使用 Redis 的最新镜像。
 • --appendonly yes --dir /data: 让 Redis 启动时开启持久化（appendonly 模式），并将数据保存到 /data 目录中。
 • -v /data/redis/config/redis.conf:/usr/local/etc/redis/redis.conf:Z：将本地 Redis 配置文件挂载到容器中 Redis 的默认配置路径。:Z 标志确保正确的 SELinux 标签以保证 Podman 的权限访问。

运行成功后，Redis 将使用该配置文件，且需要密码验证才能访问。

# 3. 测试
确保 /data/redis 目录存在且有合适的权限，否则 Redis 容器可能无法正确写入数据。

```bash
$ redis-cli -h 10.251.26.77 -p 10102 -a KRzjQRNHlVEdv94DZtsN5Q==
```

