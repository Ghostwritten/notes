

----
##  1. 环境配置
Ansible配置以ini格式存储配置数据，在Ansible中⼏乎所有配置都可以通过Ansible Playbook或环境变量来重新赋值。在运⾏Ansible命令时，命令将会按照以下顺序查找配置⽂件。

 - `ANSIBLE_CONFIG` ：⾸先，Ansible命令会检查环境变量，及这个环境变量指向的配置⽂件。
 - `./ansible.cfg` ：其次，将会检查当前⽬录下的ansible.cf g配置⽂件。
 - `~/.ansible.cfg` ：再次，将会检查当前⽤户home⽬录下的.ansible.cf g配置⽂件。
 - `/etc/ansible/ansible.cfg` ：最后，将会检查在⽤软件包管理⼯具
 - 安装Ansible时⾃动产⽣的配置⽂件。

⼤多数的Ansible参数可以通过设置带有 ANSIBLE_ 开头的环境变量进⾏配置，参数名称必须都是⼤写字母，如下配
置：

```bash
export ANSIBLE_SUDO_USER=root
```
设置了环境变量之后， ANSIBLE_SUDO_USER 就可以在后续操作中直接引⽤。

## 2. ansible.cfg配置

### 2.1 [defaults]  

```bash
[defaults]  
# inventory = /etc/ansible/hosts        # 定义Inventory  
# library = /usr/share/my_modules/      # 自定义lib库存放目录  
# remote_tmp = $HOME/.ansible/tmp       # 临时文件远程主机存放目录  
# local_tmp = $HOME/.ansible/tmp        # 临时文件本地存放目录  
# forks = 5                  # 默认开启的并发数  
# poll_interval = 15         # 默认轮询时间间隔  
# sudo_user  = root          # 默认sudo用户  
# ask_sudo_pass = True       # 是否需要sudo密码  
# ask_pass  = True           # 是否需要密码  
# roles_path = /etc/ansible/roles   # 默认下载的Roles存放的目录  
# host_key_checking = False         # 首次连接是否需要检查key认证，建议设为False  
# timeout = 10               # 默认超时时间  
# timeout = 10               # 如没有指定用户，默认使用的远程连接用户  
# log_path = /var/log/ansible.log   # 执行日志存放目录  
# module_name = command             # 默认执行的模块  
# action_plugins = /usr/share/ansible/plugins/action     # action插件的存放目录  
# callback_plugins = /usr/share/ansible/plugins/callback # callback插件的存放目录  
# connection_plugins = /usr/share/ansible/plugins/connection    # connection插件的存放目录  
# lookup_plugins = /usr/share/ansible/plugins/lookup     # lookup插件的存放目录  
# vars_plugins = /usr/share/ansible/plugins/vars         # vars插件的存放目录  
# filter_plugins = /usr/share/ansible/plugins/filter     # filter插件的存放目录  
# test_plugins = /usr/share/ansible/plugins/test         # test插件的存放目录  
# strategy_plugins = /usr/share/ansible/plugins/strategy # strategy插件的存放目录  
# fact_caching = memory                      # getfact缓存的主机信息存放方式  
# retry_files_enabled = False 
# retry_files_save_path = ~/.ansible-retry   # 错误重启文件存放目录  
```

述是日常可能用到的配置，这些多数保持默认即可。

### 2.2 [privilege_escalation]
出于安全角度考虑，部分公司不希望直接以root的高级管理员权限直接部署应用，往往会开放普通用户权限并给予sudo的权限，该部分配置主要针对sudo用户提权的配置。

```bash
[privilege_escalation]  
# become=True          # 是否sudo  
# become_method=sudo   # sudo方式  
# become_user=root     # sudo后变为root用户  
# become_ask_pass=False # sudo后是否验证密码 
```
### 2.3 [paramiko_connection]
定义paramiko_connection配置，该部分功能不常用，了解即可。

```bash
[paramiko_connection]     # 该配置不常用到  
# record_host_keys=False  # 不记录新主机的key以提升效率  
# pty=False               # 禁用sudo功能 
```

### 2.4 [ssh_connection]
Ansible默认使用SSH协议连接对端主机，该部署是主要是SSH连接的一些配置，但配置项较少，多数默认即可。

```bash
[ssh_connection]  
# pipelining = False    # 管道加速功能，需配合requiretty使用方可生效 
```

### 2.5 [accelerate]
Ansible连接加速相关配置。因为有部分使用者不满意Ansible的执行速度，所以Ansible在连接和执行速度方面也在不断地进行优化，该配置项在提升Ansibile连接速度时会涉及，多数保持默认即可。

```bash
[accelerate]  
# accelerate_port = 5099            # 加速连接端口  
# accelerate_timeout = 30           # 命令执行超时时间，单位秒  
# accelerate_connect_timeout = 5.0  # 连接超时时间，单位秒  
# accelerate_daemon_timeout = 30        # 上一个活动连接的时间，单位分钟  
# accelerate_multi_key = yes 
```

### 2.6 [selinux]
关于selinux的相关配置几乎不会涉及，保持默认配置即可。

```bash
[selinux]  
# libvirt_lxc_noseclabel = yes 
# libvirt_lxc_noseclabel = yes 
```

### 2.7 [colors]
Ansible对于输出结果的颜色也进行了详尽的定义且可配置，该选项对日常功能应用影响不大，几乎不用修改，保持默认即可。

```bash
[colors]  
# highlight = white 
# verbose = blue 
# warn = bright purple  
# error = red 
# debug = dark gray  
# deprecate = purple 
# skip = cyan 
# unreachable = red 
# ok = green 
# changed = yellow 
# diff_add = green 
# diff_remove = red 
# diff_lines = cyan 
```

上面尽可能全地介绍了运维工作中可能需要修改的配置选项，除了在关闭首次连接提示（host_key_checking = False）或提速调整（[accelerate]区域块配置调整）时可能会稍做调整，其中绝大多数选项默认即可，Ansible安装好后无需任何改动即可使用。

## 3. 互信配置
### 3.1 生成密钥对

```bash
ssh-keygen -t rsa
```
### 3.2 建立互信

```bash
ssh-copy-id -i ~/.ssh/id_rsa.pub username@192.168.1.2
```
or

```bash
将 .pub 文件复制到B机器的 .ssh 目录， 并 cat id_dsa.pub >> ~/.ssh/authorized_keys
```

根据提示完成操作即完成了互信。


### 3.3 批量建立互信
#### 3.3.1 第一种方法： playbook
建立如下 playbook：

```bash
---
  - hosts: test
    # 互信用户
    user: hoxis
    tasks:
     - name: ssh-copy
       authorized_key: user=hoxis key="{{ lookup('file', '/home/hoxis/.ssh/id_rsa.pub') }}"
```
默认，已在 Ansible 资产文件中配置待互信机器信息。

```bash
$ ansible-playbook pushssh.yaml

PLAY [test] *****************

TASK [Gathering Facts] ******
ok: [client]

TASK [ssh-copy] *************
changed: [client]

PLAY RECAP ******************
client                     : ok=2    changed=1    unreachable=0    failed=0
```
#### 3.3.2 第二种方法：shell&expect脚本

```bash
# 安装expect
[root@server2 ~]# yum -y install expect
# expect脚本
[root@server2 ~]# cat auto_sshcopyid.exp
#!/usr/bin/expect
set timeout 10
set user_hostname [lindex $argv 0]
set password [lindex $argv 1]
spawn ssh-copy-id $user_hostname
expect {
 "(yes/no)?"
 {
 send "yes\n"
 expect "*password: " { send "$password\n" }
 }
 "*password: " { send "$password\n" }
}
expect eof
```

批量调⽤expect的shell脚本
```bash
[root@server2 ~]# cat sshkey.sh
#!/bin/bash
ip=`echo -n "$(seq -s "," 59 65),150" | xargs -d "," -i echo 192.168.100.{}`
password="123456"
#user_host=`awk '{print $3}' /root/.ssh/id_rsa.pub`
for i in $ip;do
 /root/auto_sshcopyid.exp root@$i $password &>>/tmp/a.log
 ssh root@$i "echo $i ok"
done
```
执⾏shell脚本配置互信
```bash
[root@server2 ~]# chmod +x /root/{sshkey.sh,auto_sshcopyid.exp}
[root@server2 ~]# ./sshkey.sh
```
##  4. inventory配置
inventory⽤于定义ansible要管理的主机列表，可以定义单个主机和主机组。上⾯的`/etc/ansible/hosts`就是默认的
inventory。在使用时通过–i或--inventory-file指定读取，与Ansible命令结合使用时组合如下：

```bash
ansible –i /etc/ansible/hosts webs –m ping
```

Inventory可以同时存在多个，而且支持动态生成，如AWS EC2、Cobbler等均支持，本节我们来学习Inventory的使用规则。

### 4.1 定义主机和组
Inventory配置文件遵循INI文件风格，中括号中的字符为组名。其支持将同一个主机同时归并到多个不同的组中，分组的功能为IT人员维护主机列表提供了非常大的便利。此外，若目标主机使用了非默认的SSH端口，还可以在主机名称之后使用冒号加端口号来标明，以行为单位分隔配置，详细信息可参考以下代码中的注释。

```bash
 # “# ”开头的行表示该行为注释行，即当时行的配置不生效
# Inventory可以直接为IP地址
192.168.37.149
# Inventory同样支持Hostname的方式，后跟冒号加数字表示端口号，默认22号端口
ntp.magedu.com:2222
nfs.magedu.com
# 中括号内的内容表示一个分组的开始，紧随其后的主机均属于该组成员，空行后的主机亦属于该组，即web2.magedu.com这台主机也属于[websevers]组
[websevers]
web1.magedu.com
web[10:20].magedu.com # [10:20]表示10～20之间的所有数字（包括10和20），即表示web10.magedu.com、web11.magedu.com……web20.magedu.com的所有主机

web2.magedu.com[dbservers]
db-a.magedu.com
db-[b:f].magedu.com # [b:f]表示b到f之间的所有数字（包括b和f），即表示db-b.magedu.com、db-e.magedu.com……db-f.magedu.com的所有主机
```
### 4.2 定义主机变量
在日常工作中，通常会遇到非标准化的需求配置，如考虑到安全性问题，业务人员通常将企业内部的Web服务80端口修改为其他端口号，而该功能可以直接通过修改Inventory配置来实现，在定义主机时为其添加主机变量，以便在Playbook中使用针对某一主机的个性化要求。

```bash
[webservers]
web1.magedu.com http_port=808 maxRequestsPerChild=801 # 自定义http_port的端口号为808，配置maxRequestsPerChild为801
```
### 4.3 定义组变量
Ansible支持定义组变量，主要针对大量机器的变量定义需求，赋予指定组内所有主机在Playbook中可用的变量，等同于逐一给该组下的所有主机赋予同一变量。定义组变量的参考案例如下：

```bash
[groupservers]
web1.magedu.com
web2.magedu.com
[groupservers:vars]
ntp_server=ntp.magedu.com  # 定义groupservers组中所有主机ntp_server值为ntp.magedu.com
nfs_server=nfs.magedu.com # 定义groupservers组中所有主机nfs_server值为nfs.magedu.com
```
### 4.4 定义组嵌套及组变量
Inventory中，组还可以包含其他的组（嵌套），并且也可以向组中的主机指定变量。不过，这些变量只能在Ansible-playbook中使用，而Ansible不支持。组与组之间可以相互调用，并且可以向组中的主机指定变量。
参考示例如下：

```bash
[apache]
httpd1.magedu.com
httpd2.magedu.com

[nginx]
ngx1.magedu.com
ngx2.magedu.com

[webservers:children]
apache
nginx

[webservers:vars]
ntp_server=ntp.magedu.com
```
### 4.5 多重变量定义
变量除了可以在Inventory中一并定义，也可以独立于Inventory文件之外单独存储到YAML格式的配置文件中，这些文件通常以.yml、.yaml、.json为后缀或者无后缀。变量通常从如下4个位置检索：

```bash
Inventory配置文件（默认/etc/ansible/hosts）
Playbook中vars定义的区域
Roles中vars目录下的文件
Roles同级目录group_vars和hosts_vars目录下的文件
```

假如foosball主机同属于raleigh和webservers组，那么其变量在如下文件中设置均有效：

```bash
/etc/ansible/group_vars/raleigh # can optionally end in '.yml', '.yaml', or '.json'
/etc/ansible/group_vars/webservers
/etc/ansible/host_vars/foosball
```

对于变量的读取，Ansible遵循如上优先级顺序，因此大家设置变量时尽量沿用同一种方式，以方便维护人员管理。

### 4.6 其他Inventory参数列表
除了支持如上的功能外，Ansible基于SSH连接Inventory中指定的远程主机时，还内置了很多其他参数，用于指定其交互方式，如下列举了部分重要参数：

```bash
ansible_ssh_host：指定连接主机ansible_ssh_port，指定SSH连接端口，默认22
ansible_ssh_user：指定SSH连接用户ansible_ssh_pass，指定SSH连接密码ansible_sudo_pass：指定SSH连接时sudo密码
ansible_ssh_private_key_file：指定特有私钥文件
ansible_ssh_port ： ssh的端⼝。默认为22。
ansible_connection ： 使⽤何种模式连接到远程主机。默认值为smart(智能)，表⽰当本地ssh⽀持持久连接
(controlpersist)时采⽤ssh连接，否则采⽤python的paramiko ssh连接。
ansible_shell_type ： 指定远程主机执⾏命令时的shell解析器，默认为sh(不是bash，它们是有区别的，也不是
全路径)。
ansible_python_interpreter ： 远程主机上的python解释器路径。默认为/usr/bin/python。
ansible_*_interpreter ：使⽤什么解释器。例如，sh、bash、awk、sed、expect、ruby等等。
其中有⼏个参数可以在配置⽂件ansible.cf g中指定，但指定的指令不太⼀样，以下是对应的配置项：
remote_port： 对应于ansible_ssh_port。
remote_user： 对应于ansible_ssh_user。
private_key_f ile： 对应于ansible_ssh_private_key_f ile。
excutable： 对应于ansible_shell_type。但有⼀点不⼀样，excutable必须指定全路径，⽽后者只需指定
basename。
```
如果定义了"ansible_ssh_host"，那么其前⾯的主机名就称为别名。例如，以下inventory⽂件中nginx就是⼀个别
名，真正连接的对象是192.168.100.65。

```bash
nginx ansible_ssh_host=192.168.100.65 ansible_ssh_port=22
```
`inventory_hostname` 是ansible中可以使⽤的⼀个变量，该变量代表的是每个主机在inventory中的主机名称

参考链接：
[马龙帅](https://www.cnblogs.com/f-ck-need-u/)
[https://developer.aliyun.com/article/90218](https://developer.aliyun.com/article/90218)
[https://hoxis.github.io/ansible-ssh-copy.html](https://hoxis.github.io/ansible-ssh-copy.html)
