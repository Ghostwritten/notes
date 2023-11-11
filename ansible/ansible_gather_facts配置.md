


---
## 1. 过滤指定的fact

```bash
ansible localhost -m setup -a 'filter=ansible_eth*’
```

## 2. 自定义fact
对 facts 设置优化
ansible playbook 默认第一个 task 是 Gathering Facts 收集各主机的 facts 信息，以方便我们在 paybook 中直接引用 facts 里的信息。

如果不需要用到 facts 信息的话，可以设置 `gather_facts: false`，来省去 facts 采集这一步以提高 playbook 效率。

如果既想用 facts 信息，有希望能提高 playbook 的效率的话，可以采用 facts 缓存来实现。
facts 缓存支持多种方式：json 文件方式，redis 方式，memcache 方式等。各种方式的配置都是在 ansible.cfg 中配置。

### 2.1 json 文件方式

```bash
gathering=smart
fact_caching_timeout=86400
fact_caching=jsonfile
fact_caching_connection=/path/to/ansible_fact_cache
```

这里的 86400 单位为秒，表示缓存的过期时间。保存 facts 信息的 json 文件保存在 `/ path/to/ansible_fact_cache` 下面，文件名是按照 `inventory hostname` 来命名的。

### 2.2 redis 方式

```bash
gathering=smart
fact_caching_timeout=86400
fact_caching=redis
```

需要注意的是，facts 存储不支持远端的 redis，需要在 ansible 的控制服务器上安装 redis；同时，还需要安装 python 的 redis 模块。

### 2.3 memcache 方式

```bash
gathering=smart
fact_caching_timeout=86400
fact_caching=memcached
```

与 redis 方式类似，需要运行 memcached 服务，同时，安装 python 的 memcached 依赖包。
