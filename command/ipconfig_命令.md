ipconfig 命令

ipconfig 实用程序可用于显示当前的 TCP/IP 配置的设置值。这些信息一般用来检验人工配置的 TCP/IP 设置是否正确。

而且，如果计算机和所在的局域网使用了动态主机配置协议 DHCP，使用 ipconfig 命令可以了解到你的计算机是否成功地租用到了一个 IP 地址，如果已经租用到，则可以了解它目前得到的是什么地址，包括 IP 地址、子网掩码和缺省网关等网络配置信息。
 
下面给出最常用的选项：

(1) ipconfig：不带参数的Ipconfig只显示最基本的信息：IP地址、子网掩码和默认网关地址。默认情况下，仅显示绑定到 TCP/IP 的适配器的 IP 地址、子网掩码和默认网关。如果有多个网卡配置了IP地址，该命令都会一一显示出来。    
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200220102224976.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

”ipconfig /?”来寻求帮助信息
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020022010204478.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

(2) ipconfig /all：当使用 all 选项时，ipconfig 能为 DNS 和 WINS 服务器显示它已配置且所有使用的附加信息，并且能够显示内置于本地网卡中的物理地址（MAC）。如果 IP 地址是从 DHCP 服务器租用的，ipconfig 将显示 DHCP 服务器分配的 IP 地址和租用地址预计失效的日期。图为运行 ipconfig /all 命令的结果窗口。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200220102131559.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
(3) ipconfig /release 和 ipconfig /renew：这两个附加选项，只能在向 DHCP 服务器租用 IP 地址的计算机使用。如果输入 ipconfig /release，那么所有接口的租用 IP 地址便重新交付给 DHCP 服务器（归还 IP 地址）。如果用户输入 ipconfig /renew，那么本地计算机便设法与 DHCP 服务器取得联系，并租用一个 IP 地址。大多数情况下网卡将被重新赋予和以前所赋予的相同的 IP 地址。
Ipconfig /release命令是释放指定适配器的IPv4地址。
Ipconfig /release6命令是释放指定适配器的IPv6地址。
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020022010233151.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
Ipconfig /renew命令是更新指定适配器的IPv4地址。  
Ipconfig /renew6命令是更新指定适配器的IPv6地址。  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200220102426364.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

/flushdns 选项    
Ipconfig /flushdns命令清除DNS解析程序缓存。在排查DNS故障时，经常会用到该命令。    
例如：当访问一个网站时系统将从DNS缓存中读取该域名所对应的IP地址，当查找不到时就会到系统中查找hosts文件，如果还没有那么才会向DNS服务器请求一个DNS查询，DNS服务器将返回该域名所对应的IP，在你的系统收到解析地址以后将使用该IP地址进行访问，同时将解析缓存到本地的DNS缓存中。      
如果DNS地址无法解析，或者是DNS缓存中的地址错误，一般才会使用ipconfig/flushdns来清除所有的DNS缓存。    
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020022010252893.png)
/displaydns 选项    
Ipconfig /displaydns显示DNS解析器缓存的内容，包括从本地Hosts文件预装载的记录以及由域名解析服务器解析的所有资源记录。    
当执行完ipconfig /flushdns命令后，再执行ipconfig /displaydns命令时，如果本地hosts文件有手工添加的DNS记录，将会一一显示出来，如果本地hosts文件无DNS记录，将显示如下的信息.
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200220102554108.png)
