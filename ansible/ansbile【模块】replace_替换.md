

----
## 1. 简介
replace 模块可以根据我们指定的正则表达式替换文件中的字符串，文件中所有被匹配到的字符串都会被替换。replace 模块和 lineinfile、blockinfile 两个模块不同，它并不是 Ansible 的核心模块，而是由社区维护。

##  2. 参数
path 参数：

 - 必须指定的参数。
 - 和 file 模块的 path 参数一样，指定要操作的文件。
 - 别名：dest, destfile, name。

regexp 参数：

 - 必须指定的参数。
 - 指定一个Python 的正则表达式，文件中与正则表达式匹配的字符串将会被替换。

replace 参数：

 - 替换regexp参数指定的正则表达式匹配的字符串，如果没有replace参数，则删除匹配的所有字符串。


after 参数：

 - 指定一个 Python 的正则表达式，仅该正则表达式之后的内容会被替换或者移除，可以和before参数一起使用。
 - 如果使用DOTALL表示特殊字符.可以匹配新行。

before 参数：

 - 指定一个 Python 的正则表达式，仅该正则表达式之前的内容会被替换或者移除，可以和after参数一起使用。
 - 如果使用DOTALL表示特殊字符.可以匹配新行。

encoding 参数：

 - 需要操作的文件的字符编码。
 - 默认值"utf-8"。

validate 参数：

 - 修改文件之前进行校验。
 - 使用“%s”表示path参数指定的需要修改的文件。

##  3. 示例
参考 [ansible replace](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/replace_module.html)

### 3.1 将/etc/hosts文件中的old.host.name修改为new.host.name：

```bash
- name: Before Ansible 2.3, option 'dest', 'destfile' or 'name' was used instead of 'path'
  replace:
    path: /etc/hosts
    regexp: '(\s+)old\.host\.name(\s+.*)?$'
    replace: '\1new.host.name\2'
```

### 3.2 注释 Apache 配置文件/etc/apache2/sites-available/default.conf中NameVirtualHost [*]行之后的所有内容：

```bash
- name: Replace after the expression till the end of the file (requires Ansible >= 2.4)
  replace:
    path: /etc/apache2/sites-available/default.conf
    after: 'NameVirtualHost [*]'
    regexp: '^(.+)$'
    replace: '# \1'
```

### 3.3 注释 Apache 配置文件/etc/apache2/sites-available/default.conf中# live site config行之前的所有内容：

```bash
- name: Replace before the expression till the begin of the file (requires Ansible >= 2.4)
  replace:
    path: /etc/apache2/sites-available/default.conf
    before: '# live site config'
    regexp: '^(.+)$'
    replace: '# \1'
```

### 3.4 注释 Apache 配置文件/etc/apache2/sites-available/default.conf中<VirtualHost [*]>行和</VirtualHost>行之间的内容：



```bash
- name: Replace between the expressions (requires Ansible >= 2.4)
  replace:
    path: /etc/apache2/sites-available/default.conf
    after: '<VirtualHost [*]>'
    before: '</VirtualHost>'
    regexp: '^(.+)$'
    replace: '# \1'
```

### 3.5 删除jdoe用户 SSH 的known_hosts文件中的old.host.name及之后的空行，同时修改文件属性和权限：

```bash
- name: Supports common file attributes
  replace:
    path: /home/jdoe/.ssh/known_hosts
    regexp: '^old\.host\.name[^\n]*\n'
    owner: jdoe
    group: jdoe
    mode: '0644'
```

### 3.6 修改/etc/apache/ports文件并校验：

```bash
- name: Supports a validate command
  replace:
    path: /etc/apache/ports
    regexp: '^(NameVirtualHost|Listen)\s+80\s*$'
    replace: '\1 127.0.0.1:8080'
    validate: '/usr/sbin/apache2ctl -f %s -t'
```

### 3.7 简短的格式：

```bash
- name: Short form task (in ansible 2+) necessitates backslash-escaped sequences
  replace: path=/etc/hosts regexp='\\b(localhost)(\\d*)\\b' replace='\\1\\2.localdomain\\2 \\1\\2'
```

### 3.8 多行的格式：

```bash
- name: Long form task does not
  replace:
    path: /etc/hosts
    regexp: '\b(localhost)(\d*)\b'
    replace: '\1\2.localdomain\2 \1\2'
```

### 3.9 明确指定替换中的位置匹配组：

```bash
- name: Explicitly specifying positional matched groups in replacement
  replace:
    path: /etc/ssh/sshd_config
    regexp: '^(ListenAddress[ ]+)[^\n]+$'
    replace: '\g<1>0.0.0.0'
```

### 3.9 明确指定命名匹配的组：

```bash
- name: Explicitly specifying named matched groups
  replace:
    path: /etc/ssh/sshd_config
    regexp: '^(?P<dctv>ListenAddress[ ]+)(?P<host>[^\n]+)$'
    replace: '#\g<dctv>\g<host>\n\g<dctv>0.0.0.0'
```

