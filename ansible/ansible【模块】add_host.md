add_host
在playbook执行的过程中，动态的添加主机到指定的主机组中
常用参数：

 - groups：添加主机至指定的组
 - name：要添加的主机名或IP地址

示例：

```bash
- name: add a host to group webservers
  hosts: webservers
  tasks:
    - add_host name={{ ip_from_ec2 }} group=webservers foo=42    #添加主机到webservers组中，主机的变量foo的值为42
```

