## 参数

```bash
msg  打印的自定义消息
var  要调试的变量名。与msg选项互斥。
verbosity   一个控制调试运行时间的数字，如果设置为3，则仅在-vvv或更高版本时运行调试
```
示例：

```bash
---

 - name: talk to all hosts just so we can learn about them
   hosts: all
   vars:
     password_used: '123456'
   tasks: 
   - name: Print the gateway for each host when defined
     debug:
       msg: System {{ inventory_hostname }} has gateway {{ ansible_default_ipv4.gateway }}
     when: ansible_default_ipv4.gateway is defined

   - name: Get uptime information
     shell: /usr/bin/uptime
     register: result

   - name: Print return information from the previous task
     debug:
       var: result
       verbosity: 3

   - name: Display all variables/facts known for a host
     debug:
       var: hostvars[inventory_hostname]
       verbosity: 4

   - name: Prints two lines of messages, but only if there is an environment value set
     debug:
       msg:
       - "Provisioning based on YOUR_KEY which is: {{ lookup('env', 'YOUR_KEY') }}"
       - "These servers were built using the password of '{{ password_used }}'. Please retain this for later use." 
```

执行输出：

```bash
$ ansible-playbook debug.yaml 

PLAY [talk to all hosts just so we can learn about them] ***************************************************************************************************************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************************************************************************************************************************************************
ok: [192.168.211.62]
ok: [192.168.211.61]
ok: [192.168.211.60]

TASK [Print the gateway for each host when defined] ********************************************************************************************************************************************************************************************************
ok: [192.168.211.60] => {
    "msg": "System 192.168.211.60 has gateway 192.168.211.2"
}
ok: [192.168.211.61] => {
    "msg": "System 192.168.211.61 has gateway 192.168.211.2"
}
ok: [192.168.211.62] => {
    "msg": "System 192.168.211.62 has gateway 192.168.211.2"
}

TASK [Get uptime information] ******************************************************************************************************************************************************************************************************************************
changed: [192.168.211.62]
changed: [192.168.211.61]
changed: [192.168.211.60]

TASK [Print return information from the previous task] *****************************************************************************************************************************************************************************************************
skipping: [192.168.211.60]
skipping: [192.168.211.61]
skipping: [192.168.211.62]

TASK [Display all variables/facts known for a host] ********************************************************************************************************************************************************************************************************
skipping: [192.168.211.60]
skipping: [192.168.211.61]
skipping: [192.168.211.62]

TASK [Prints two lines of messages, but only if there is an environment value set] *************************************************************************************************************************************************************************
ok: [192.168.211.60] => {
    "msg": [
        "Provisioning based on YOUR_KEY which is: ", 
        "These servers were built using the password of '123456'. Please retain this for later use."
    ]
}
ok: [192.168.211.61] => {
    "msg": [
        "Provisioning based on YOUR_KEY which is: ", 
        "These servers were built using the password of '123456'. Please retain this for later use."
    ]
}
ok: [192.168.211.62] => {
    "msg": [
        "Provisioning based on YOUR_KEY which is: ", 
        "These servers were built using the password of '123456'. Please retain this for later use."
    ]
}

PLAY RECAP *************************************************************************************************************************************************************************************************************************************************
192.168.211.60             : ok=4    changed=1    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0   
192.168.211.61             : ok=4    changed=1    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0   
192.168.211.62             : ok=4    changed=1    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0   
```



