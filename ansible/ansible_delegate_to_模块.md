


----
## 1. 场景介绍
在对一组服务器 server_group1 执行操作过程中，需要在另外一台机器 A 上执行一个操作，比如在 A 服务器上添加一条 hosts 记录，这些操作必须要在一个 playbook 联动完成。也就是是说 A 服务器这个操作与 server_group1 组上的服务器有依赖关系。Ansible 默认只会在定义好的一组服务器上执行相同的操作，这个特性对于执行批处理是非常有用的。但如果在这过程中需要同时对另外 1 台机器执行操作时，就需要用到 Ansible 的任务委派功能（delegate_to）。使用 delegate_to 关键字可以委派任务到指定的机器上运行。在 playbook 的操作如下：

```bash
- name: add host record 
  shell: 'echo "192.168.1.100 test.xyz.com" >> /etc/hosts'

 - name: add host record to center server 
  shell: 'echo "192.168.1.100 test.xyz.com " >> /etc/hosts'
  delegate_to: 192.168.1.1
```

任务委派功能还可以用于以下场景：

 - 在部署之前将一个主机从一个负载均衡集群中删除；
 - 当你要对一个主机做改变之前去掉相应 dns 的记录；
 - 当在一个存储设备上创建 iscsi 卷的时候；
 - 当使用外的主机来检测网络出口是否正常的时候。

## 2. 委托（delegate）
通过"`delegate_to`", 用户可以把某一个任务放在委托的机器上执行.

```bash
- hosts: webservers
  serial: 5

  tasks:

  - name: take out of load balancer pool
    command: /usr/bin/take_out_of_pool {{ inventory_hostname }}
    delegate_to: 127.0.0.1
```

上面的task会在跑ansible的机器上执行, "delegate_to: 127.0.0.1" 可以用local_action来代替

```bash
  tasks:

  - name: take out of load balancer pool
    local_action: command /usr/bin/take_out_of_pool {{ inventory_hostname }}
```

## 3. 委托者的facts
默认情况下, 委托任务的facts是`inventory_hostname`中主机的`facts`, 而不是被委托机器的facts. 在ansible 2.0 中, 设置`delegate_facts`为true可以让任务去收集被委托机器的facts.

```bash
- hosts: app_servers
  tasks:
    - name: gather facts from db servers
      setup:
      delegate_to: "{{item}}"
      delegate_facts: True
      with_items: "{{groups['dbservers'}}"
```

该例子会收集dbservers的facts并分配给这些机器, 而不会去收集app_servers的facts

##  4. run_once
通过`run_once: true`来指定该task只能在某一台机器上执行一次. 可以和`delegate_to` 结合使用

```bash
- command: /opt/application/upgrade_db.py
  run_once: true
  delegate_to: web01.example.org
```

指定在"web01.example.org"上执行这
如果没有delegate_to, 那么这个task会在第一台机器上执行
