#  process-exporter 容器安装部署
tags: exporters
<!-- catalog: ~process-exporter~ -->
![在这里插入图片描述](https://img-blog.csdnimg.cn/7ea9afc12d974088b12612d6ed03ea34.png)




## 1. 部署

```bash
docker run -tid --rm -p 9256:9256 --privileged -v /proc:/host/proc -v `pwd`:/config --name process-exporter ncabatoff/process-exporter --procfs /host/proc -config.path /config/process.yml
```

配置文件中监控僵尸进程为目标
```bash
 cat process.yml 
process_names:
  - name: "{{.Matches}}"
    cmdline:
    - 'zombie'
```
## 2. metrics说明

```bash
namedprocess_namegroup_states 查看僵尸
```



## 3. 配置说明
进程可能只属于一个组：即使多个项匹配，文件中列出的第一个项也会获胜。
（旁注：为避免与cmdline YAML元素混淆，我们将引用进程的命令行参数/proc/<pid>/cmdline作为数组 argv[]。）
使用配置文件：组名称
每个项目都process_names给出了识别和命名过程的方法。可选name标记定义了用于命名匹配进程的模板; 如果未指定，则name默认为{{.ExeBase}}。
可用的模板变量：

 - {{.Comm}} 包含原始可执行文件的基本名称，即第二个字段 /proc/<pid>/stat
 - {{.ExeBase}} 包含可执行文件的基名
 - {{.ExeFull}} 包含可执行文件的完全限定路径
 - {{.Username}} 包含有效用户的用户名
 - {{.Matches}} map包含应用cmdline regexps产生的所有匹配项

使用配置文件：进程选择器
每个项目process_names必须包含一个或多个选择器（comm，exe 或cmdline）; 如果存在多个选择器，则它们必须全部匹配。每个选择是一个字符串列表来匹配过程的comm，argv[0]或在的情况下cmdline，一个正则表达式应用到命令行。cmdline regexp使用Go语法。
对于comm和exe，字符串列表是OR，这意味着任何匹配任何字符串的进程都将添加到项目的组中。
因为cmdline，正则表达式列表是AND，意味着它们都必须匹配。正则表达式中的任何捕获组都必须使用该?P<name>选项为捕获分配名称，该名称用于填充.Matches。
性能提示：除了任何cmdline子句之外，还要提供exe或comm子句，以便在可执行文件名不匹配时避免执行regexp。



```bash

process_names:
  # comm is the second field of /proc/<pid>/stat minus parens.
  # It is the base executable name, truncated at 15 chars.
  # It cannot be modified by the program, unlike exe.
  - comm:
    - bash

  # exe is argv[0]. If no slashes, only basename of argv[0] need match.
  # If exe contains slashes, argv[0] must match exactly.
  - exe:
    - postgres
    - /usr/local/bin/prometheus

  # cmdline is a list of regexps applied to argv.
  # Each must match, and any captures are added to the .Matches map.
  - name: "{{.ExeFull}}:{{.Matches.Cfgfile}}"
    exe:
    - /usr/local/bin/process-exporter
    cmdline:
    - -config.path\s+(?P<Cfgfile>\S+)

```
## 4. prometheus接收metrics的配置

```bash
  - job_name: 'process_exporter'
    static_configs:
    - targets:
      - '192.168.1.88:9256'
    relabel_configs:
    - source_labels: [__address__]
      regex: (.+):(.+)
      target_label: __tmp_ip
      replacement: ${1}
    - source_labels: [__tmp_ip]
      regex: (.*)
      target_label: ip
      replacement: ${1}
```

## 5. 检查 并重启prometheus
检查并重启服务

```bash
$ promtool check config prometheus.yml 
$ curl -X POST http://localhost:9090/-/reloa
```

