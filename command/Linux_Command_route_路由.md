
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/4908b5b162d741898b8cb171ec28cafe.png)



### 深入了解 Linux route 命令

在 Linux 系统中，`route` 是一个用于管理和查看网络路由表的命令。网络路由表是操作系统用来决定如何通过网络发送数据包的关键组件。

#### 一、route 命令的作用

- **查看路由表**：显示当前系统的网络路由配置。
- **添加路由**：向路由表中添加静态路由规则。
- **删除路由**：移除指定的路由规则。
- **修改路由**：更改现有的路由配置。

尽管 `route` 命令功能强大，但现代 Linux 发行版逐渐用 `ip route` 替代了它。然而，`route` 命令仍然是理解和管理网络路由的一个重要工具。

#### 二、route 命令的基本语法

```bash
route [选项] [操作] [目标] [网关] [接口]
```

常用选项包括：
- `-n`：以数字形式显示 IP 地址和网关，而不是解析为主机名。
- `add`：添加路由。
- `del`：删除路由。

#### 三、常见用法

##### 1. 查看路由表
运行以下命令可以显示当前路由表：

```bash
route -n
```

输出示例：
```
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         192.168.1.1     0.0.0.0         UG    100    0        0 eth0
192.168.1.0     0.0.0.0         255.255.255.0   U     0      0        0 eth0
```
- `Destination`：目标网络或主机。
- `Gateway`：数据包需要通过的网关。
- `Iface`：使用的网络接口。

##### 2. 添加静态路由
将网络 10.0.0.0/24 的流量通过网关 192.168.1.254 发送：

```bash
route add -net 10.0.0.0 netmask 255.255.255.0 gw 192.168.1.254
```

指定接口添加路由：

```bash
route add -net 10.0.0.0 netmask 255.255.255.0 dev eth1
```

##### 3. 删除路由
移除默认路由：

```bash
route del default gw 192.168.1.1
```

##### 4. 设置默认路由
设置 192.168.1.1 为默认网关：

```bash
route add default gw 192.168.1.1
```

#### 四、route 命令的使用场景

- **临时路由配置**：快速添加或删除路由规则，适用于调试或临时网络需求。
- **网络分段**：为多网段环境手动配置路由规则。
- **特殊流量路由**：为特定目标设置独立的流量路径。

#### 五、替代工具

随着 `ip` 命令的普及，`route` 命令逐渐被弃用。以下是等价的 `ip` 命令示例：

- 查看路由表：

```bash
ip route show
```

- 添加路由：

```bash
ip route add 10.0.0.0/24 via 192.168.1.254 dev eth1
```

- 删除路由：

```bash
ip route del 10.0.0.0/24
```

#### 六、注意事项

1. **持久化配置**：
通过 `route` 命令添加的路由是临时的，系统重启后会丢失。为了持久化路由规则，需要将其写入网络配置文件，例如 `/etc/network/interfaces` 或 `/etc/sysconfig/network-scripts/` 下的相关配置文件。

2. **权限要求**：
大多数 `route` 命令需要超级用户权限运行，可以使用 `sudo` 提升权限。

3. **使用替代命令**：
建议逐步切换到 `ip` 命令以适应现代 Linux 系统。

#### 七、总结

`route` 命令是 Linux 网络管理中的重要工具之一，它为手动配置和管理路由提供了直观的方式。虽然它正在被更强大的 `ip` 命令取代，但熟悉 `route` 命令仍然有助于理解网络路由的基础概念，尤其是在老旧系统或特定环境中仍有应用价值。

通过灵活使用 `route` 命令，管理员可以轻松调试和优化网络流量路径，为网络通信提供可靠保障。



参考：

 - [route命令（详细）](https://blog.csdn.net/u013485792/article/details/51700808)
 - [route command in Linux with Examples](https://www.geeksforgeeks.org/route-command-in-linux-with-examples/)
 - [route(8) — Linux manual page](https://man7.org/linux/man-pages/man8/route.8.html)
 - [7 Linux Route Command Examples (How to Add Route in Linux)](https://www.thegeekstuff.com/2012/04/route-examples/)
 - [An introduction to Linux network routing](https://opensource.com/business/16/8/introduction-linux-network-routing)

