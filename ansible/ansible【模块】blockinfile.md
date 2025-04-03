![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/80f3bbccf060b517c62292e0ecb2b895.png#pic_center)


----


---
## 1. 简介
blockinfile 是 Ansible 的一个非常实用的模块，和单行替换模块 lineinfile 类似，但是可以帮助我们在文件中插入一段文本。

## 2. 常用参数

 - `path` 参数：
必须指定的参数。
和 file 模块的 path 参数一样，指定要操作的文件。
别名：dest, destfile, name。

 - `state` 参数：
确保段落存在（state=present）或者不存在（state=absent）。
默认值为`state=present`，会将指定的一段文本插入到文件中，如果文件中已经存在标记的文本，默认会更新对应段落。
如果`state=absent`，则表示从文件中删除对应标记的段落。

 - `marker` 参数：
默认值`"# {mark} ANSIBLE MANAGED BLOCK"`。
我们想要在指定文件中插入一段文本，Ansible 会自动为这段文本添加两个标记，一个开始标记，一个结束标记，默认情况下，开始标记为`# BEGIN ANSIBLE MANAGED BLOCK`，结束标记为`# END ANSIBLE MANAGED BLOCK`。
我们可以使用marker参数自定义标记，比如`marker = #{mark}test`，这样开始标记变成了`# BEGIN test`，结束标记变成了`# END test`。
`{mark}`变量会自动被替换成开始标记中的marker_begin和结束标记中的marker_end，如果使用没有{mark}变量的自定义标记，可能会导致重复插入。

 - `marker_begin` 参数：
设置marker参数的开始标记中的{mark}变量。
默认值"BEGIN"。

 - `marker_end` 参数：
设置marker参数的结束标记中的{mark}变量。
默认值"END"。

 - `block` 参数：
指定一段需要操作的文本。
如果没有block参数或者参数的值为空，则移除文本块，效果等同于`state = absent`。
别名：content。

 - `insertafter` 参数：
插入段落（state=present）时使用。
值为EOF或者正则表达式，默认值为EOF，表示`End Of File`，插入到文件的末尾。
如果设置为正则表达式，默认将文本插入到正则表达式匹配的最后一行之后。
如果设置为正则表达式，但是没有匹配到任何行，则插入到文件末尾。

 - `insertbefore` 参数：
插入段落（`state=present`）时使用。
值为`BOF`或者正则表达式，默认值为BOF，表示`Begin Of File`，插入到文件的开头。
如果设置为正则表达式，默认将文本插入到正则表达式匹配的最后一行之前。
如果设置为正则表达式，但是没有匹配到任何行，则插入到文件开头。

 - `validate` 参数：
修改文件之前进行校验。
使用`“%s”`表示path参数指定的需要修改的文件。

 - `file`模块的所有参数，常用的有：
`backup`：是否在修改文件之前对文件进行备份。
`create` ：yes：当要操作的文件不存在时，创建对应的文件；默认为no：当要操作的文件不存在时，语句报错。
`mode`：文件的属性。
`owner`：文件的属主。
`group`：文件的属组。


## 3. 示例
### 3.1 修改 SSHD 配置文件禁止ansible-agent用户使用密码登录：

```bash
- name: Insert/Update "Match User" configuration block in /etc/ssh/sshd_config
  blockinfile:
    path: /etc/ssh/sshd_config
    block: |
      Match User ansible-agent
      PasswordAuthentication no
```
### 3.2 在 Debian/Ubuntu 网络配置文件/etc/network/interfaces中添加网卡 eth0 的配置信息：

```bash
- name: Insert/Update eth0 configuration stanza in /etc/network/interfaces
        (it might be better to copy files into /etc/network/interfaces.d/)
  blockinfile:
    path: /etc/network/interfaces
    block: |
      iface eth0 inet static
          address 192.0.2.23
          netmask 255.255.255.0
```

### 3.3 备份/etc/ssh/ssh_config文件，并在文件末尾插入./local/ssh_config文件的内容，最后使用/usr/sbin/sshd -T -f /etc/ssh/ssh_config命令校验：

```bash
- name: Insert/Update configuration using a local file and validate it
  blockinfile:
    block: "{{ lookup('file', './local/ssh_config') }}"
    dest: /etc/ssh/ssh_config
    backup: yes
    validate: /usr/sbin/sshd -T -f %s
```
### 3.4 在 HTML 文件/var/www/html/index.html的"<body>"之后插入自定义标记的内容：

```bash
- name: Insert/Update HTML surrounded by custom markers after <body> line
  blockinfile:
    path: /var/www/html/index.html
    marker: "<!-- {mark} ANSIBLE MANAGED BLOCK -->"
    insertafter: "<body>"
    block: |
      <h1>Welcome to {{ ansible_hostname }}</h1>
      <p>Last updated on {{ ansible_date_time.iso8601 }}</p>
```

### 3.5 移除在 HTML 文件/var/www/html/index.html中的添加的内容：

```bash
- name: Remove HTML as well as surrounding markers
  blockinfile:
    path: /var/www/html/index.html
    marker: "<!-- {mark} ANSIBLE MANAGED BLOCK -->"
    block: ""
```
### 3.6 在/etc/hosts文件中添加解析记录：

```bash
- name: Add mappings to /etc/hosts
  blockinfile:
    path: /etc/hosts
    block: |
      {{ item.ip }} {{ item.name }}
    marker: "# {mark} ANSIBLE MANAGED BLOCK {{ item.name }}"
  with_items:
  - { name: host1, ip: 10.10.1.10 }
  - { name: host2, ip: 10.10.1.11 }
  - { name: host3, ip: 10.10.1.12 }
```

