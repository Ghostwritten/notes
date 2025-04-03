


## firewald 与 iptables

在centos7中，有几种防火墙共存：firewald , iptables . 默认情况下，CentOS是使用firewalld来管理netfilter子系统，不过底层调用的命令仍然是iptables。

- firewalld 可以动态修改单挑规则，而不像iptables那样，在修改了规则后必须全部刷新才可以生效。

- firewalld在使用上比iptables人性化很多，即使不明白"五张表五条链"而且对TCP/IP协议也不理解也可以实现大部分功能。

- firewalld跟iptables比起来，不好的地方是每个服务都需要去设置才能放行，因为默认是拒绝。而iptables里默认每个服务是允许，需要拒绝才去限制。

- firewalld自身并不具备防火墙的功能，而是和iptables一样需要通过内核的netfilter来实现，也就是说firewalld和iptables一样，他们的作用是用于维护规则，而真正使用规则干活的是内核的netfilter,只不过firewalld和iptables的结构以及使用方法不一样罢了。

## firewalld
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/1c9dc21b13cf7c6cdc808c0ea9dc46f2.png)

注：Firewalld的默认区域是public

firewalld默认提供了九个zone配置文件：block.xml、dmz.xml、drop.xml、external.xml、 home.xml、internal.xml、public.xml、trusted.xml、work.xml，他们都保存在“/usr/lib /firewalld/zones/”目录下。

###  我应该选用哪个区域?
例如，公共的 WIFI 连接应该主要为不受信任的，家庭的有线网络应该是相当可信任的。根据与你使用的网络最符合的区域进行选择。
如何配置或者增加区域?
你可以使用任何一种 firewalld 配置工具来配置或者增加区域，以及修改配置。工具有例如 firewall-config 这样的图形界面工具， firewall-cmd 这样的命令行工具，以及D-BUS接口。或者你也可以在配置文件目录中创建或者拷贝区域文件。 @PREFIX@/lib/firewalld/zones 被用于默认和备用配置，/etc/firewalld/zones 被用于用户创建和自定义配置文件。
### 如何为网络连接设置或者修改区域？
区域设置以 ZONE= 选项 存储在网络连接的ifcfg文件中。如果这个选项缺失或者为空，firewalld 将使用配置的默认区域。
如果这个连接受到 NetworkManager 控制，你也可以使用 nm-connection-editor 来修改区域。
### 由 NetworkManager 控制的网络连接
防火墙不能够通过 NetworkManager 显示的名称来配置网络连接，只能配置网络接口。因此在网络连接之前 NetworkManager 将配置文件所述连接对应的网络接口告诉 firewalld 。如果在配置文件中没有配置区域，接口将配置到 firewalld 的默认区域。如果网络连接使用了不止一个接口，所有的接口都会应用到 fiwewalld。接口名称的改变也将由 NetworkManager 控制并应用到firewalld。
如果一个接口断开了， NetworkManager 也将告诉 firewalld 从区域中删除该接口。当 firewalld 由 systemd 或者 init 脚本启动或者重启后，firewalld 将通知 NetworkManager 把网络连接增加到区域。

```bash
$ rpm -qa|grep firewalld;rpm -qa|grep firewall-config 
$ yum -y install firewall-config
$ yum -y update firewalld 
$ rpm -qi firewalld firewall-config
$ systemctl start firewalld 
$ systemctl status firewalld  
$ systemctl disable firewalld  
$ systemctl enable firewalld
```

## firewall-cmd详解

在不改变状态的条件下重新加载防火墙：

```bash
$  firewall-cmd --reload
```

如果你使用 --complete-reload ，状态信息将会丢失。这个选项应当仅用于处理防火墙问题时，例如，状态信息和防火墙规则都正常，但是不能建立任何连接的情况。
获取支持的区域列表

```bash
 $ firewall-cmd --get-zones
```

这条命令输出用空格分隔的列表。
获取所有支持的服务

```bash
$  firewall-cmd --get-services
```

这条命令输出用空格分隔的列表。
获取所有支持的ICMP类型

```bash
$  firewall-cmd --get-icmptypes
```

这条命令输出用空格分隔的列表。
列出全部启用的区域的特性

```bash
$  firewall-cmd --list-all-zones
```

输出区域 <zone> 全部启用的特性。如果生略区域，将显示默认区域的信息。

```bash
$  firewall-cmd [--zone=<zone>] --list-all
```

获取默认区域的网络设置

```bash
$  firewall-cmd --get-default-zone
```

设置默认区域

```bash
$  firewall-cmd --set-default-zone=<zone>
```

流入默认区域中配置的接口的新访问请求将被置入新的默认区域。当前活动的连接将不受影响。
获取活动的区域

```bash
$  firewall-cmd --get-active-zones
```

这条命令将用以下格式输出每个区域所含接口：

```bash
 <zone1>: <interface1> <interface2> ..
 <zone2>: <interface3> ..
```

根据接口获取区域

```bash
 firewall-cmd --get-zone-of-interface=<interface>
```

这条命令将输出接口所属的区域名称。
将接口增加到区域

```bash
 firewall-cmd [--zone=<zone>] --add-interface=<interface>
```

如果接口不属于区域，接口将被增加到区域。如果区域被省略了，将使用默认区域。接口在重新加载后将重新应用。
修改接口所属区域

```bash
 firewall-cmd [--zone=<zone>] --change-interface=<interface>
```

这个选项与 --add-interface 选项相似，但是当接口已经存在于另一个区域的时候，该接口将被添加到新的区域。
从区域中删除一个接口

```bash
 firewall-cmd [--zone=<zone>] --remove-interface=<interface>
```

查询区域中是否包含某接口

```bash
 firewall-cmd [--zone=<zone>] --query-interface=<interface>
```

返回接口是否存在于该区域。没有输出。
列举区域中启用的服务

```bash
 firewall-cmd [ --zone=<zone> ] --list-services
```

启用应急模式阻断所有网络连接，以防出现紧急状况

```bash
 firewall-cmd --panic-on
```

禁用应急模式

```bash
 firewall-cmd --panic-off
```

查询应急模式

```bash
 firewall-cmd --query-panic
```

此命令返回应急模式的状态，没有输出。可以使用以下方式获得状态输出：

```bash
 firewall-cmd --query-panic && echo "On" || echo "Off"
```

处理运行时区域
运行时模式下对区域进行的修改不是永久有效的。重新加载或者重启后修改将失效。
启用区域中的一种服务

```bash
 firewall-cmd [--zone=<zone>] --add-service=<service> [--timeout=<seconds>]
```

此举启用区域中的一种服务。如果未指定区域，将使用默认区域。如果设定了超时时间，服务将只启用特定秒数。如果服务已经活跃，将不会有任何警告信息。
例: 使区域中的 ipp-client 服务生效60秒:

```bash
 firewall-cmd --zone=home --add-service=ipp-client --timeout=60
```

例: 启用默认区域中的http服务:

```bash
 firewall-cmd --add-service=http
```

禁用区域中的某种服务

```bash
 firewall-cmd [--zone=<zone>] --remove-service=<service>
```

此举禁用区域中的某种服务。如果未指定区域，将使用默认区域。
例: 禁止 home 区域中的 http 服务:

```bash
 firewall-cmd --zone=home --remove-service=http
```

区域种的服务将被禁用。如果服务没有启用，将不会有任何警告信息。
查询区域中是否启用了特定服务
 

```bash
firewall-cmd [--zone=<zone>] --query-service=<service>
```

如果服务启用，将返回1,否则返回0。没有输出信息。
启用区域端口和协议组合

```bash
 firewall-cmd [--zone=<zone>] --add-port=<port>[-<port>]/<protocol> [--timeout=<seconds>]
```

此举将启用端口和协议的组合。端口可以是一个单独的端口 <port> 或者是一个端口范围 <port>-<port> 。协议可以是 tcp 或 udp。
禁用端口和协议组合

```bash
 firewall-cmd [--zone=<zone>] --remove-port=<port>[-<port>]/<protocol>
```

查询区域中是否启用了端口和协议组合

```bash
 firewall-cmd [--zone=<zone>] --query-port=<port>[-<port>]/<protocol>
```

如果启用，此命令将有返回值。没有输出信息。
启用区域中的 IP 伪装功能

```bash
 firewall-cmd [--zone=<zone>] --add-masquerade
```

此举启用区域的伪装功能。私有网络的地址将被隐藏并映射到一个公有IP。这是地址转换的一种形式，常用于路由。由于内核的限制，伪装功能仅可用于IPv4。
禁用区域中的 IP 伪装

```bash
 firewall-cmd [--zone=<zone>] --remove-masquerade
```

查询区域的伪装状态

```bash
 firewall-cmd [--zone=<zone>] --query-masquerade
```

如果启用，此命令将有返回值。没有输出信息。

```bash
 firewall-cmd [--zone=<zone>] --add-icmp-block=<icmptype>
```

此举将启用选中的 Internet 控制报文协议 （ICMP） 报文进行阻塞。 ICMP 报文可以是请求信息或者创建的应答报文，以及错误应答。
禁止区域的 ICMP 阻塞功能

```bash
 firewall-cmd [--zone=<zone>] --remove-icmp-block=<icmptype>
```

查询区域的 ICMP 阻塞功能

```bash
 firewall-cmd [--zone=<zone>] --query-icmp-block=<icmptype>
```

如果启用，此命令将有返回值。没有输出信息。
例: 阻塞区域的响应应答报文:

```bash
 firewall-cmd --zone=public --add-icmp-block=echo-reply
```

在区域中启用端口转发或映射

```bash
 firewall-cmd [--zone=<zone>] --add-forward-port=port=<port>[-<port>]:proto=<protocol> { :toport=<port>[-<port>] | :toaddr=<address> | :toport=<port>[-<port>]:toaddr=<address> }
```

端口可以映射到另一台主机的同一端口，也可以是同一主机或另一主机的不同端口。端口号可以是一个单独的端口 <port> 或者是端口范围 <port>-<port> 。协议可以为 tcp 或udp 。目标端口可以是端口号 <port> 或者是端口范围 <port>-<port> 。目标地址可以是 IPv4 地址。受内核限制，端口转发功能仅可用于IPv4。
禁止区域的端口转发或者端口映射

```bash
 firewall-cmd [--zone=<zone>] --remove-forward-port=port=<port>[-<port>]:proto=<protocol> { :toport=<port>[-<port>] | :toaddr=<address> | :toport=<port>[-<port>]:toaddr=<address> }
```

查询区域的端口转发或者端口映射

```bash
 firewall-cmd [--zone=<zone>] --query-forward-port=port=<port>[-<port>]:proto=<protocol> { :toport=<port>[-<port>] | :toaddr=<address> | :toport=<port>[-<port>]:toaddr=<address> }
```

如果启用，此命令将有返回值。没有输出信息。
例: 将区域 home 的 ssh 转发到 127.0.0.2

```bash
 firewall-cmd --zone=home --add-forward-port=port=22:proto=tcp:toaddr=127.0.0.2
```

处理永久区域
永久选项不直接影响运行时的状态。这些选项仅在重载或者重启服务时可用。为了使用运行时和永久设置，需要分别设置两者。 选项 --permanent 需要是永久设置的第一个参数。
获取永久选项所支持的服务

```bash
 firewall-cmd --permanent --get-services
```

获取永久选项所支持的ICMP类型列表

```bash
 firewall-cmd --permanent --get-icmptypes
```

获取支持的永久区域

```bash
 firewall-cmd --direct --get-rules { ipv4 | ipv6 | eb } <table> <chain>
```

获取表 <table> 中所有增加到链 <chain> 的规则，并用换行分隔。
如果启用，此命令将有返回值。此命令没有输出信息。

```bash
 firewall-cmd --direct --query-rule { ipv4 | ipv6 | eb } <table> <chain> <args>
```

查询 带参数 <args> 的链 <chain> 是否存在表 <table> 中. 如果是，返回0,否则返回1.

```bash
 firewall-cmd --direct --remove-rule { ipv4 | ipv6 | eb } <table> <chain> <args>
```

从表 <table> 中删除带参数 <args> 的链 <chain>。

```bash
 firewall-cmd --direct --add-rule { ipv4 | ipv6 | eb } <table> <chain> <priority> <args>
```

为表 <table> 增加一条参数为 <args> 的链 <chain> ，优先级设定为 <priority>。

```bash
 firewall-cmd --direct --get-chains { ipv4 | ipv6 | eb } <table>
```

获取用空格分隔的表 <table> 中链的列表。
如果启用，此命令将有返回值。此命令没有输出信息。
 

```bash
firewall-cmd --direct --query-chain { ipv4 | ipv6 | eb } <table> <chain>
```

查询 <chain> 链是否存在与表 <table>. 如果是，返回0,否则返回1.

```bash
 firewall-cmd --direct --remove-chain { ipv4 | ipv6 | eb } <table> <chain>
```

从表 <table> 中删除链 <chain> 。

```bash
 firewall-cmd --direct --add-chain { ipv4 | ipv6 | eb } <table> <chain>
```

为表 <table> 增加一个新链 <chain> 。

```bash
 firewall-cmd --direct --passthrough { ipv4 | ipv6 | eb } <args>
```

将命令传递给防火墙。参数 <args> 可以是 iptables, ip6tables 以及 ebtables 命令行参数。
选项 --direct 需要是直接选项的第一个参数。
直接选项主要用于使服务和应用程序能够增加规则。 规则不会被保存，在重新加载或者重启之后必须再次提交。传递的参数 <args> 与 iptables, ip6tables 以及 ebtables 一致。
直接选项

```bash
 firewall-cmd --permanent --zone=home --add-forward-port=port=22:proto=tcp:toaddr=127.0.0.2
```

例: 将 home 区域的 ssh 服务转发到 127.0.0.2
如果服务启用，此命令将有返回值。此命令没有输出信息。

```bash
 firewall-cmd --permanent [--zone=<zone>] --query-forward-port=port=<port>[-<port>]:proto=<protocol> { :toport=<port>[-<port>] | :toaddr=<address> | :toport=<port>[-<port>]:toaddr=<address> }
```

查询区域的端口转发或者端口映射状态

```bash
 firewall-cmd --permanent [--zone=<zone>] --remove-forward-port=port=<port>[-<port>]:proto=<protocol> { :toport=<port>[-<port>] | :toaddr=<address> | :toport=<port>[-<port>]:toaddr=<address> }
```

永久禁止区域的端口转发或者端口映射
端口可以映射到另一台主机的同一端口，也可以是同一主机或另一主机的不同端口。端口号可以是一个单独的端口 <port> 或者是端口范围 <port>-<port> 。协议可以为 tcp 或udp 。目标端口可以是端口号 <port> 或者是端口范围 <port>-<port> 。目标地址可以是 IPv4 地址。受内核限制，端口转发功能仅可用于IPv4。

```bash
 firewall-cmd --permanent [--zone=<zone>] --add-forward-port=port=<port>[-<port>]:proto=<protocol> { :toport=<port>[-<port>] | :toaddr=<address> | :toport=<port>[-<port>]:toaddr=<address> }
```

在区域中永久启用端口转发或映射

```bash
 firewall-cmd --permanent --zone=public --add-icmp-block=echo-reply
```

例: 阻塞公共区域中的响应应答报文:
如果服务启用，此命令将有返回值。此命令没有输出信息。
 

```bash
firewall-cmd --permanent [--zone=<zone>] --query-icmp-block=<icmptype>
```

查询区域中的ICMP永久状态

```bash
 firewall-cmd --permanent [--zone=<zone>] --remove-icmp-block=<icmptype>
```

永久禁用区域中的ICMP阻塞
此举将启用选中的 Internet 控制报文协议 （ICMP） 报文进行阻塞。 ICMP 报文可以是请求信息或者创建的应答报文或错误应答报文。

```bash
 firewall-cmd --permanent [--zone=<zone>] --add-icmp-block=<icmptype>
```

永久启用区域中的ICMP阻塞
如果服务启用，此命令将有返回值。此命令没有输出信息。

```bash
 firewall-cmd --permanent [--zone=<zone>] --query-masquerade
```

查询区域中的伪装的永久状态

```bash
 firewall-cmd --permanent [--zone=<zone>] --remove-masquerade
```

永久禁用区域中的伪装
此举启用区域的伪装功能。私有网络的地址将被隐藏并映射到一个公有IP。这是地址转换的一种形式，常用于路由。由于内核的限制，伪装功能仅可用于IPv4。

```bash
 firewall-cmd --permanent [--zone=<zone>] --add-masquerade
```

永久启用区域中的伪装

```bash
 firewall-cmd --permanent --zone=home --add-port=443/tcp
```

例: 永久启用 home 区域中的 https (tcp 443) 端口
如果服务启用，此命令将有返回值。此命令没有输出信息。

```bash
 firewall-cmd --permanent [--zone=<zone>] --query-port=<port>[-<port>]/<protocol>
```

查询区域中的端口-协议组合是否永久启用

```bash
 firewall-cmd --permanent [--zone=<zone>] --remove-port=<port>[-<port>]/<protocol>
```

永久禁用区域中的一个端口-协议组合

```bash
 firewall-cmd --permanent [--zone=<zone>] --add-port=<port>[-<port>]/<protocol>
```

永久启用区域中的一个端口-协议组合

```bash
 firewall-cmd --permanent --zone=home --add-service=ipp-client
```

例: 永久启用 home 区域中的 ipp-client 服务
如果服务启用，此命令将有返回值。此命令没有输出信息。

```bash
 firewall-cmd --permanent [--zone=<zone>] --query-service=<service>
```

查询区域中的服务是否启用

```bash
 firewall-cmd --permanent [--zone=<zone>] --remove-service=<service>
```

禁用区域中的一种服务
此举将永久启用区域中的服务。如果未指定区域，将使用默认区域。

```bash
 firewall-cmd --permanent [--zone=<zone>] --add-service=<service>
```

启用区域中的服务

```bash
 firewall-cmd --permanent --get-zones
 firewall-cmd --permanent --get-zones
```

更多阅读：

- [Linux Commnad iptables 防火墙](https://blog.csdn.net/xixihahalelehehe/article/details/104895129)
- [https://mp.weixin.qq.com/s/PReHkFPotG5KjOjUoyciuw](https://mp.weixin.qq.com/s/PReHkFPotG5KjOjUoyciuw)








