# Linux Command tree 树状图

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b16450470a9be287540f20999079aad2.png)






##  1. 简介
Linux tree命令用于以树状图列出目录的内容。

执行tree指令，它会列出指定目录下的所有文件，包括子目录里的文件。

## 2. 语法

```bash
tree [-aACdDfFgilnNpqstux][-I <范本样式>][-P <范本样式>][目录...]
```
参数说明：


- -a 显示所有文件和目录。
- -A 使用ASNI绘图字符显示树状图而非以ASCII字符组合。
- -C 在文件和目录清单加上色彩，便于区分各种类型。
- -d 显示目录名称而非内容。
- -D 列出文件或目录的更改时间。
- -f 在每个文件或目录之前，显示完整的相对路径名称。
- -F 在执行文件，目录，Socket，符号连接，管道名称名称，各自加上"*","/","=","@","|"号。
- -g 列出文件或目录的所属群组名称，没有对应的名称时，则显示群组识别码。
- -i 不以阶梯状列出文件或目录名称。
- -L level 限制目录显示层级。
- -l 如遇到性质为符号连接的目录，直接列出该连接所指向的原始目录。
- -n 不在文件和目录清单加上色彩。
- -N 直接列出文件和目录名称，包括控制字符。
- -p 列出权限标示。
- -P<范本样式> 只显示符合范本样式的文件或目录名称。
- -q 用"?"号取代控制字符，列出文件和目录名称。
- -s 列出文件或目录大小。
- -t 用文件和目录的更改时间排序。
- -u 列出文件或目录的拥有者名称，没有对应的名称时，则显示用户识别码。
- -x 将范围局限在现行的文件系统中，若指定目录下的某些子目录，其存放于另一个文件系统上，则将该子目录予以排除在寻找范围外。

## 3. 实例
### 3.1 无参数执行

```bash
 tree
.
├── docker
│   └── volume
│       └── Dockerfile
├── iptables-scripts
└── wireguard
    ├── docker-compose
    │   ├── config
    │   │   ├── coredns
    │   │   │   └── Corefile
    │   │   ├── peer1
    │   │   │   ├── peer1.conf
    │   │   │   ├── peer1.png
    │   │   │   ├── presharedkey-peer1
    │   │   │   ├── privatekey-peer1
    │   │   │   └── publickey-peer1
    │   │   ├── server
    │   │   │   ├── privatekey-server
    │   │   │   └── publickey-server
    │   │   ├── templates
    │   │   │   ├── peer.conf
    │   │   │   └── server.conf
    │   │   └── wg0.conf
    │   └── docker-compose.yml
    └── private

9 directories, 15 files
```

### 3.2 只显示目录

```bash
tree -d
.
├── docker
│   └── volume
└── wireguard
    └── docker-compose
        └── config
            ├── coredns
            ├── peer1
            ├── server
            └── templates

9 directories
```

### 3.3 显示文件大小

```bash
$ tree -L 2 --du -h
.
├── [ 36K]  payments
│   ├── [2.4K]  deploy.sh
│   ├── [4.0K]  dist
│   ├── [4.0K]  lib
│   ├── [4.0K]  node_modules
│   ├── [1.5K]  package.json
│   ├── [ 356]  readme.md
│   ├── [4.0K]  src
│   ├── [4.0K]  test
│   └── [ 406]  tsconfig.json
│ ......
└── [ 30K]  transaction_engine
    ├── [2.4K]  deploy.sh
    ├── [4.0K]  dist
    ├── [4.0K]  lib
    ├── [4.0K]  node_modules
    ├── [ 570]  package.json
    ├── [ 658]  readme.md
    ├── [4.0K]  src
    └── [ 397]  tsconfig.json
```
设置别名

```bash
$ alias t="tree --du -h -L"
$ t 2
.
├── [ 36K]  payments
│   ├── [2.4K]  deploy.sh
│   ├── [4.0K]  dist
│   ├── [4.0K]  lib
│   ├── [4.0K]  node_modules
│   ├── [1.5K]  package.json
│   ├── [ 356]  readme.md
│   ├── [4.0K]  src
│   ├── [4.0K]  test
│   └── [ 406]  tsconfig.json
│ ......
└── [ 30K]  transaction_engine
    ├── [2.4K]  deploy.sh
    ├── [4.0K]  dist
    ├── [4.0K]  lib
    ├── [4.0K]  node_modules
    ├── [ 570]  package.json
    ├── [ 658]  readme.md
    ├── [4.0K]  src
    └── [ 397]  tsconfig.json
```
当然显示文件大小也可以通过du

```bash
$ du -ah --max-depth=2
7.5M    ./payments/dist
64K     ./payments/lib
74M     ./payments/node_modules
28K     ./payments/src
60K     ./payments/test
82M     ./payments
...
108K    ./transaction_engine/dist
44K     ./transaction_engine/lib
62M     ./transaction_engine/node_modules
16K     ./transaction_engine/src
0       ./transaction_engine/test
62M     ./transaction_engine
```

