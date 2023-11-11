kubernetes apparmor 语法
tags: apparmor,安全
<!-- catalog:~语法~ -->



[![在这里插入图片描述](https://img-blog.csdnimg.cn/c7297f9f01e64f33b5a240cb9fca127c.png)](https://www.behance.net/gallery/75682725/RIGHT-A-Reverse-Film)


上一篇：[kubernetes apparmor 入门](https://ghostwritten.blog.csdn.net/article/details/126229173)

##  1. 简介
AppArmor 策略是使用管理员友好的配置文件语言创建的，然后将其编译为二进制策略以加载到内核中。AppArmor 策略存储在
`/etc/apparmor.d/`中的一组文件中。AppArmor 策略分为配置文件，这些配置文件通常旨在限制特定的应用程序。配置文件将声明访问规则以允许访问资源，并且当配置文件中没有匹配规则时，通过日志记录隐式拒绝访问。

配置文件以配置文件名称开头，后跟可选标志字段，然后是开头`{`后跟配置文件规则，并以结束`}`结束如果配置文件名称不以/开头，则应在前面加上关键字配置文件. 例如：

```bash
 /usr/bin/firefox {
    # profile contents
 }

 /usr/bin/firefox flags=(complain) {
    # profile contents
 }

 profile /usr/bin/ {
    # profile contents
 }

 profile user1 {
    # profile contents
 }

```
配置文件名称可以包含文件规则通配符，以允许它们应用于多个可执行文件。

##  2. 注释

```bash
 #  Comment 1
 #  Comment 2

 profile example {  # comment 3
    # comment 4
    /home/foo rw,  # comment at the end of a file rule
 }

```
##  3. Include Rules

```bash
 #include <file>
 #include “a/relative/path/file”
 include <file>
```
#包含与评论规则冲突并优先。#和`include`不能与空格分开，否则将被视为注释

```bash
# include is a comment
 #include <file>
```
##  4. Child profiles
配置文件可以包含子配置文件。子配置文件可用于以特殊方式限制应用程序，或者当您希望子在系统上不受限制，但在从父调用时被限制。例如：

```bash
 /parent/profile {
     /path/to/child1 cx -> child1,
     /path/to/child2 cx,
     /path/to/* cx,           # for * matching child3 will transition to child3,
                              # child4, child5, ... will transition to /path/to/child*
                              # if matching child profile does not exist will fail
     /another/path/to/* cx -> child1,        # send all matching execs to child1
     profile child1 {
     }
     profile /path/to/child2 {
     }
     profile /path/to/child3 {
     }
     # generic fall back profile
     profile /path/to/child* {
     }
  }
```
##  5. Hats
`Hats` 是一个特殊的子配置文件，可以与 `change_hat API` 调用一起使用。要表示帽子，请在帽子名称前加上`^`，不要有空格。例如：

```bash
 /parent/profile {
     ^hat {
     }
 }
```
##  6. Capability Rules
`AppArmor` 支持粗粒度访问 Linux 的 POSIX 风格的功能（请参阅“man 功能”），并且功能规则用于允许访问这些功能。例如，删除权限的 setuid 应用程序可能需要.

```bash
 /profile {
    capability setuid,
    capability setgid,
 }

```
##  7. Network Rules
`AppArmor` 目前支持通过网络规则对网络进行粗粒度访问。例如，网络守护程序可能需要：

```bash
 /profile {
   network inet dgram,
   network inet stream,
 }
```
或者数据包分析器可能需要：

```bash
 /profile {
   network raw,
   network packet,
 }

```
##  8. File rules
文件规则控制文件的访问方式，并且只出现在配置文件中。它们由路径名、权限集组成，并以逗号结尾。它们可以先写权限，也可以先写路径名，尽管约定是先列出路径。有效的路径名总是以/开头。例如：

```bash
 /profile {
    /path/to/file  rw,   # file rule beginning with a pathname (convention)
    rw /path/to/file2,   # file rule beginning with permissions
    /path/to/file3       # file rule split over multiple lines
         rw,
 }

```
文件规则可以包含允许匹配多个文件的特殊通配符

###  9. 文件通配
AppArmor 使用类似于 bash shell 使用的文件通配语法。通配符不是标准的完整正则表达式语法，而是使用一些称为通配符的字符。AppArmor 通配符的语义与 bash 的语义略有不同。

 - `*` - 在目录级别匹配零个或多个字符。当将路径视为字符串时，它将匹配除/之外的每个字符
   - 这将匹配点文件（文件名以.开头），特殊点文件除外。和..，如果它紧跟在目录 / 例如之后。/目录/*
   - 这将不匹配空目录字符串，例如。/目录//
   - pcre 等价于 ([^/\000]*)
- `**` - 在多个目录级别上匹配 0 个或多个字符。
   - 这将匹配点文件（文件名以.开头），特殊点文件除外。和..，如果它紧跟在目录 / 例如之后。/目录/**
  - pcre 等价于 ([^\000]*)

- `?` - 匹配不是/的单个字符
   - pcre 等价于 [^/]
- `{}` - 替代 - 可以匹配的替代字符串的逗号分隔列表。允许使用空字符串，这意味着空字符串是可行的替代方案
  - pcre 等价于 (|)
- `[]` - 字符类
  - 与 pcre 语法相同
- `[^]` - 反转字符类
   - 与 pcre 语法相同
 - 交替嵌套表达式（从 AppArmor 2.3 开始）：
   - 转义字符 `\*`
   - 将字符表示为# \001

以下是对当前文件通配的建议添加，当前未实施：

 - `{*^}` - 一个类似于 * 的 glob，具有不允许匹配的事物的交替样式列表。例如。`/etc/{*^shadow}` 与允许 `/etc/*` 匹配的所有内容相同，除了 `/etc/shadow` 例如。`/etc/{*^shadow,passwd}` 与 `/etc/*` -  `/etc/{shadow,passwd}` 相同，例如。`/etc/{*^*shadow}` 与 `/etc/*` 相同 - `/etc/*shadow *` 请注意，不允许使用空的替代条目，即。`{*^shadow,}`

- {**^} - 类似于 ** 的 glob 不允许匹配具有交替样式列表的事物，例如。`/etc/{**^shadow}` 与 /etc/** 匹配 - `/etc/shadow` 例如。`/etc/{**^shadow,passwd}` 与 `/etc/** - /etc/{shadow,passwd}` 相同，例如。`/etc/{**^*shadow}` 与 `/etc/**` 相同 - `/etc/*shadow *` 请注意，不允许使用空的替代条目，即。`{*^shadow,}`


###  9.1 文件通配示例

```bash
/dir/file     - match a specific file
/dir/*        - match any files in a directory (including dot files)
/dir/a*       - match any file in a directory starting with 'a'
/dir/*.png    - match any file in a directory ending with '.png'
/dir/[^.]*    - 匹配一个目录中除点文件以外的任何文件

/dir/         - match a directory
/dir/*/       - match any directory within /dir/
/dir/a*/      - match any directory within /dir/ starting with a
/dir/*a/      - match any directory within /dir/ ending with a

/dir/**       - 匹配/dir/中或之下的任何文件或目录
/dir/**/      - 匹配 /dir/ 中或之下的任何目录
/dir/**[^/]   - 匹配/dir/目录中或之下的任何文件

/dir{,1,2}/** - match any file or directory in or below /dir/, /dir1/, and /dir2/
```

###  9.2 文件权限
支持以下文件权限：

 - r - 读
 - w - 写
 - a - 附加（由 w 暗示）
 - x - 执行
   - ux - 执行无限制（保留环境） - 警告：只应在非常特殊的情况下使用
   - Ux - 无限制执行（清理环境）
   - px - 在特定配置文件下执行（保留环境） - 警告：仅应在特殊情况下使用
   - Px - 在特定配置文件下执行（清理环境）
   - pix - 为 px，但如果找不到目标配置文件，则回退到继承当前配置文件
   - Pix - 作为 Px，但如果找不到目标配置文件，则回退到继承当前配置文件
   - pux - 作为 px 但如果找不到目标配置文件则回退到执行 unconfined
   - Pux - 作为 Px，但如果找不到目标配置文件，则回退到执行 unconfined
   - ix - 执行并继承当前配置文件
   - cx - 执行并转换到子配置文件（保护环境）
   - Cx - 执行并转换到子配置文件（清理环境）
   - cix - 作为 cx，但如果找不到目标配置文件，则回退到继承当前配置文件
   - Cix - 作为 Cx，但如果找不到目标配置文件，则回退到继承当前配置文件
   - cux - 作为 cx 但如果找不到目标配置文件则回退到执行 unconfined
   - Cux - 与 Cx 相同，但如果未找到目标配置文件，则回退到执行 unconfined


- m - 内存映射可执行文件
- k - 锁定（需要 r 或 w，AppArmor 2.1 及更高版本）
- l - 链接

`owner` 关键字可以用作限定词，使权限以拥有文件为条件（进程 fsuid == 文件的 uid）。

```bash
 owner /foo rw,
```
以下内容正在开发中：

 - 创建（由 w 暗示）
 - 删除（由 w 暗示）
 - chown - 更改所有权（由 w 暗示）
 - chmod - 改变模式（由 w 暗示）

创建和/或删除文件的权限是：

```bash
 /foo/bar      w,
```

复制文件的权限是：

```bash
 /foo/src      r,
 /foo/dst      w,
```


移动文件的权限是：

```bash
 /foo/src     rw,
 /foo/dst      w,
```

###  9.3 执行权限
AppArmor 区分文件执行的不同方式。因为在执行文件时会创建一个新进程，所以可以说该进程在执行过程中转换到另一个（可能相同）配置文件。
基本执行权限是：

 - `ix` - 新进程应该在当前配置文件下运行
 - `cx` - 新进程应在与可执行文件名称匹配的子配置文件下运行
 - `px` - 新进程应在与可执行文件名称匹配的另一个配置文件下运行
 - `ux` - 新进程应该不受限制地运行

一个无限制运行的进程实际上是在内置的无限制配置文件中，它允许一切没有日志记录。

使用大写前导字符（ Px、Cx、Ux ）编写的 px、cx 和 ux 权限将触发 `libc` 的安全执行。开发配置文件时，通常应使用安全执行变体，以便执行的程序在干净的环境中启动。

px 和 cx 规则（及其安全执行变体）也可能具有 ix 或 ux 后备，表示为 pix、pux、cix 或 cux。使用回退表示如果存在配置文件，则该进程应在​​配置文件下运行，否则配置文件转换应使用指定的 ix 或 ux 转换。使用“PUx”而不是“Ux”通常是个好主意，这样当执行的程序稍后添加 AppArmor 配置文件时，您不必更新配置文件。例如，如果应允许受限程序运行“evince”，则配置文件可能具有：

```bash
 /usr/bin/evince PUx,
```
px 和 cx 规则（及其所有变体）也可以修改为按名称指定配置文件，而不是使用与可执行文件名称匹配的配置文件。这是通过提供`->`转换箭头和配置文件的名称来完成的。

```bash
 /foo px -> profile1,
```
对于目录，UNIX 执行权限映射到搜索访问，`AppArmor` 不会进一步控制目录搜索访问。换句话说，如果 DAC 允许，则允许遍历目录。

###  9.4 AppArmor 文件标签
AppArmor 将为文件分配一个默认标签（而不是将该标签存储在文件的 inode 中）。当一个进程打开一个文件时，文件对象被分配一个标签，它可以被认为是配置文件的名称。当不同的进程想要访问同一个文件时，一个文件可以有多个标签，以允许不同的进程具有不同的访问控制。实际上，在开发策略时，只需按名称引用文件，内核在幕后处理所有必要的标签。因此，AppArmor 通常被称为“基于路径名”。
由于 AppArmor 按路径名标记文件（而不是在磁盘上标记），因此管理员无需在文件被覆盖或移动后执行重新标记步骤。例如，如果一个进程被授予对 /etc/shadow 的读取权限并且系统管理员将 /etc/shadow 重命名为 /etc/shadow.old 并用一个副本替换它（例如，其中可能有一个额外的用户） ，该进程将有权访问新的 /etc/shadow，而不是 /etc/shadow.old。


##  10. Rule 修饰符
当某个资源没有对应的规则时，AppArmor 会阻止对该资源的访问并记录下来。当策略中有规则时，允许访问资源而无需记录。可以在规则前添加以下修饰符来更改此行为：

 - audit：强制记录
 - deny：明确拒绝，不记录
 - audit deny：明确拒绝的组合，但记录

实例

```bash
/profile {
    /path/to/file*            r,  # allow read to /path/to/file*
    /path/to/file1            w,  # allow write to /path/to/file1
    deny /path/to/file2,      w,  # deny write to /path/to/file2, without logging
    audit /path/to/file3      w,  # allow write to /path/to/file3, with logging
    audit deny /path/to/file4 r,  # deny read to /path/to/file4, with logging
 }

```

> 重要提示：拒绝规则在允许规则之前进行评估，并且不能被允许规则覆盖。它们通常用于覆盖文件通配规则。例如，在上述策略中，上面的“`audit deny /path/to/file4 r`”规则会覆盖“`/path/to/file* r`”规则。

参考：

 - [apparmor Documation](https://gitlab.com/apparmor/apparmor/-/wikis/Documentation)
 - [How to create an AppArmor Profile](https://ubuntu.com/tutorials/beginning-apparmor-profile-development#1-overview)

