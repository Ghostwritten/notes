
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/7b02c74c0918b8fe552ae023f76f2c62.png)



姓名                 日期               分数             

所有题目以RHEL7（CentOS7）系统为测试目标，答题期间请勿使用电脑或手机查询资料。
满分100分，面试及格线70分，内部考评及格线85分。

## 一、	笔答题（5题） 【25分】
1.1	文件系统管理 【评分标准，每个命令1分，必须正确才能得分】
写出5个与XFS文件系统相关的命令（格式化、查看、修复、评估、扩容）

```bash
mkfs.xfs		xfs.info		xfs_repair		xfs_estimate		xfs_growfs
```

1.2	磁盘分区管理 【评分标准，每个命令1分，parted必须先写出，不写分数减2分】
写出5个parted分区相关的命令（卷标、分区、设置类型、查看、删除）

```bash
parted		mklabel		mkpart 		set		print		rm		
```

1.3	Systemd服务管理 【评分标准，每个命令1分，必须正确才能得分】
写出5个Systemd方式管理httpd服务的命令（启动、停止、重启、状态、重载）

```bash
systemctl start httpd
systemctl stop httpd 
systemctl restart httpd 
systemctl status httpd 
systemctl reload httpd
```

1.4	防火墙管理 【评分标准，2分，必须正确才能得分，部分正确不得分】
写出RHEL7里的防火墙软件名称，并写出永久性地允许入站HTTP流量通过默认防火墙区域的命令

```bash
firewalld 
firewalld-cmd  - -permanent  - -add-service=http
```

1.5	网络服务管理 【评分标准，每个端口号正确得0.5分，每个端口类型正确得0.5分】
填写以下常见服务的网络端口和端口类型（TCP或UDP）
|服务名称	|端口号|	端口类型	|
|--|--|--|
|telnet	|21	|TCP	
|ftp	|23	|TCP	
|tftp	|69	|UDP	
|http	|80	|TCP	
|https|	443	|TCP	
|pop3|	110|	TCP	
|smtp|	25	|TCP	
|nfs v4	|2049|	TCP	



## 二、	判断题：（6题） 【6分，每题正确得1分】
2.1	RHEL7 没有32bit版本的安装介质 （  对 ）
2.2	RHEl7 里所有的系统服务状态都可以使用chkconfig命令查看 （  错  ）  
2.3	RHEL7 的 / 文件系统只能使用EXT4格式 （ 错   ）
2.4	RHEL7 使用nmcli命令管理网络后看不到真实文件 （ 错   ）
2.5	RHEL7 里看到的网卡名称ens192是安装方式错误导致的 （ 错   ）
2.6	RHEL7 内置的虚拟机技术是KVM （  对  ）

## 三、	单项选择题（10题） 【10分，每题1分】
3.1	RHEL7内核版本是 （  B ）
A:  2.6
B:  3.10
C:  4.1

3.2	下面哪张图片显示的是RHEL7内存结果 （ B   ）
A:  
B:  

3.3	RHEL7 内置的容器技术是 （  B ）
A:  CoreOS 
B:  Docker
C:  PowerVM

3.4	RHEL7 默认的文件系统类型是 （  A ）
A:  XFS
B:  EXT3 
C:  EXT4 

3.5	RHEL7 的默认系统和服务管理器是 （  C ）
A:  SysV init
B:  NetworkManager
C:  Systemd

3.6	RHEL7 的主机名配置文件是 （ C  ）
A:  /etc/hosts
B:  /etc/sysconfig/network
C:  /etc/hostname

3.7	RHEL7 系统推荐的时钟服务是 （ B  ）
A:  NTP
B:  Chrony
C:  Local Time

3.8	RHEL7 的哪个目录里的systemd unit配置文件优先级最高 （  C ）
A:  /usr/lib/systemd/system/
B:  /run/systemd/system/
C:  /etc/systemd/system/

3.9	XFS文件系统最大认证容量是（ C   ）
A:  64TB
B:  256TB
C:  500TB

3.10	RHEL7自带的数据库软件是（  C  ）
A:  MySQL
B:  PostgreSQL
C:  MariaDB


## 四、	多项选择题（2题） 【4分，每题2分，必须全部正确才得分】
4.1	System unit 文件包含哪些主要Section （ A E D ）
按照从上到下的正确顺序将编号填写到括号的内
A:  [Unit]
B:  [Target] 
C:  [Server]
D: 	[Install]
E:  [Unit Type]

4.2	cron计划任务包含哪些时间周期（   E D C B G   ）
按照从前到后的正确顺序将编号填写到后面的内
A: 年   B: 月  C: 日  D: 时	E: 分	F: 秒	G：周


## 五、	实验模拟题（1题）【15分，每题3分，要求动作命令必须全部正确才得2分，检查命令有加1分，没有检查命令每题最多只能得2分】
系统新增磁盘/dev/sdc，大小100G，请写出完成以下工作的准确命令
要求：必须按照每个小题目逐步完成，不可以跳过前面题目直接完成最后题目。

5.1	在新增磁盘上创建2个物理卷pv，大小分别为40G，50G
使用fdisk 或 parted 对磁盘 /dev/sdc 进行分区，分出 /dev/sdc1 大小40G ， /dev/sdc2 大小50G

```bash
pvcreate /dev/sdc1
pvcreate /dev/sdc2
pvs （检查命令）
```


5.2	创建新卷组vgdata, 将创建的第一个pv增加到新卷组vgdata中

```bash
vgcreate vgdata /dev/sdc1
vgs （检查命令）
```

5.3	在卷组vgdata中创建逻辑卷lvdata，容量20G，文件系统xfs，挂载点/data

```bash
lvcreate -L +20G -n lvdata vgdata
lvs （检查命令）
mkfs.xfs /dev/vgdata/lvdata
mkdir /data 
mount /dev/vgdata/lvdata /data
df -Th （检查命令）
```

5.4	将逻辑卷lvdata和文件系统都扩容到80G

```bash
vgextend vgdata /dev/sdc2
vgs （检查命令）
lvextend -L 80G vgdata/lvdata
lvs （检查命令）
xfs_growfs /dev/vgdata/lvdata
df -Th （检查命令）
```

5.5	系统启动需要自动挂载逻辑卷lvdata，每次启动检查，检查顺序2，并且确保下次系统重启能够正确引导。

```bash
vi /etc/fstab
/dev/vgdata/lvdata /data xfs defaults 1 2
mount -a （检查命令）
```

 

## 六、	故障排错题（1题） 【10分，第一条答案2分，其余每条答案1分，意思描述正确即可得分】
用户反映Web服务器（httpd-2.4）突然无法访问，如果你是Linux系统管理员，请写出你排查故障的思路和操作命令。
1. 先询问故障现象，了解是否有过变更，了解故障时间点，了解业务高峰时段等，通过以上信息初步分析可能引起故障的原因是变更造成的还是性能原因，为下一步排查准备基本信息
2. 查看操作系统基本信息，包括系统版本，内核版本，存储使用率，内存使用率，top等，检查命令uname -r df -Th free -m top 
3. 查看Web服务器查看软件版本 rpm -qi httpd
4. 检查httpd进程 ps -ef|grep httpd
5. 检查httpd端口 netstat -antp |grep http 【注意陷阱：题目并没有说用的是什么端口，也没有说是否采用SSL加密，所以不能直接查询80或443端口】，查看防火墙和SELinux是否被开启，ping测试网关等
6. 查看httpd日志error_log
7. 配置文件语法检查httpd -t 
8. 根据以上排查结果，定位原因，修正错误或优化参数，重新启动服务，并请业务验证，后续加强观察
9. 做好故障排错记录，整理故障分析和排错报告
 

## 七、	设计题（1题）【15分，逻辑图5分，服务器端2分，，网络交换机1分，客户端2分；方案主要工作内容每条答案2分，意思描述正确即可得分】
客户新采购了50台物理服务器，希望在较短时间内完成所有服务器的Linux操作系统的安装，并且希望安装的系统都是一样的标准，请你设计一套有效的方案，画出实现逻辑图，并写出实现该方案的主要工作内容。

逻辑图

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/7e0f9c2bdf1a8f5657584122878c9be0.png)

1. 首先向客用户了解希望在完成的时间周期（“短时间”只是一个基本表述，一天、一周、半个月、一个月都可以算是段时间，关键是放在多长的时间周期内来看）要通过了解过程，判断这个所谓的“短时间”是否合理，是否需要其他部门配合，是否有其它限制条件，如物理服务器上架是否也包含在内，网络布线，存储连接，交换机是否就绪等，这些都需要通过向客户了解才能知道，而这些信息都是完成这个“短时间”目标不可忽视的前提条件）然后再根据情况去实际动手完成具体工作。
2. 了解客户的希望达到的“一样的标准”是什么，根据用户给出的标准，安装出标准操作系统模板，并且请用户确认是符合“标准”的。要完成50台目标服务器相同标准的安装工作，首选是Linux自带的Kickstart方案，根据“标准模板”服务器，生成安装Kickstart配置，并且在有限的环境内测试，确保可用。
3. 如果要使用Kickstart方案，首先是通过网络方式安装，效率最高。但是这里又涉及到用户安装环境网络是否已经就绪（至少是部分网络就绪，可以实现网络安装）；如果网络就绪，用户是否允许在生产环境使用PXE+DHCP自动引导，如果允许，需要搭建PXE+DHCP服务器；如果不允许，需要手工引导（如使用安装光盘或USB引导盘）+指定临时IP方式。如果网络不就绪或者要求在网络就绪前就完成系统标准化安装工作，就需要定制包含Kickstart配置的安装光盘（或USB安装盘），或者标准安装盘+定制Kickstart文件媒体方式安装。
4. 使用之前生成的Kickstart文件，安装少量测试服务器，并请用户验证部署后的服务器符合“标准”，如有不符合的地方，需要及时修改并请用户确认修改后完全正确。根据确认后的“标准Kickstart”文件批量安装服务器，在用户要求的“短时间”内完成工作，并请用户验收确认。
5. 做好方案设计文档、安装配置操作手册、工作日报、服务记录、项目总结等文字工作。


## 八、	自述题  【10分，第一条答案1分，意思描述正确即可得分】
请简述你在学习Linux知识时候的主要的途径和方法（不少于100字）
1. 参加专业培训课程，包括网上培训，现场培训等
2. 阅读专业书籍，包括纸质书，电子书等
3. 关注专业技术微信公众号，如Linux中国，运维帮等
4. 遇到问题到谷歌搜索（坚决不用百度！）
5. 浏览各大开源社区网站，如Linux kernel，Apache，Puppet，Zabbix，ELK等
6. 关注安全网站，如CVE、NIST等
7. 源码分享网站GitHub
8. 向专业大牛请教，向老工程师请教，与同事交流问题，参加社区活动
9. 讨论问题的方式方法包括邮件，电话，微信，微信群，社区反馈等
10. 测试，练习，重复测试和练习，并在重复过程中学会如何提高效率并总结经验



