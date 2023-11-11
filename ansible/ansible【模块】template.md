

----
## 1. 简介
template模块会在ansible控制机中对模板文件进行渲染，最终生成各个主机对应的配置文件，然后拷贝到远程主机的指定位置中。

## 2. 参数

 - `owner`: 指定最终生成的文件拷贝到远程主机后的属主。
 - `group`: 指定最终生成的文件拷贝到远程主机后的属组。
 - `mode`:指定最终生成的文件拷贝到远程主机后的权限，如果你想将权限设置为”rw-r–r–“，则可以使用mode=0644表示，如果你想要在user对应的权限位上添加执行权限，则可以使用mode=u+x表示。

 除了上述参数，还有如下参数也很常用

 - `force`: 远程主机的目标路径中已经存在同名文件，并且与最终生成的文件内容不同时，是否强制覆盖，可选值有yes和no，默认值为yes，表示覆盖，如果设置为no，则不会执行覆盖拷贝操作，远程主机中的文件保持不变。

 - `backup`:当远程主机的目标路径中已经存在同名文件，并且与最终生成的文件内容不同时，是否对远程主机的文件进行备份，可选值有yes和no，当设置为yes时，会先备份远程主机中的文件，然后再将最终生成的文件拷贝到远程主机。

## 3. 示例
### 3.1 使用template模块在jinja2中引用变量，先来目录结构树
[root@master ansible]# tree

```bash
.
├── ansible.cfg
├── hosts
├── roles
│   └── temp
│       ├── tasks
│       │   └── main.yaml
│       ├── templates
│       │   ├── test_if.j2
│       │   └── test.j2
│       └── vars
│           └── main.yaml
└── work_dir
    ├── copy_configfile.retry
    └── copy_configfile.yaml
```
打开定义好的变量：

```bash
[root@master ansible]# cat roles/temp/vars/main.yaml 
master_ip: 192.168.101.14
master_hostname: master
node1_ip: 192.168.101.15
node1_hostname: node1
```
打开hosts文件查看节点信息：

```bash
[root@master ansible]# egrep -v "^#|^$" hosts 
[nodes]
192.168.101.14 
192.168.101.15
```
现在通过定义好的变量在templates目录下创建j2文件：

```bash
[root@master ansible]# cat roles/temp/templates/test.j2 
ExecStart=/usr/local/bin/etcd --name {{ master_hostname }} --initial-advertise-peer-urls http://{{ master_ip }}:2380
```
查看tasks主任务定义：

```bash
[root@master ansible]# cat roles/temp/tasks/main.yaml 
- name: copy configfile to nodes
  template:
    src: test.j2
    dest: /tmp/test.conf
```
查看工作目录下面的执行yaml：

```bash
[root@master ansible]# cat work_dir/copy_configfile.yaml 
- hosts: nodes
  remote_user: root
  roles: 
    - temp
```
在tasks目录下面的main.yaml定义使用了template模块，调用templates目录下面的test.j2文件

执行：

```bash
[root@master ansible]# ansible-playbook work_dir/copy_configfile.yaml
```
然后在两个节点查看：

```bash
[root@master ~]# cat /tmp/test.conf     
ExecStart=/usr/local/bin/etcd --name master --initial-advertise-peer-urls http://192.168.101.14:2380
[root@node1 ~]# cat /tmp/test.conf
ExecStart=/usr/local/bin/etcd --name master --initial-advertise-peer-urls http://192.168.101.14:2380
```

可以看见在各个节点的tem目录下面的文件都用变量替换了。

### 3.2 使用template模块调用的j2文件使用`{% if %} {% endif %}`进行控制：

```bash
[root@master ansible]# cat roles/temp/templates/test_if.j2 
{% if ansible_hostname == master_hostname %}
ExecStart=/usr/local/bin/etcd --name {{ master_hostname }} --initial-advertise-peer-urls http://{{ master_ip }}:2380
{% elif ansible_hostname == node1_hostname %}
ExecStart=/usr/local/bin/etcd --name {{ node1_hostname }} --initial-advertise-peer-urls http://{{ node1_ip }}:2380
{% endif %}
```
在上面中使用if进行了判断，如果`ansible_hostname`变量与定义的`master_hostname`变量值相等，那么将此文件copy到节点上就使用条件1，而过不满足条件1那么执行条件2

`ansible_hostname`这个变量是setup模块中的值，是节点的固定值。

```bash
[root@master ~]# ansible all -m setup -a "filter=ansible_hostname"
192.168.101.15 | SUCCESS => {
    "ansible_facts": {
        "ansible_hostname": "node1"
    }, 
    "changed": false, 
    "failed": false
}
192.168.101.14 | SUCCESS => {
    "ansible_facts": {
        "ansible_hostname": "master"
    }, 
    "changed": false, 
    "failed": false
}
```
现在查看tasks下面的文件：

```bash
[root@master ansible]# cat roles/temp/tasks/main.yaml 
- name: copy configfile to nodes
  template:
    src: test_if.j2
    dest: /tmp/test.conf
```
将上面的test.j2改为了if条件的j2，然后执行：

```bash
[root@master ansible]# ansible-playbook work_dir/copy_configfile.yaml
```
查看各节点生成的文件内容：

```bash
[root@master ~]# cat /tmp/test.conf 
ExecStart=/usr/local/bin/etcd --name master --initial-advertise-peer-urls http://192.168.101.14:2380
[root@node1 ~]# cat /tmp/test.conf
ExecStart=/usr/local/bin/etcd --name node1 --initial-advertise-peer-urls http://192.168.101.15:2380
```
可以看见生成的文件内容不一样，于是这样就可以将节点的不同内容进行分离开了

当然还可以使用另外的方式隔离节点的不同：

```bash
ExecStart=/usr/local/bin/etcd --name {{ ansible_hostname }} --initial-advertise-peer-urls http://{{ ansible_ens33.ipv4.address }}:2380
```

因为各个节点的ansible_hostname和ip都是固定的所以也可以根据上面进行区分不同（不过这种方式限制了一定的范围）

### 3.3 使用template模块调用j2文件使用for循环：

 创建`jinja`关于for的文件：

```bash
 [root@master ansible]# cat roles/temp/templates/test_for.j2 
{% for i in range(1,10) %}
test{{ i }}
{% endfor %}
```

```bash
[root@master ansible]# cat roles/temp/tasks/main.yaml 
- name: copy configfile to nodes
  template:
    src: test_for.j2
    dest: /tmp/test.conf
```
执行该角色：

```bash
[root@master ansible]# ansible-playbook work_dir/copy_configfile.yaml

```
验证两节点的文件内容：

```bash
[root@master ~]# cat /tmp/test.conf 
test1
test2
test3
test4
test5
test6
test7
test8
test9
```

```bash
[root@node1 ~]# cat /tmp/test.conf 
test1
test2
test3
test4
test5
test6
test7
test8
test9
```
### 3.4 使用`default()`默认值

当我们定义了变量的值时，采用变量的值，当我们没有定义变量的值时，那么使用默认给定的值：

首先查看定义的变量：

```bash
[root@master ansible]# cat roles/temp/vars/main.yaml 
master_ip: 192.168.101.14
master_hostname: master
node1_ip: 192.168.101.15
node1_hostname: node1
```

然后查看jinja2的文件：

```bash
[root@master ansible]# cat roles/temp/templates/test_default.j2 
Listen: {{ server_port|default(80) }}
```

可以看见并没有定义server_port这个变量

查看tasks文件：

```bash
[root@master ansible]# cat roles/temp/tasks/main.yaml 
- name: copy configfile to nodes
  template:
    src: test_default.j2
    dest: /tmp/test.conf
```

执行完成后，查看文件内容：

```bash
[root@master ~]# cat /tmp/test.conf 
Listen: 80
```

现在向vars/main.yaml中定义server_port变量，并给定值：

```bash
[root@master ansible]# cat roles/temp/vars/main.yaml    
master_ip: 192.168.101.14
master_hostname: master
node1_ip: 192.168.101.15
node1_hostname: node1
server_port: 8080
```

再次执行，然后查看文件内容：

```bash
[root@master ~]# cat /tmp/test.conf 
Listen: 8080
```

可以看见使用了定义的值.

参考链接
[https://docs.ansible.com/ansible/latest/collections/ansible/builtin/template_module.html](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/template_module.html)
[https://www.cnblogs.com/jsonhc/p/7895399.html](https://www.cnblogs.com/jsonhc/p/7895399.html)
