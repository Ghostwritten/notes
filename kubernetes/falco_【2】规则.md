#  falco 规则
tags: falco,安全




[![在这里插入图片描述](https://img-blog.csdnimg.cn/3d3023c98b784393ad375cead0c8c739.png)](https://www.rottentomatoes.com/m/cloud_atlas_2012)

{% youtube %}
https://www.youtube.com/watch?v=M3f6-ioY9rs
{% endyoutube %}


上篇文章我们对[falco 进行简单的入门](https://ghostwritten.blog.csdn.net/article/details/126095764)，其中涉及到、架构、特点、安装、部署、运行等，接下来我们了解 falco 的 rules。


## 1. 简介
Falcorule文件是包含三种元素的YAML文件：
| 元素 | 描述                                             |
|----|------------------------------------------------|
| Rules| 应生成警报的条件。rule伴随着与警报一起发送的描述性输出字符串。                |
| Macros  | 可以在rule甚至其他宏中重复使用的rule条件片段。宏提供了一种命名常见模式和排除rule冗余的方法。 |
| Lists | 可以包含在rule、宏或其他列表中的项目集合。与rule和宏不同，列表不能被解析为过滤表达式     |

还有两个与版本控制相关的可选元素：
| 元素                       | 描述                         |
|--------------------------|----------------------------|
| required_engine_version  | 用于跟踪rule内容和 falco引擎版本之间的兼容性。 |
| required_plugin_versions | 用于跟踪rule内容和插件版本之间的兼容性。       |


## 2. 版本控制
有时，我们会更改不向后兼容旧版本 Falco 的rule文件格式。类似地，libsinsp 和 libscap 可以定义新的 filtercheck 字段、操作符等。我们想要表示一组给定的rule依赖于这些库中的字段/操作符。从 Falco 版本`0.14.0` 开始，Falco rule支持 Falco 引擎和 Falco rule文件的显式版本控制。

### 2.1 Falco 引擎版本控制
falco可执行文件和falco_engineC++ 对象现在支持返回版本号。初始版本是 2（暗示之前的版本是 1）。每当我们对rule文件格式进行不兼容的更改或向 Falco 添加新的过滤检查字段/运算符时，我们都会增加此版本。

### 2.2 Falco rule文件版本控制
Falco 包含的 Falco rule文件包括一个新的顶级对象 ，required_engine_version: N它指定读取此rule文件所需的最低引擎版本。如果不包括在内，则在读取rule文件时不执行版本检查。这是一个例子：

```bash
# This rules file requires a falco with falco engine version 7.
- required_engine_version: 7
```

如果rule文件的engine_version版本高于 Falco 引擎版本，则会加载rule文件并返回错误。

##  3. 关键词
| Key                    | Required | Description                                                                                                                                                                                                                                                                      | Default |
|------------------------|----------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------|
| rule                   | yes      | rule的简短、唯一名称。                                                                                                                                                                                                                                         |         |
| condition              | yes      | 应用于事件以检查它们是否与rule匹配的过滤表达式。                                                                                                                                                                                      |         |
| desc                   | yes      | 对rule检测到的内容的更长描述。                                                                                                                                                                                                                                 |         |
| output                 | yes      | 指定发生匹配事件时应输出的消息                                                                                                                                                                                              |         |
| priority               | yes      | 事件严重性的不区分大小写的表示。应该是以下之一：emergency, alert, critical, error, warning, notice, informational, debug..                                                                                                        |         |
| exceptions             | no       | 一组导致rule不生成警报的异常。                                                                                                                                                                                                                |         |
| enabled                | no       | 如果设置为false，则既不加载rule，也不匹配任何事件。.                                                                                                                                                                                                        | true    |
| tags                   | no       | 应用于rule的标签列表                                                                                                                                                                                                                    |         |
| warn_evttypes          | no       | 如果设置为false，Falco 会抑制与没有事件类型的rule相关的警告                                                                                                                                                                      | true    |
| skip-if-unknown-filter | no       | 如果设置为true，如果rule条件包含过滤检查，例如fd.some_new_field，此版本的 Falco 不知道，Falco 会静默接受该rule但不执行它；如果设置为false，Falco 报告错误并在发现未知过滤检查时存在。 | false   |
| source                 | no       | 评估此rule的事件源。典型值为syscall,k8s_audit或源插件公布的源。                                                                                                                                     |         |

##  4. 语法



###  4.1 condition
rule的关键部分是条件字段。条件是使用[条件语法](https://falco.org/docs/rules/conditions/)表示的布尔谓词。可以使用它们各自[支持的字段](https://falco.org/docs/rules/supported-fields/)来表达所有[支持事件](https://falco.org/docs/rules/supported-events/)的条件。

这是一个在容器内运行 bash shell 时发出警报的条件示例：

```bash
container.id != host and proc.name = bash
```
第一个子句检查事件是否发生在容器中（其中`container.id`等于"`host`"事件是否发生在常规主机上）。第二个子句检查进程名称是否为`bash`. 由于此条件不包括带有系统调用的子句，它只会检查事件元数据。因此，如果 bash shell 确实在容器中启动，Falco 会为该 shell 执行的每个系统调用输出事件。

如果您只想在容器中每次成功生成 shell 时收到警报，请在条件中添加适当的事件类型和方向：

```bash
evt.type = execve and evt.dir=< and container.id != host and proc.name = bash
```
因此，使用上述条件的完整rule可能是：

```bash
- rule: shell_in_container
  desc: notice shell activity within a container
  condition: evt.type = execve and evt.dir=< and container.id != host and proc.name = bash
  output: shell in a container (user=%user.name container_id=%container.id container_name=%container.name shell=%proc.name parent=%proc.pname cmdline=%proc.cmdline)
  priority: WARNING
```
通常都以`evt.type`表达式或宏开头，安全rule通常一次只考虑一种系统调用类型。例如，您可能希望在打开文件时考虑open或openat捕获可疑活动、execve检查生成的进程等等。您不必猜测系统调用名称，因为您可以[查看支持的系统调用事件的完整列表](https://falco.org/docs/rules/supported-events/)并了解您可以使用哪些事件。

每个系统调用都有一个进入事件和一个退出事件，它们显示在`evt.dir`字段中，也称为系统调用的“方向”。值`>`表示入口，在调用系统调用时触发，而`<`标记退出，表示调用已返回。事实上，通过查看支持的系统调用列表，您可以看到我们的事件有两个条目。例如：

```bash
> setuid(UID uid)
< setuid(ERRNO res)
```
大多数情况下，引擎会通知您退出事件，因为您想了解事件执行完成后发生的情况。您可以通过使用与打开文件关联的事件来查看。

```bash
> open()
< open(FD fd, FSPATH name, FLAGS32 flags, UINT32 mode, UINT32 dev)
> openat()
< openat(FD fd, FD dirfd, FSRELPATH name, FLAGS32 flags, UINT32 mode, UINT32 dev)
```
每个事件都有一个与之关联的参数列表，您可以使用`evt.arg.<argname>`. 例如，要识别进程何时打开文件以覆盖它，您需要检查标志列表是否包含`O_TRUNC`. 您可以使用上面显示的和退出事件的`evt.arg.flags`字段，然后rule将如下所示：`open`、`openat`

```bash
evt.type in (open, openat) and evt.dir = < and evt.arg.flags contains O_TRUNC
```
请注意，参数不一定与 Linux 内核中使用的原始参数匹配，而是由 Falco 解析以更容易编写rule。通过使用这些[evt字段](https://falco.org/docs/rules/supported-fields/#field-class-evt)。

虽然`evt`字段允许您编写非常有表现力的条件，但参数和公共字段通常不足以编写完整的安全rule。很多时候，您希望根据事件发生的流程上下文添加条件，或者容器内是否正在发生某些事情，甚至将每个事件与集群​​、pod 等的相关 Kubernetes 元数据相关联。出于这个原因，Falco 用[其他字段类](https://falco.org/docs/rules/supported-fields/)丰富了许多事件。

### 4.2 rule operators
| =,!=                  | 等式和不等式运算符。                                                                                                                                 |
|-----------------------|--------------------------------------------------------------------------------------------------------------------------------------------|
| <=, <, >=,>           | 数值的比较运算符。                                                                                                                                  |
| contains,icontains    | 如果一个字符串包含另一个字符串，则对于字符串将评估为真，并且icontains是不区分大小写的版本。对于标志，如果设置了标志，它将评估为真。例子：proc.cmdline contains "-jar", evt.arg.flags contains O_TRUNC.     |
| startswith,endswith   | 检查字符串的前缀或后缀。                                                                                                                               |
| glob                  | 评估标准 glob 模式。示例：fd.name glob "/home/*/.ssh/*"。                                                                                             |
| in,intersects         | 设置操作。                                                                                                                                      |
| pmatch                | 将文件路径与一组文件或目录前缀进行比较。示例：fd.name pmatch (/tmp/hello)将针对 评估为真/tmp/hello，/tmp/hello/world但不是/tmp/hello_world。                                  |
| exists                | 检查是否设置了字段。示例：k8s.pod.name exists。                                                                                                          |
| bcontains,bstartswith | 这些运算符的工作方式与原始字节字符串类似，contains并且startswith允许对原始字节字符串执行字节匹配，接受十六进制字符串作为输入。例子：evt.buffer bcontains CAFEBABE, evt.buffer bstartswith 012AB3CC. |


###  4.3 rule macro
如上所述，宏提供了一种以可重用方式定义rule的公共子部分的方法。通过查看上面的条件，它看起来像两者`evt.type = execve and evt.dir=<`，并且`container.id != host`会被其他rule使用很多，所以为了使我们的工作更容易，我们可以轻松地为两者定义宏：

```bash
- macro: container
  condition: container.id != host


- macro: spawned_process
  condition: evt.type = execve and evt.dir=<

```
定义了这个宏后，我们可以将上述rule的条件重写为`spawned_process and container and proc.name = bash`

[falco 默认的 macro集合在这里！](https://blog.csdn.net/xixihahalelehehe/article/details/126183489)


###  4.4 rule list
列表是可以包含在rule、宏甚至其他列表中的项目的命名集合。请注意，列表不能被解析为过滤表达式。每个列表节点都有以下键：

| 钥匙    | 描述               |
|-------|------------------|
| list  | 列表的唯一名称（作为 slug） |
| items | 值列表              |

以下是一些示例列表以及使用它们的宏：

```bash
- list: shell_binaries
  items: [bash, csh, ksh, sh, tcsh, zsh, dash]

- list: userexec_binaries
  items: [sudo, su]

- list: known_binaries
  items: [shell_binaries, userexec_binaries]

- macro: safe_procs
  condition: proc.name in (known_binaries)
```
如果您使用多个 Falco rule文件，您可能希望将新项目附加到现有列表、rule或宏中。为此，请定义一个与现有项目同名的项目并将一个`append: true`属性添加到列表中。追加列表时，项目被添加到列表的末尾。附加rule/宏时，附加文本附加到条件：rule/宏的字段。

**请注意，在附加到列表、rule或宏时，rule配置文件的顺序很重要**！例如，如果您附加到现有的默认rule（例如Terminal shell in container），您必须确保您的自定义配置文件（`/etc/falco/rules.d/custom-rules.yaml`）在默认配置文件（`/etc/falco/falco_rules.yaml` ）之后加载。这可以使用多个参数以正确的顺序配置，直接在 falco 配置文件 (`falco.yaml` ) 中通过，或者您使用官方 Helm charts。

####  4.4.1 附加到列表
/etc/falco/falco_rules.yaml

```bash
- list: my_programs
  items: [ls, cat, pwd]

- rule: my_programs_opened_file
  desc: track whenever a set of programs opens a file
  condition: proc.name in (my_programs) and (evt.type=open or evt.type=openat)
  output: a tracked program opened a file (user=%user.name command=%proc.cmdline file=%fd.name)
  priority: INFO

```
/etc/falco/falco_rules.local.yaml

```bash
- list: my_programs
  append: true
  items: [cp]
```
每当打开文件时，该rule`my_programs_opened_file`都会触发。`ls`、`cat`、`pwd`、`cp`

####  4.4.2 附加到宏
/etc/falco/falco_rules.yaml

```bash
- macro: access_file
  condition: evt.type=open

- rule: program_accesses_file
  desc: track whenever a set of programs opens a file
  condition: proc.name in (cat, ls) and (access_file)
  output: a tracked program opened a file (user=%user.name command=%proc.cmdline file=%fd.name)
  priority: INFO

```
/etc/falco/falco_rules.local.yaml

```bash
- macro: access_file
  append: true
  condition: or evt.type=openat

```
该rule将在/使用/文件program_accesses_file时触发`ls`、`cat`、`open`或者`openat`

####  4.4.3 附加到rule
/etc/falco/falco_rules.yaml

```bash
- rule: program_accesses_file
  desc: track whenever a set of programs opens a file
  condition: proc.name in (cat, ls) and evt.type=open
  output: a tracked program opened a file (user=%user.name command=%proc.cmdline file=%fd.name)
  priority: INFO

```
/etc/falco/falco_rules.local.yaml

```bash
- rule: program_accesses_file
  append: true
  condition: and not user.name=root

```
该rule将在/用于文件`program_accesses_file`时触发，但如果用户是 root 则不会触发`ls`、`cat`、`open`





###  4.5 rule output
rule输出是一个字符串，它可以使用条件可以使用的相同字段`%`来执行插值，类似于`printf`. 例如：

```bash
Disallowed SSH Connection (command=%proc.cmdline connection=%fd.name user=%user.name user_loginuid=%user.loginuid container_id=%container.id image=%container.image.repository)
```
可以输出：

```bash
Disallowed SSH Connection (command=sshd connection=127.0.0.1:34705->10.0.0.120:22 user=root user_loginuid=-1 container_id=host image=<NA>)
```
请注意，不必在特定事件中设置所有字段。正如您在上面的示例中看到的，如果连接发生在容器外部，则`%container.image.repository`不会设置该字段，`<NA>`而是显示该字段。

### 4.6 rule priorities
每条 Falco rule都有一个优先级，表明违反rule的严重程度。优先级包含在消息/JSON 输出/等中。以下是可用的优先级：

 - EMERGENCY
 - ALERT
 - CRITICAL
 - ERROR
 - WARNING
 - NOTICE
 - INFORMATIONAL
 - DEBUG

用于为rule分配优先级的一般准则如下：

 - 如果rule与写入状态（即文件系统等）有关，则其优先级为ERROR.
 - 如果一条rule与未经授权的状态读取有关（即读取敏感文件等），则其优先级为WARNING.
 - 如果rule与意外行为（在容器中生成意外 shell、打开意外网络连接等）相关，则其优先级为NOTICE.
 - 如果rule与违反良好实践的行为（意外的特权容器、具有敏感挂载的容器、以 root 身份运行交互式命令）有关，

则其优先级为INFO.

###  4.7 rule tags
从 0.6.0 开始，rule具有一组可选标签，用于将rule集分类为相关rule组。这是一个例子：

```bash
 - rule: File Open by Privileged Container
  desc: Any open by a privileged container. Exceptions are made for known trusted images.
  condition: (open_read or open_write) and container and container.privileged=true and not trusted_containers
  output: File opened for read/write by privileged container (user=%user.name command=%proc.cmdline %container.info file=%fd.name)
  priority: WARNING
  tags: [container, cis]
```
标签的使用方法：
 - 您可以使用该`-T <tag>`参数禁用具有给定标签的rule。`-T`可以指定多次。例如，要跳过所有带有“`filesystem`”和“`cis`”标签的rule，你可以使用`falco  -T filesystem -T cis ....`, -T不能用 指定`-t`。
 - 您可以使用该-t <tag>参数仅运行具有给定标记的rule。`-t`可以指定多次。例如，要仅使用“`filesystem`”和“`cis`”标签运行这些rule，您可以使用`falco -t filesystem -t cis ....` ,-t不能用-Tor指定`-D <pattern>`（通过rule名称正则表达式禁用rule）

当前 Falco rule集的标签
我们还检查了默认rule集，并用一组初始标签标记了所有rule。以下是我们使用的标签：
| 标签            | 描述                            |
|---------------|-------------------------------|
| filesystem    | 该rule与读/写文件有关                   |
| software_mgmt | 该rule涉及任何软件/包管理工具，如 rpm、dpkg 等。 |
| process       | 该rule与启动新进程或更改当前进程的状态有关         |
| database      | 该rule与数据库有关                     |
| host          | 该rule仅适用于容器之外                   |
| shell         | 该rule特别涉及启动 shell               |
| container     | 该rule仅适用于容器内                    |
| cis           | 该rule与 CIS Docker 基准相关          |
| users         | 该rule涉及用户管理或更改正在运行的进程的身份        |
| network       | 该rule与网络活动有关                    |

###  4.8 fd.sip.name
域的实际查找是在单独的线程上完成的，以避免停止主系统调用事件循环。此外，域的 IP 集会定期刷新，策略如下：

 - 域名的基本刷新时间为 10 秒。
 - 如果在刷新周期后 IP 地址未更改，则该域名的刷新超时时间会加倍，直到 320 秒（约 5 分钟）

```bash
- list: https_miner_domains
  items: [
    "ca.minexmr.com",
    "cn.stratum.slushpool.com",
    "de.minexmr.com",
    "fr.minexmr.com",
    "mine.moneropool.com",
    "mine.xmrpool.net",
    "pool.minexmr.com",
    "sg.minexmr.com",
    "stratum-eth.antpool.com",
    "stratum-ltc.antpool.com",
    "stratum-zec.antpool.com",
    "stratum.antpool.com",
    "xmr.crypto-pool.fr"
  ]

# Add rule based on crypto mining IOCs
- macro: minerpool_https
  condition: (fd.sport="443" and fd.sip.name in (https_miner_domains))
```

```bash
- rule: Connect to Yahoo
  desc: Detect Connects to yahoo.com IPs
  condition: evt.type=connect and fd.sip.name=yahoo.com
  output: Some connect to yahoo (command=%proc.cmdline connection=%fd.name IP=%fd.sip.name)
  priority: INFO
```
相反，此rule将永远不会显示有意义的输出...IP=%fd.sip.name，因为比较使用否定匹配：

```bash
- rule: Connect to Anything but Yahoo
  desc: Detect Connects to anything other than yahoo.com IPs
  condition: evt.type=connect and fd.sip.name!=yahoo.com
  output: Some connect to something other than yahoo (command=%proc.cmdline connection=%fd.name IP=%fd.sip.name)
  priority: INFO

```
###  4.9 rule exceptions

```bash
- rule: Write below binary dir
  desc: an attempt to write to any file below a set of binary directories
  condition: >
    bin_dir and evt.dir = < and open_write
    and not package_mgmt_procs
    and not exe_running_docker_save
    and not python_running_get_pip
    and not python_running_ms_oms
    and not user_known_write_below_binary_dir_activities
...

```
以前，这些例外被表示为原始rule条件的串联。例如，查看宏 `package_mgmt_procs`：

```bash
- macro: package_mgmt_procs
  condition: proc.name in (package_mgmt_binaries)

```
结果附加`and not proc.name in (package_mgmt_binaries)`到rule的条件。

一个更极端的情况是 `Write_below_etc` 宏被 `Write below etc` rule使用。它有几十个例外：

```bash
...
    and not sed_temporary_file
    and not exe_running_docker_save
    and not ansible_running_python
    and not python_running_denyhosts
    and not fluentd_writing_conf_files
    and not user_known_write_etc_conditions
    and not run_by_centrify
    and not run_by_adclient
    and not qualys_writing_conf_files
    and not git_writing_nssdb
...
```
这些例外通常都遵循相同的结构——命名一个程序和 /etc 下允许该程序写入文件的目录前缀
从 `0.28.0` 开始，`falco` 支持`exceptions`rule的可选属性。例外键是标识符列表加上过滤检查字段的元组列表。这是一个例子：

```bash
- rule: Write below binary dir
  desc: an attempt to write to any file below a set of binary directories
  condition: >
    bin_dir and evt.dir = < and open_write
    and not package_mgmt_procs
    and not exe_running_docker_save
    and not python_running_get_pip
    and not python_running_ms_oms
    and not user_known_write_below_binary_dir_activities
  exceptions:
   - name: proc_writer
     fields: [proc.name, fd.directory]
     comps: [=, =]
     values:
       - [my-custom-yum, /usr/bin]
       - [my-custom-apt, /usr/local/bin]
   - name: cmdline_writer
     fields: [proc.cmdline, fd.directory]
     comps: [startswith, =]
   - name: container_writer
     fields: [container.image.repository, fd.directory]
   - name: proc_filenames
     fields: [proc.name, fd.name]
     comps: [=, in]
     values:
       - [my-custom-dpkg, [/usr/bin, /bin]]
   - name: filenames
     fields: fd.filename
     comps: in

```
该rule定义了四种例外情况：

 - `proc_writer`：使用 `proc.name` 和 `fd.directory` 的组合
 - `cmdline_writer`：使用 `proc.cmeline` 和 `fd.directory` 的组合
 - `container_writer`：使用 `container.image.repository` 和 `fd.directory` 的组合
 - `proc_filenames`：使用进程和文件名列表的组合。
 - 文件名：使用文件名列表

该`fields`属性包含一个或多个将从事件中提取值的字段。该`comps`属性包含将 1-1 与 fields 属性中的项目对齐的比较运算符。该`values`属性包含值的元组。元组中的每个项目都应与相应的字段和比较运算符 1-1 对齐。每个值元组都与字段/组合组合在一起，以修改条件以将排除项添加到rule的条件中。

例如，对于上面的异常“proc_writer”，字段/comps/values 相当于将以下内容添加到rule的条件中：

```bash
... and not ((proc.name=my-custom-yum and fd.directory=/usr/bin) or (proc.name=my-custom-apt and fd.directory=/usr/local/bin))
```
请注意，当比较运算符为“in”时，对应的值元组项应该是一个列表。上面的`“proc_filenames”`使用该语法，相当于：

```bash
... and not (proc.name=my-custom-dpkg and fd.name in (/usr/bin, /bin))
```
####  4.9.1 Appending Exception Values
异常值通常在rule中使用 append: true 定义。这是一个例子：

```bash
- list: apt_files
  items: [/bin/ls, /bin/rm]

- rule: Write below binary dir
  exceptions:
  - name: proc_writer
    values:
    - [apk, /usr/lib/alpine]
    - [npm, /usr/node/bin]
  - name: container_writer
    values:
    - [docker.io/alpine, /usr/libexec/alpine]
  - name: proc_filenames
    values:
    - [apt, [apt_files]]
    - [rpm, [/bin/cp, /bin/pwd]]
  - name: filenames
    values: [python, go]
  append: true

```
在这种情况下，这些值将附加到基本rule的任何值，然后将字段/comps/values 添加到rule的条件中。

综上所述，该rule的有效rule条件为：

```bash
... and not ((proc.name=my-custom-yum and fd.directory=/usr/bin) or                             # proc_writer
             (proc.name=my-custom-apt and fd.directory=/usr/local/bin) or
	     (proc.name=apk and fd.directory=/usr/lib/alpine) or
	     (proc.name=npm and fd.directory=/usr/node/bin) or
	     (container.image.repository=docker.io/alpine and fd.name=/usr/libexec/alpine) or   # container_writer
	     (proc.name=apt and fd.name in (apt_files)) or                                      # proc_filenames
	     (proc.name=rpm and fd.name in (/bin/cp, /bin/pwd)) or
	     (fd.filename in (python, go))                                                      # filenames
```
定义异常时，请尝试考虑`actor`、`action`和`target`，并尽可能将所有三个项目用于异常。例如，除了简单地使用`proc.name` or `container.image.repository`用于基于文件的异常，还包括通过`fd.name`、`fd.directory`等操作的文件。类似地，如果rule是特定于容器的，则不仅包括 image `container.image.repository`，还包括进程名称`proc.name`

####  4.9.2 Exceptions Syntax Shortcuts
rule可以定义字段和组合，但不能定义值。这允许稍后使用“append：true”的rule将值添加到异常中（更多内容见下文）。上 面的异常“cmdline_writer”具有以下格式：


```bash
   - name: cmdline_writer
     fields: [proc.cmdline, fd.directory]
     comps: [startswith, =]
```

定义异常的另一种方法是让字段包含单个字段，而 `comps` 包含单个比较运算符（必须是“`in`”、“`pmatch`”、“`intersects`”之一）。在这种格式中，`values` 是一个值列表，而不是元组列表。上面的异常“文件名”具有以下格式：

```bash
   - name: filenames
     fields: fd.filename
     comps: in
```
在这种情况下，异常相当于：

```bash
... and not (fd.filename in (...))
```
如果未提供 comps，则填写默认值。当 fields 为列表时，comps 将设置为 = 的等长列表。上面的异常“`container_writer`”具有该格式，相当于：

```bash
   - name: container_writer
     fields: [container.image.repository, fd.directory]
     comps: [=, =]
```

当 fields 是单个字段时，comps 设置为单个字段“in”。



##  5. 转义特殊字符 
在某些情况下，rule可能需要包含特殊字符，如(、空格等。例如，您可能需要查找 a `proc.name` of (systemd)，包括周围的括号。

您可以使用"这些特殊字符来捕获。这是一个例子：

```bash
- rule: Any Open Activity by Systemd
  desc: Detects all open events by systemd.
  condition: evt.type=open and proc.name="(systemd)" or proc.name=systemd
  output: "File opened by systemd (user=%user.name command=%proc.cmdline file=%fd.name)"
  priority: WARNING
```
在列表中包含项目时，请确保不会通过用单引号将带引号的字符串从 YAML 文件中解释双引号。这是一个例子：

```bash
- list: systemd_procs
  items: [systemd, '"(systemd)"']

- rule: Any Open Activity by Systemd
  desc: Detects all open events by systemd.
  condition: evt.type=open and proc.name in (systemd_procs)
  output: "File opened by systemd (user=%user.name command=%proc.cmdline file=%fd.name)"
  priority: WARNING
```
## 6. rule 文件
###  6.1 默认 rule 文件
默认的 falco rule文件安装在`/etc/falco/falco_rules.yaml`. 它包含一组预定义的rule，旨在在各种情况下提供良好的覆盖。目的是不修改此rule文件，并用每个新的软件版本替换。

###  6.2 本地 rule 文件
本地 falco rule文件安装在`/etc/falco/falco_rules.local.yaml`. 除了一些评论之外，它是空的。目的是将主rule文件的添加/覆盖/修改添加到此本地文件中。它不会被每个新的软件版本所取代。


##  7. rule 禁用默认
###  7.1 通过现有的宏
大多数默认rule都提供了某种`consider_*`宏，这些宏已经是rule条件的一部分。这些`consider_*`宏通常设置为(`never_true`)或(`always_true`)基本上启用或禁用相关rule。现在，如果您想启用默认禁用的rule（例如Unexpected outbound connection destination），您只需在自定义 Falco 配置中覆盖rule的`consider_*`宏（consider_all_outbound_conns在这种情况下）。

您的自定义 Falco 配置示例（注意(always_true)条件）：

```bash
- macro: consider_all_outbound_conns
  condition: (always_true)
```

> 请再次注意，指定配置文件的顺序很重要！最后定义的同名宏有效。

###  7.2 通过 Falco 参数
Falco 提供以下参数来限制应该启用/使用哪些默认rule，哪些不应该：

```bash
-D <substring>                Disable any rules with names having the substring <substring>. Can be specified multiple times.

-T <tag>                      Disable any rules with a tag=<tag>. Can be specified multiple times.
                               Can not be specified with -t.

-t <tag>                      Only run those rules with a tag=<tag>. Can be specified multiple times.
                               Can not be specified with -T/-D.
```
`extraArgs`如果您通过官方 `Helm charts` 部署 Falco，这些参数也可以指定为 Helm 图表值 ( ).

###  7.3 通过自定义rule定义
`enabled: false`最后但并非最不重要的一点是，您可以使用rule 属性禁用默认启用的rule。这对于在默认条件下不提供`consider_*`宏的rule特别有用。

确保在默认配置文件之后加载自定义配置文件。您可以使用多个-r参数配置正确的顺序，直接在 falco 配置文件中`falco.yaml`通过`rules_file`. 如果您使用的是官方 Helm charts，则使用该`falco.rules` File值配置顺序。

例如，在定义自定义rule时禁用User mgmt binaries默认rule：
`/etc/falco/falco_rules.yaml`   `/etc/falco/rules.d/custom-rules.yaml`

```bash
- rule: User mgmt binaries
  enabled: false
```
同时，可以使用`enabled: true` rule 属性重新启用禁用的rule。例如，默认情况下禁用的`Change thread namespace`rule`/etc/falco/falco_rules.yaml`可以通过以下方式手动启用：

```bash
- rule: Change thread namespace
  enabled: true

```

##  8. rule 陷阱
带有rule/宏附加和逻辑运算符的陷阱 
请记住，在附加rule和宏时，第二个rule/宏的文本只是简单地添加到第一个rule/宏的条件中。如果原始rule/宏具有潜在的模棱两可的逻辑运算符，这可能会导致意外结果。这是一个例子：

```bash
- rule: my_rule
  desc: ...
  condition: evt.type=open and proc.name=apache
  output: ...

- rule: my_rule
  append: true
  condition: or proc.name=nginx
```
应该`proc.name=nginx`解释为相对于`and proc.name=apache`，即允许 apache/nginx 打开文件，或者相对于`evt.type=open`，即允许 apache 打开文件或允许 nginx 做任何事情？

在这种情况下，请务必尽可能用括号限定原始条件的逻辑运算符范围，或者在不可能时避免附加条件。

参考：

 - [falco rules](https://falco.org/docs/rules/)

