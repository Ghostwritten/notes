#  Linux Command locate 查询文件名



---
## 1. 简介
`locate`命令其实是`find -name`的另一种写法，但是要比后者快得多，原因在于它不搜索具体目录，而是搜索一个数据库文件，这个数据库中含有本地所有文件信息。Linux系统自动创建这个数据库，并且每天自动更新一次，所以使用locate命令查不到最新变动过的文件。为了避免这种情况，可以在使用locate之前，先使用`updatedb`命令，手动更新数据库。使用updatedb更新数据库时会占用系统的资源，特别是对特别复杂的目录或要文件较多时，要进行更新数据时，可选择系统空闲的时间段进行。
## 2. 语法

```bash
locate [选择参数] [样式]
```

## 3. 选项

```c
-b, --basename  # 仅匹配路径名的基本名称
-c, --count     # 只输出找到的数量
-d, --database DBPATH # 使用DBPATH指定的数据库，而不是默认数据库 /var/lib/mlocate/mlocate.db
-e, --existing  # 仅打印当前现有文件的条目
-1 # 如果 是 1．则启动安全模式。在安全模式下，使用者不会看到权限无法看到 的档案。这会始速度减慢，因为 locate 必须至实际的档案系统中取得档案的  权限资料。
-0, --null            # 在输出上带有NUL的单独条目
-S, --statistics      # 不搜索条目，打印有关每个数据库的统计信息
-q                    # 安静模式，不会显示任何错误讯息。
-P, --nofollow, -H    # 检查文件存在时不要遵循尾随的符号链接
-l, --limit, -n LIMIT # 将输出（或计数）限制为LIMIT个条目
-n                    # 至多显示 n个输出。
-m, --mmap            # 被忽略，为了向后兼容
-r, --regexp REGEXP   # 使用基本正则表达式
    --regex           # 使用扩展正则表达式
-q, --quiet           # 安静模式，不会显示任何错误讯息
-s, --stdio           # 被忽略，为了向后兼容
-o                    # 指定资料库存的名称。
-h, --help            # 显示帮助
-i, --ignore-case     # 忽略大小写
-V, --version         # 显示版本信息
```

## 4. 实战
### 4.1 查找和pwd相关所有文件

```bash
$  locate pwd
/bin/pwd
/etc/.pwd.lock
/sbin/unix_chkpwd
/usr/bin/pwdx
/usr/include/pwd.h
/usr/lib/python2.7/dist-packages/twisted/python/fakepwd.py
/usr/lib/python2.7/dist-packages/twisted/python/fakepwd.pyc
/usr/lib/python2.7/dist-packages/twisted/python/test/test_fakepwd.py
/usr/lib/python2.7/dist-packages/twisted/python/test/test_fakepwd.pyc
/usr/lib/syslinux/pwd.c32
/usr/share/help/C/empathy/irc-join-pwd.page
/usr/share/help/ca/empathy/irc-join-pwd.page
/usr/share/help/cs/empathy/irc-join-pwd.page
/usr/share/help/de/empathy/irc-join-pwd.page
/usr/share/help/el/empathy/irc-join-pwd.page
```

### 4.2  搜索etc目录下所有以sh开头文件

```bash
$ locate /etc/sh
/etc/shadow
/etc/shadow-
/etc/shells
```

### 4.3 搜索刚创建的文件

```bash
$ touch new.txt
$ locate new.txt
$ updatedb
$ locate new.txt
/root/new.txt
```
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/900d346970ce409807fa2898d285f53c.gif#pic_center)

