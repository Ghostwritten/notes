![](https://i-blog.csdnimg.cn/blog_migrate/4f810c83d3f1b3a8c44d79d57a5a1908.jpeg)




参考：

- 官网地址：https://nlnetlabs.nl/projects/unbound/about/

- 详细文档：https://unbound.docs.nlnetlabs.nl/en/latest/index.html
- DNS服务Unbound部署于使用 
- [https://cloud.tencent.com/developer/article/2314579](https://cloud.tencent.com/developer/article/2314579)

## 安装

Centos

```bash
yum install -y unbound
```

ubuntu

```bash
apt install unbound -y
```

## unbound-control
远程管理服务

格式
unbound-control [-h] [-c cfgfile] [-s server] command

- -h 显示版本和命令行选项帮助。
- -c <配置文件> 所要使用的配置文件。/usr/local/etc/unbound/unbound.conf如果未给出，则使用默认配置文件 。
- -s <服务器[@端口]> 要联系的服务器的 IPv4 或 IPv6 地址。如果未给出，则从配置文件中读取地址。
command部分

- start 使用-c指定的配置文件启动服务，如果没指定，则使用默认配置文件

- stop 停止服务

- reload 重新加载配置文件，未指定则重新加载默认配置文件

- status 显示服务状态。退出代码 3 如果没有运行（端口连接被拒绝），错误为 1，如果运行为 0。

- reload_keep_cache 重新加载服务器，但如果（重新）配置允许，请尝试保留 RRset 和消息缓存。 这意味着缓存大小和线程数在重新加载之间不得更改。

- verbosity <number> 设置日志等级，与配置文件中的verbosity字段等同

- stats 打印统计数据。将内部计数器重置为零，这可以使用 statistics-cumulative: config 语句进行控制。每行打印一个统计信息。[name]: [value]

- stats_noreset 查看统计数据。像 stats 命令一样打印它们，但不会将内部计数器重置为零。

- local_zone <name> <type> 添加具有名称和类型的新本地区域。像本地区域配置语句。如果区域已经存在，则类型更改为给定的参数。

- local_zone_remove <name> 删除具有给定名称的本地区域。删除其中的所有本地数据。如果该区域不存在，则命令成功。

- local_data `<RR data...>` 添加新的本地数据，给定的资源记录。类似于local-data:关键字，除非不存在覆盖区域。在这种情况下，此远程控制命令会创建一个与此记录同名的透明区域。

- local_data_remove <name> 从本地名称中删除所有 RR 数据。如果该名称已经没有项目，则什么也不会发生。通常会导致名称的 NXDOMAIN（在静态区域中），但如果名称已变为空的非终结符（已删除名称下方的域名中仍有数据），则 NOERROR 无数据答案是该名称的结果。

- local_zones, local_zones_remove, local_datas, local_datas_remove 这些跟上面的相同，只是批量添加，从标准输入中读取 每行一条

- dump_cache 缓存的内容以文本格式打印到标准输出。您可以将其重定向到文件以将缓存存储在文件中。

- load_cache 缓存的内容是从标准输入加载的。使用与 dump_cache 相同的格式。用旧的或错误的数据加载缓存可能会导致将旧的或错误的数据返回给客户端。支持以这种方式将数据加载到缓存中以帮助调试。

- lookup <name> 将用于查找指定名称的名称服务器打印到标准输出。

- flush <name> 刷新缓存，从缓存中删除名称。删除类型 A、AAAA、NS、SOA、CNAME、DNAME、MX、PTR、SRV、NAPTR、SVCB 和 HTTPS。因为这样做很快。可以使用flush_type或flush_zone删除其他记录类型。

- flush_type <name> <type> 从缓存中删除名称、类型信息。

- flush_zone <name> 从缓存中删除名称处或名称下方的所有信息。删除 rrsets 和键条目，以便执行新的查找。这需要遍历和检查整个缓存，并且是一个缓慢的操作。条目在该命令的实现中被设置为过期（因此，启用服务过期后，它将提供该信息但安排预取新信息）。

- flush_stats 缓存计数清零

- flush_requestlist 刷新请求列表

- dump_requestlist 显示正在处理的内容。打印服务器当前正在处理的所有查询。打印用户等待的时间。对于内部请求，不会打印时间。然后打印出模块状态。这将打印来自第一个线程的查询，而不是正在从其他线程提供服务的查询。

- flush_infra [all | ip] 如果是all，则整个基础设施缓存被清空。如果是特定 IP 地址，则该地址的条目将从缓存中删除。它包含 EDNS、ping 和跛行数据。

- list_stubs 列出正在使用的根区域。这些被一一打印到输出中。这包括正在使用的根提示。

- list_forwards 列出正在使用转发的区域。这些按区域打印到输出。

- list_insecure 列出不安全的区域。

- list_local_zones 列出正在使用的本地区域。这些以区域类型每行打印一个。

- list_local_data 列出正在使用的本地数据 RR。打印资源记录。

- list_auth_zones 列出配置的授权区域。每行打印一个状态，指示区域是否过期和当前序列号。包括已配置的 RPZ 区域。

- forward_add [+i] zone addr.. 添加一个新的前向区域以运行 Unbound。With+i选项还为该区域添加了一个不安全的域（因此如果您为其他名称配置了 DNSSEC 根信任锚，它可以不安全地解析）。地址可以是 IP4、IP6 或名称服务器名称，如 unbound.conf 中的 forward-zone 配置。

- forward [off | addr ...] 设置转发模式。配置服务器是否应该询问其他上游名称服务器，应该去互联网根名称服务器本身，还是显示当前配置。您可以在 DHCP 更新后传递名称服务器。

- 如果没有参数，则打印用于将所有查询转发到的当前地址列表。在启动时，这是来自前向区"."配置。之后它显示状态。当没有使用转发时，它会打印出来。
- 如果传递了 off，则禁用转发并使用根名称服务器。这可用于避免错误或非 DNSSEC 支持从 DHCP 返回的名称服务器。但可能不适用于酒店或热点。
- 如果给出了一个或多个 IPv4 或 IPv6 地址，则这些地址将用于转发查询。地址必须用空格分隔。可以-显'@port'式设置端口号（默认端口为 53 (DNS)）。
- "."默认情况下，使用根配置文件中的转发器信息 。配置文件未更改，因此重新加载后这些更改将消失。配置文件中的其他转发区域不受此命令的影响


## 配置
配置的格式是以键值对的形式配置

注释以 # 开头，一直到行尾。 空行和行首的空格一样被忽略。

下面是一个最小的配置文件。源代码分发包含一个example.conf包含所有选项的扩展文件。

```bash
 cat  /etc/unbound/conf.d/my.conf 
server: 
        # 设置监听所有本地网络地址
        interface: 0.0.0.0
        # 限制只允许哪些地址可以使用该DNS服务
        port: 53
        do-ip4: yes
        do-udp: yes
        access-control: 127.0.0.0/8 allow       # 允许本地访问
        access-control: 192.168.0.0/16 allow    # 允许指定网段访问
        module-config: "iterator"       # 禁用DNSSEC的校验功能, 否则后面的转发会失败

        logfile: "/var/log/unbound.log" # 自定义日志存放的文件, 注意设置该文件的权限 chown unbound:unbound
        use-syslog: no  # 关闭系统级的日志输出, 使用journalctl就看不到了
        log-queries: yes        # 日志中记录详细的查询记录
        verbosity: 1    # 日志记录的详细程度, 0代表不记录详细信息, 范围1-5越高越啰嗦
    local-zone: "upcr.com." static
    local-data: "demo.upcr.com A 192.168.48.24"
    local-data-ptr: "192.168.48.24 demo.upcr.com"
    local-data: "demo.upcr.com A 192.168.48.25"
    local-data-ptr: "192.168.48.25 demo.upcr.com"
    local-data: "demo.upcr.com A 192.168.48.26"
    local-data-ptr: "192.168.48.26 demo.upcr.com"

# 对指定域名进行条件转发
    forward-zone:
        name: "."
        forward-addr: "119.29.29.29"
        forward-addr: "223.5.5.5"
    stub-zone:
        name: "upcr.com"
        stub-addr: 10.168.48.10
```
## 启动服务
重启服务

```bash
systemctl restart unbound && systemctl status unbound
```

## 测试

```bash
$ cat /etc/resolv.conf
# Generated by NetworkManager
nameserver 192.168.48.10
nameserver 8.8.8.8
search localhost

$ dig demo.upcr.com 

; <<>> DiG 9.16.23 <<>> demo.upcr.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 43494
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 3, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;demo.upcr.com.                 IN      A

;; ANSWER SECTION:
demo.upcr.com.          3600    IN      A       192.168.48.25
demo.upcr.com.          3600    IN      A       192.168.48.26
demo.upcr.com.          3600    IN      A       192.168.48.24

;; Query time: 0 msec
;; SERVER: 192.168.48.10#53(192.168.48.10)
;; WHEN: Mon Mar 25 22:25:29 CST 2024
;; MSG SIZE  rcvd: 90

```

