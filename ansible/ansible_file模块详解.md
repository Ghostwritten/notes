## file模块
专门用来设定文件属性；

```bash
force：需要在两种情况下强制创建软链接，一种是源文件不存在，但之后会建立的情况下；另一种是目标软链接已存在，需要先取消之前的软链，然后创建新的软链，有两个选项：yes|no
group：定义文件/目录的属组
mode：定义文件/目录的权限
owner：定义文件/目录的属主
path：必选项，定义文件/目录的路径
recurse：递归的设置文件的属性，只对目录有效
src：要被链接的源文件的路径，只应用于state=link的情况
dest：被链接到的路径，只应用于state=link的情况

state：
=directory：如果目录不存在，创建目录
=file：即使文件不存在，也不会被创建
=link：创建软链接
=hard：创建硬链接
=touch：如果文件不存在，则会创建一个新的文件，如果文件或目录已存在，则更新其最后修改时间
=absent：删除目录、文件或者取消链接文件
```

创建文件

```bash
$ ansible test -m file -a "path=/tmp/test state=touch owner=root group=root"
```

创建目录：

```bash
$ ansible test -m file -a 'path=/tmp/www state=directory'
$ ansible test -m file -a "path=/tmp/tdir state=directory mode=0755"
```

