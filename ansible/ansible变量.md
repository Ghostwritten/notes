


---
## 1. 变量优先级

变量优先级由小到大排列（优先级大的变量可以覆盖优先级小的变量）：

```powershell
command line values (eg “-u user”)
role defaults [1]
inventory file or script group vars [2]
inventory group_vars/all [3]
playbook group_vars/all [3]
inventory group_vars/* [3]
playbook group_vars/* [3]
inventory file or script host vars [2]
inventory host_vars/* [3]
playbook host_vars/* [3]
host facts / cached set_facts [4]
play vars
play vars_prompt
play vars_files
role vars (defined in role/vars/main.yml)
block vars (only for tasks in block)
task vars (only for the task)
include_vars
set_facts / registered vars
role (and include_role) params
include params
extra vars (always win precedence)
```


## 2. 命令行参数extra_vars配置变量
--extra_vars = -e

```powershell
ansible test70 -e "pkg=httpd" -m yum -a "name=httpd state=present"
```

```yaml
$ vim install.yml
 ---
- hosts: node
  remote_user: root
  tasks:
    - name: install httpd service
      yum: name={{ pkg }} state=present
```


     
```bash
ansible-playbook install.yml --extra-vars " pkg = httpd"
ansible-playbook install.yml -i <指定ip或组> -e " pkg = httpd"
```

## 3. inventory 主机清单中定义的变量

```bash
$ cat /etc/ansible/hosts
[load-node]
openstack-load1 
openstack-load2

[compute-node]
openstack-compute1 ansible_ssh_host=10.0.1.10 ansible_ssh_port=2002 ansible_ssh_user=stanley ansible_ssh_pass=etyfhzmweadf
openstack-compute2

[control-node]
openstack-control1 filename=control1.txt    # 主机变量
openstack-control2 filename=control2.txt

[openstack:children]
load-node
compute-node
control-node

[openstack:vars]
issue="Hello, World"    # 组变量
```

## 4. ansible-playbook中vars、vars_files定义的变量

### 4.1 vars

```yaml
$ vim yaml/vars.yaml
- hosts: compute-node
  remote_user: root
  vars:
    pkg: httpd        # 定义变量
  tasks:
    - name: install httpd service
      yum: name={{ pkg }} state=present
```
### 4.2 vars_files
```powershell
$ vim  /tmp/var.yaml
pkg: httpd 
```

```yaml
$ vim install.yml
---
- hosts: all
  gather_facts: False
  vars_files:
    - /tmp/var.yaml
  tasks:
    - name: install httpd service
      yum: name={{ pkg }} state=present
```
多变量

```yaml
$ vim hello.yaml 
---
- name: hosts ip
  hosts: 192.168.1.190
  remote_user: root
  vars:
    client: liming 
    country: china
 
  tasks:
  - name: "echo a word"
    debug:
      msg: "Hello World, {{client}}, my {{country}} friend!"
```

```powershell
$ ansible-playbook hello.yaml --syntax-check
$ ansible-playbook hello.yaml
PLAY [hosts ip] ****************************************************************************************************************************

TASK [Gathering Facts] *********************************************************************************************************************
ok: [192.168.1.190]

TASK [echo a word] *************************************************************************************************************************
ok: [192.168.1.190] => {
    "msg": "Hello World, liming, my china friend!"
}

PLAY RECAP *********************************************************************************************************************************
192.168.1.190              : ok=2    changed=0    unreachable=0    failed=0   
```

```powershell
$ ansible-playbook hello.yaml -e "client=xiaogang, country=japan"
$ ansible-playbook hello.yaml -e '{"client": "xiaogang", "country": "japan"}'
```

变量值为列表

```yaml
$ vim hello.yaml
---
- name: hosts ip
  hosts: localhost
  remote_user: root
  vars:
    client: liming 
    country: china
 
  tasks:
  - name: "echo a word"
    debug:
      msg: "Hello World, {{client[0]}} {{client[1]}} and {{client[0]}},my {{country}} friends!"
```

```powershell
$ ansible-playbook hello.yaml -e '{"client": ["xiaohua""john","hugo"], "country": "japan"}'
```

```yaml
$ vim clients
country: china
client:
- liming
- xiaogang
- xuanzi
```

```powershell
ansible-playbook hello.yaml -e "@clients"
```

### 4.3 vars_prompt(可实现人机交互）

```yaml
$ vim print.yml
---
- hosts: test70
  remote_user: root
  vars_prompt:
    - name: "your_name"
      prompt: "What is your name"
      private: no   #当该值为yes，则用户的输入不会被打印，否则反之。
    - name: "your_age"
      prompt: "How old are you"
      private: no
  tasks:
   - name: output vars
     debug:
      msg: Your name is {{your_name}},You are {{your_age}} years old.
```

## 5. 注册变量

```yaml
$ vim date.yaml
---
- hosts: localhost
  remote_user: root
  tasks:
    - name: show date
      shell: "/bin/date"
      register: date        # 注册一个变量
    - name: Record time log
      shell: "echo {{ date.stdout }} > /tmp/date.log"
```

```powershell
$ ansible-playbook date.yaml 
$ cat /tmp/date.log 
Mon Feb 17 00:23:47 CST 2020
```


