

## 1. 简介
用于管理服务
## 2. 选项
该模块包含如下选项： 

 - arguments：给命令行提供一些选项
 - enabled：是否开机启动 yes|no
 - name：必选项，服务名称
 - pattern：定义一个模式，如果通过status指令来查看服务的状态时，没有响应，就会通过ps指令在进程中根据该模式进行查找，如果匹配到，则认为该服务依然在运行
 - runlevel：运行级别
 - sleep：如果执行了restarted，在则stop和start之间沉睡几秒钟
 - state：对当前服务执行启动，停止、重启、重新加载等操作（started,stopped,restarted,reloaded）

## 3. 示例

```bash
ansible all -m shell -a "systemctl status NetworkManager.service |grep Active"
ansible all -m systemd -a "name=NetworkManager state=stopped"
ansible all -m systemd -a "name=NetworkManager enabled=no"
ansible all -m shell -a "systemctl status NetworkManager.service |grep Active"
```

