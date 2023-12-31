

--
## 1. 介绍
set_fact模块在tasks中定义变量

## 2. 示例

### 2.1 定义并输出变量
set_fact.yaml
```bash
---
- hosts: localhost
  remote_user: root
  tasks:
  - set_fact:
      test: "123456"
  - debug:
      msg: "{{test}}"
```
执行输出：

```bash
ansible-playbook set_fact.yaml

PLAY [localhost] *******************************************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************************************************************************************************************************************************
ok: [localhost]

TASK [set_fact] ********************************************************************************************************************************************************************************************************************************************
ok: [localhost]

TASK [debug] ***********************************************************************************************************************************************************************************************************************************************
ok: [localhost] => {
    "msg": "123456"
}

PLAY RECAP *************************************************************************************************************************************************************************************************************************************************
localhost                  : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

### 2.2 返回值设置变量
set_fact2.yaml

```bash
- hosts: localhost
  remote_user: root
  vars:
    test1: "123456"
  tasks:
  - shell: echo "hello world"
    register: result
  - set_fact:
      test_one: "{{test1}}"
      test_two: "{{result.stdout}}"
  - debug:
      msg: " test_one is {{test_one}}; test_two is {{test_two}}"
```
输出：

```bash
$ ansible-playbook set_fact2.yaml 

PLAY [localhost] *******************************************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************************************************************************************************************************************************
ok: [localhost]

TASK [shell] ***********************************************************************************************************************************************************************************************************************************************
changed: [localhost]

TASK [set_fact] ********************************************************************************************************************************************************************************************************************************************
ok: [localhost]

TASK [debug] ***********************************************************************************************************************************************************************************************************************************************
ok: [localhost] => {
    "msg": " test_one is 123456; test_two is hello world"
}

PLAY RECAP *************************************************************************************************************************************************************************************************************************************************
localhost                  : ok=4    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```
### 2.3 跨play调用变量
[略]

