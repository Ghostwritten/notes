
##  执行顺序
![在这里插入图片描述](https://img-blog.csdnimg.cn/ed5279bc015342aa9766968906703b73.png)
1.检查play中是否存在pre_tasks定义，存在的话则顺序执行pre_tasks中定义的所有tasks
2.如果存在pre_tasks定义，则检查是否存在触发handler，如存在则顺序执行相关触发handlers
3.检查是否存在roles定义，如存在则顺序执行roles下的所有tasks
4.检查是否存在task, 如存在则顺序执行所有定义的task
5.检查roles和task中是否存在触发handler，如存在顺序执行
6.检查是否存在post_tasks定义，存在则顺序执行post_tasks中定义的所有tasks
7.如果存在post_tasks，则检查post_tasks下面的tasks是否存在触发handlers,如存在则顺序执行

一个包含pre_tasks, roles和post_tasks的实例

```bash
---
- hosts: tag_ansible_group_windows_webservers
  serial: 1
  gather_facts: False
  connection: winrm
  vars:
    ansible_ssh_port : 5986

  # These are the tasks to run before applying updates:
  pre_tasks:
  - name: Remove host from load balancing pool
    local_action:
      module: ec2_elb
      region: us-east-1
      instance_id: "{{ ec2_id }}"
      ec2_elbs: "ansible-windows-demo-lb"
      wait_timeout: 330
      state: 'absent'

  roles:
  - web

  # These tasks run after the roles:
  post_tasks:
  - name: Wait for webserver to come up
    local_action: wait_for host={{ inventory_hostname }} port=80 state=started timeout=80

  - name: Add host to load balancing pool
    local_action:
      module: ec2_elb
      region: us-east-1
      instance_id: "{{ ec2_id }}"
      ec2_elbs: "ansible-windows-demo-lb"
      wait_timeout: 330
      state: 'present'


```
##  指定执行
官方大致提供如下几个特性：对于测试或调试新的playbook很有帮助.
- 1：tag
- 2：start-at
- 3：skip-tags
- 4：step

Demo playbook:

```bash
---
- name: shutdown etcd
  service: name=etcd state=stopped enabled=no
  ignore_errors: yes
  tags:
      - shutdown

- name: del etcd dir
  shell: 'rm -rf {{ item }}'
  with_items:
      - { ETCD_DIR }
  tags:
      - deldir

- name: create etcd dir
  file:
       path: '{{ item }}'
       state: directory
       mode: 755
  with_items:
      - { ETCD_DIR }
  tags:
      - mkdir
```

### --tags
如果你只想运行 playbook 中的”shutdown”和”mkdir”，你可以这样做

```bash
ansible-playbook example.yml – tags “shutdown,mkdir”
ansible-playbook -i inventory/local/inventory.ini --become --become-user=root cluster.yml --tags  etchosts
```
tag 特性是一个不错的功能，但如果真的是要维护一个大型的 playbook，还是建议将 playbook 按功能或应用拆分成多个 playbook，然后再在主 playbook include 其他子 playbook，这样即既利于维护也方便管理.


### –start-at
从指定任务开始运行palybook以及分步运行playbook,如果你想从指定的任务开始执行playbook,可以使用–start-at选项:
以下命令就会在名为”deldir”的任务开始执行你的playbook.

```bash
ansible-playbook playbook.yml --start-at="deldir"
```
###  --skip-tags 
如果你只想执行 playbook 中某个特定任务之外的所有任务，你可以这样做：

```bash
ansible-playbook example.yml – skip-tags “deldir”
```
###  --step

```bash
ansible-playbook playbook.yml --step

比如你有个名为``deldir``的任务,playbook执行到这里会停止并询问:

Perform task: deldir (y/n/c):

“y”回答会执行该任务,
”n”回答会跳过该任务,
而”c”回答则会继续执行剩余的所有任务而不再询问你.
```


###  混合

```bash
ansible-playbook playbook.yml --tags myrole --start-at-task "task_name"
```

参考：
- [ansible-playbook运行步骤调度](https://blog.51cto.com/michaelkang/2415453)
