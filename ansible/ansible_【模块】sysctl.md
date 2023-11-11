


----
## 参数

```c
name：变量名
value：值
reload：文件被更新时，是否使用 sysctl -p reload 文件
state：是在文件中 移除(absent)或者设置(present)
sysctl_file：如果不是默认文件，指定其他文件
sysctl_set：使用sysctl 命令设置，不一定需要reload 文件
ignoreerrors: 默认值：no 类型：bool,使用此选项将忽略一些不知道的错误 key，即所设置的 name 参数
```

## 示例

```bash
EXAMPLES:
# Set vm.swappiness to 5 in /etc/sysctl.conf
- sysctl:
    name: vm.swappiness
    value: 5
    state: present

# Remove kernel.panic entry from /etc/sysctl.conf
- sysctl:
    name: kernel.panic
    state: absent
    sysctl_file: /etc/sysctl.conf

# Set kernel.panic to 3 in /tmp/test_sysctl.conf
- sysctl:
    name: kernel.panic
    value: 3
    sysctl_file: /tmp/test_sysctl.conf
    reload: no

# Set ip forwarding on in /proc and do not reload the sysctl file
- sysctl:
    name: net.ipv4.ip_forward
    value: 1
    sysctl_set: yes
# Set ip forwarding on in /proc and in the sysctl file and reload if necessary
- sysctl:
    name: net.ipv4.ip_forward
    value: 1
    sysctl_set: yes
    state: present
    reload: yes
```

