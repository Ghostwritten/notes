yum

 - config_file：yum的配置文件
 - disable_gpg_check：关闭gpg_check
 - disablerepo：不启用某个源
 - enablerepo：启用某个源
 - name：要进行操作的软件包的名字，也可以传递一个url或者一个本地的rpm包的路径
 - state：状态（present，absent，latest）

删除软件包

```bash
ansible t1 -m yum -a 'name="lrzsz" state=absent'
```

删除多个软件包

```bash
ansible t1 -m yum -a 'name="lrzsz,lftp" state=absent'
```

安装软件包

```bash
ansible t1 -m yum -a 'name="lrzsz"'
```

安装多个软件包

```bash
ansible t1 -m yum -a 'name="lrzsz,lftp"'
```

