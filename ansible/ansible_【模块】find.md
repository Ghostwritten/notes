

----
## 1. 介绍
find 模块可以帮助我们在被管理主机中查找符合条件的文件，就像 find 命令一样。
## 2. 参数

 - `paths` ：必须参数，指定在哪个目录中查找文件，可以指定多个路径，路径间用逗号隔开，此参数有别名，使用别名 path 或者别名 name 可以代替 paths。
 - `recurse` : 默认情况下，只会在指定的目录中查找文件，也就是说，如果目录中还包含目录，ansible并不会递归的进入子目录查找对应文件，如果想要递归的查找文件，需要使用 recurse 参数，当 recurse 参数设置为 yes时，表示在指定目录中递归的查找文件。
 - `hidden`：默认情况下，隐藏文件会被忽略，当 hidden 参数的值设置为 yes 时，才会查找隐藏文件。
 - `file_type` : 默认情况下，ansible只会根据条件查找”文件”，并不会查找”目录”或”软链接”等文件类型，如果想要指定查找的文件类型，可以通过 file_type指定文件类型，可指定的文件类型有 any、directory、file、link 四种。
 - `patterns` ： 使用此参数指定需要查找的文件名称，支持使用shell（比如通配符）或者正则表达式去匹配文件名称，默认情况下，使用 shell 匹配对应的文件名，如果想要使用 python的正则去匹配文件名，需要将 use_regex 参数的值设置为 yes。
 - `use_regex`：默认情况下，find 模块不会使用正则表达式去解析 patterns 参数中对应的内容，当 use_regex 设置为yes 时，表示使用 python 正则解析 patterns 参数中的表达式，否则，使用 glob 通配符解析 patterns参数中的表达式。
 - `contains`：使用此参数可以根据文章内容查找文件，此参数的值为一个正则表达式，find 模块会根据对应的正则表达式匹配文件内容。
 - `age` ：使用此参数可以根据时间范围查找文件，默认以文件的 mtime 为准与指定的时间进行对比，比如，如果想要查找 mtime在3天之前的文件，那么可以设置 age=3d，如果想要查找 mtime 在3天以内的文件，可以设置 age=-3d，这里所说的3天是按照当前时间往前推3天，可以使用的单位有秒(s)、分(m)、时(h)、天(d)、星期(w)。
 - `age_stamp`：文件的时间属性中有三个时间种类，atime、ctime、mtime，当我们根据时间范围查找文件时，可以指定以哪个时间种类为准，当根据时间查找文件时，默认以
   mtime 为准。
 - `size`：使用此参数可以根据文件大小查找文件，比如，如果想要查找大于3M的文件，那么可以设置size=3m,如果想要查找小于50k的文件，可以设置 size=-50k，可以使用的单位有 t、g、m、k、b。
 - `get_checksum` ：当有符合查找条件的文件被找到时，会同时返回对应文件的
   sha1校验码，如果要查找的文件比较大，那么生成校验码的时间会比较长

## 3. 命令
在 ansible-demo3 主机的 /testdir目录中查找文件内容中包含 abc 字符串的文件，隐藏文件会被忽略，不会进行递归查找。

```bash
ansible ansible-demo3 -m find -a 'paths=/testdir contains=".*abc.*" '
```
在 ansible-demo3 主机的 /testdir 目录以及其子目录中查找文件内容中包含 abc 字符串的文件，隐藏文件会被忽略。

```bash
ansible ansible-demo3 -m find -a 'paths=/testdir contains=".*abc.*" recurse=yes '
```

在 ansible-demo3 主机的 /testdir 目录中查找以 .sh 结尾的文件，包括隐藏文件，但是不包括目录或其他文件类型，不会进行递归查找。

```bash
ansible ansible-demo3 -m find -a 'paths=/testdir patterns="*.sh" hidden=yes'
```

在 ansible-demo3 主机的 /testdir 目录中查找以 .sh 结尾的文件，包括隐藏文件，包括所有文件类型，比如文件、目录、或者软链接，但是不会进行递归查找。

```bash
ansible ansible-demo3 -m find -a 'paths=/testdir patterns="*.sh" file_type=any hidden=yes'
```

在 ansible-demo3 主机的 /testdir 目录中查找以 .sh 结尾的文件，只不过 patterns 对应的表达式为正则表达式，查找范围包括隐藏文件，包括所有文件类型，但是不会进行递归查找，不会对 /testdir 目录的子目录进行查找。

```bash
ansible ansible-demo3 -m find -a 'paths=/testdir patterns=".*\.sh" use_regex=yes file_type=any hidden=yes'
```

在 ansible-demo3 主机的 /testdir 目录中以及其子目录中查找 mtime 在1天以内的文件，不包含隐藏文件，不包含目录或软链接文件等文件类型。

```bash
ansible ansible-demo3 -m find -a "path=/testdir age=-1d recurse=yes"
```

在 ansible-demo3 主机的 /testdir 目录中以及其子目录中查找大于 2g 的文件，不包含隐藏文件，不包含目录或软链接文件等文件类型。

```bash
ansible ansible-demo3 -m find -a "paths=/testdir size=2g recurse=yes"
```

在 ansible-demo3 主机的 /testdir 目录中以及其子目录中查找以 .sh 结尾的文件，并且返回符合条件文件的 sha1 校验码，包括隐藏文件。

```bash
ansible ansible-demo3 -m find -a "paths=/testdir patterns=*.sh get_checksum=yes hidden=yes recurse=yes"
```
## 4. yaml

```bash
- name: Recursively find /tmp files older than 2 days
  find:
    paths: /tmp
    age: 2d
    recurse: yes

- name: Recursively find /tmp files older than 4 weeks and equal or greater than 1 megabyte
  find:
    paths: /tmp
    age: 4w
    size: 1m
    recurse: yes

- name: Recursively find /var/tmp files with last access time greater than 3600 seconds
  find:
    paths: /var/tmp
    age: 3600
    age_stamp: atime
    recurse: yes

- name: Find /var/log files equal or greater than 10 megabytes ending with .old or .log.gz
  find:
    paths: /var/log
    patterns: '*.old,*.log.gz'
    size: 10m

# Note that YAML double quotes require escaping backslashes but yaml single quotes do not.
- name: Find /var/log files equal or greater than 10 megabytes ending with .old or .log.gz via regex
  find:
    paths: /var/log
    patterns: "^.*?\\.(?:old|log\\.gz)$"
    size: 10m
    use_regex: yes

- name: Find /var/log all directories, exclude nginx and mysql
  find:
    paths: /var/log
    recurse: no
    file_type: directory
    excludes: 'nginx,mysql'

# When using patterns that contain a comma, make sure they are formatted as lists to avoid splitting the pattern
- name: Use a single pattern that contains a comma formatted as a list
  find:
    paths: /var/log
    file_type: file
    use_regex: yes
    patterns: ['^_[0-9]{2,4}_.*.log$']

- name: Use multiple patterns that contain a comma formatted as a YAML list
  find:
    paths: /var/log
    file_type: file
    use_regex: yes
    patterns:
      - '^_[0-9]{2,4}_.*.log$'
      - '^[a-z]{1,5}_.*log$'
```

