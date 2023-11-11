


----
## 1. 简介
Ansible允许你成为另一个用户，与登录到本机的用户或远程用户不同。这是使用现有的特权升级工具（`privilege escalation tools`）完成的，您可能已经使用或已经配置了这些工具，如`sudo，su，pfexec，doas，pbrun，dzdo，ksu`等。
说明：

 - （1）在`1.9
   Ansible`之前，大多数情况下都允许使用`sudo`和有限的`su`来允许登录/远程用户成为不同的用户并执行任务，用第二个用户的权限创建资源。从`1.9`开始`become`代替旧的sudo /su，同时仍然向后兼容。这个新系统也使得添加诸如pbrun（Powerbroker），pfexec，dzdo（Centrify）等其他特权升级工具变得更加容易。
 - （2）变量和指令是独立的，即设置`become_user`并不是设置become。 Ansible中

 
## 2. become的使用

### 2.1 become

> set to ‘true’/’yes’ to activate privilege escalation.

使用“true”或“yes”来表示启用这个特权，如：`become=true`
表示打开了become开关。
### 2.2 become_user

> set to user with desired privileges — the user you ‘become’, NOT the user you login as. Does NOT imply become: yes, to allow it to be set at host level.

`become_user=root` 设置为root账户，相当于我们以普通账户登入到远程主机时，再使用`su - root`切换为root账户。
### 2.3 become_method

> (at play or task level) overrides the default method set in ansible.cfg, set to sudo/su/pbrun/pfexec/doas/dzdo/ksu

`become_method=su` 表示用什么方式将普通账户切换到root或所需的其他账户，这里可以用su或sudo。
### 2.4 become_flags

> (at play or task level) permit to use specific flags for the tasks or role. One common use is to change user to nobody when the shell is set to no login. Added in Ansible 2.2.

表示允许为任务或角色使用特定的标志。一个常见的用法是在shell设置为不登录时将用户更改为`nobody`。`ansible2.2`版本中增加。
Ansible中become的使用举例
说明：
例如，要以非root用户身份连接到服务器时，需要root用户权限：
（1）To run a command as the apache user:（ 以apache账户运行命令），play.yml脚本如下：

```bash
name: Run a command as the apache user
command: somecommand
become: true
become_user: apache
```

（2）To do something as the nobody user when the shell is nologin:（在shell设置为不登录时将用户更改为nobody），play.yml脚本如下：

```bash
name: Run a command as nobody
command: somecommand
become: true
become_method: su
become_user: nobody
become_flags: '-s /bin/sh'
```

## 3. become变量在hosts使用
说明：允许您设置每个组和/或主机的选项，这些选项通常在hosts中定义，但可以用作正常变量来使用。
### 3.1 ansible_become
equivalent of the become directive, decides if privilege escalation is used or not.（相当于成为指令，决定是否使用特权升级。）
### 3.2 ansible_become_method
allows to set privilege escalation method（允许设置权限升级方法）
### 3.3 ansible_become_user
allows to set the user you become through privilege escalation, does not imply ansible_become: True
（允许通过权限升级来设置你成为用户，记得同时使用ansible_become：true）
### 3.4 ansible_become_pass
allows you to set the privilege escalation password
（即如你要使用root账户，则这里要写的就是root账户的密码！）
举例如下：

```bash
[root@iZ2ze5n6gzuzcclehq1v9gZ ansible]# vim hosts
[yunwei]        ##运维 wtf
192.168.2.1 ansible_ssh_user=product  ansible_become_user=root ansible_become=true  ansible_become_pass='123456'
```

