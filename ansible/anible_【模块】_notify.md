


----
## 1. 简介
Ansible会监控changed的状态，如果 changed=1，则表示关注的状态发生了改变，即本次任务的执行不具备幂等性，如果 changed=0，则表示本次任务要么没执行，要么执行了也没有影响，即本次任务具备幂等性。

Ansible提供了notify指令和handlers功能。如果在某个task中定义了notify指令，当Ansible在监控到该任务 changed=1时，会触发该notify指令所定义的handler，然后去执行handler。所谓handler，其实就是task，无论在写法上还是作用上它和task都没有区别，唯一的区别在于hander是被触发而被动执行的，不像普通task一样会按流程正常执行。

## 2. 示例

```bash
$ cat httpd.yml 
```

```bash
---
- name: play1
  hosts: all
  remote_user: root
  gather_facts: false
  
  tasks:
    - name: install httpd
      yum: name=httpd state=installed
    - name: copy httpd config
      copy: src=/etc/httpd/conf/httpd.conf  dest=/etc/httpd/conf/httpd.conf
    - name: start httpd
      service: name=httpd state=started enabled=true
```

```bash
$ vim /etc/httpd/conf/httpd.conf 
Listen 8888
 
#把本地的httpd配置文件复制到其他节点上。配置文件唯一的修改就是端口从80端口改为了8888端口
```

```bash
$ lsof -i:8080
COMMAND   PID   USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
httpd   14697   root    4u  IPv6 799818      0t0  TCP *:webcache (LISTEN)
httpd   14698 apache    4u  IPv6 799818      0t0  TCP *:webcache (LISTEN)
httpd   14699 apache    4u  IPv6 799818      0t0  TCP *:webcache (LISTEN)
httpd   14700 apache    4u  IPv6 799818      0t0  TCP *:webcache (LISTEN)
httpd   14701 apache    4u  IPv6 799818      0t0  TCP *:webcache (LISTEN)
httpd   14702 apache    4u  IPv6 799818      0t0  TCP *:webcache (LISTEN)


$ ansible-playbook httpd.yml 
 
PLAY [play1] ******************************************************************************************************************************
TASK [install httpd] **********************************************************************************************************************
ok: [192.168.179.100]
 
TASK [copy httpd config] ******************************************************************************************************************
changed: [192.168.179.100]
TASK [start httpd] ************************************************************************************************************************
ok: [192.168.179.100]
PLAY RECAP ********************************************************************************************************************************
192.168.179.100            : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
```
执行完成之后我们到节点上使用`lsof -i:8888`端口，会发现服务并没有重启

处理器是根据对应Task的返回状态来进行判断的。当Task(任务)状态为changed时，处理器就会执行你写好的handlers(处理器)操作

```bash
[root@localhost ~]# vim /etc/httpd/conf/httpd.conf
Listen 6666
 
[root@localhost ~]# cat httpd.yml 
---
- name: play1
  hosts: all
  remote_user: root
  gather_facts: false
  
  tasks:
    - name: install httpd
      yum: name=httpd state=installed
    - name: copy httpd config
      copy: src=/etc/httpd/conf/httpd.conf  dest=/etc/httpd/conf/httpd.conf
      notify:
       - restart httpd
    - name: start httpd
      service: name=httpd state=started enabled=true
    
  handlers:
    - name: restart httpd
      service: name=httpd state=restarted
#这里只要对httpd.conf配置文件作出了修改，修改后需要重启生效，在tasks中定义了restart httpd这个action，然后在handlers中引用上面tasks中定义的notify。
 
[root@localhost ~]# ansible-playbook httpd.yml 
 
PLAY [play1] ******************************************************************************************************************************
TASK [install httpd] **********************************************************************************************************************
ok: [192.168.179.100]
 
TASK [copy httpd config] ******************************************************************************************************************
changed: [192.168.179.100]
#这里的状态是changed，处理器的触发条件就是change 所以这里独发了处理器。
TASK [start httpd] ************************************************************************************************************************
ok: [192.168.179.100]
RUNNING HANDLER [restart httpd] ***********************************************************************************************************
changed: [192.168.179.100]
#触发器触发了
PLAY RECAP ********************************************************************************************************************************
192.168.179.100            : ok=4    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
[root@www ~]# lsof -i:6666
COMMAND   PID   USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
httpd   15395   root    4u  IPv6 803733      0t0  TCP *:ircu-2 (LISTEN)
httpd   15398 apache    4u  IPv6 803733      0t0  TCP *:ircu-2 (LISTEN)
httpd   15399 apache    4u  IPv6 803733      0t0  TCP *:ircu-2 (LISTEN)
httpd   15400 apache    4u  IPv6 803733      0t0  TCP *:ircu-2 (LISTEN)
httpd   15401 apache    4u  IPv6 803733      0t0  TCP *:ircu-2 (LISTEN)
httpd   15402 apache    4u  IPv6 803733      0t0  TCP *:ircu-2 (LISTEN)
```
handlers是处理器，他的使用方法和tasks(任务)一样。定义一个名称以及要执行的操作就行了

notify:则是在该Task中调用handlers里写好的操作，通过上图我们可以清楚的看到，在copy httpd config这个Task中我们就调用了处理器，而这个处理器要执行的命令则是重启httpd服务。

参考链接：
[https://blog.csdn.net/qq_34556414/article/details/108365191](https://blog.csdn.net/qq_34556414/article/details/108365191)

