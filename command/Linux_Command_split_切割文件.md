#  Linux Command split 切割文件



---
## 1. 简介
split命令可以将一个大文件分割成很多个小文件，有时需要将文件分割成更小的片段，比如为提高可读性，生成日志等。

## 2. 参数

```c
-a, --suffix-length=N   use suffixes of length N (default 2)   #输出文件后缀长度，默认为：2
-b, --bytes=SIZE   put SIZE bytes per output file       #按照文件大小分割文件，单位：字节
一个字节 = 8位二进制数（1byte == 8bit）

ASCII码：一个英文字母（不分大小写）占一个字节的空间，一个中文汉字占两个字节的空间
UTF-8编码：一个英文字符等于一个字节，一个中文（含繁体，及中文标点）等于三个字节
Unicode编码：一个英文（含标点）等于两个字节，一个中文（含繁体，及中文标点）等于两个字节

-d, --numeric-suffixes  use numeric suffixes instead of alphabetic  #添加数字后缀（因为默认添加的是字母后缀，所有要想加数字需要自己添加）
-l, --lines=NUMBER      put NUMBER lines per output file   #按照行数分割文件，默认1000行一个文件
--verbose  print a diagnostic just before each output file is opened  #打印运行状态信息
--help     display this help and exit          查看说明文档
--version  output version information and exit 查看版本信息
```

## 3. 举例
###  3.2 默认切割
默认情况下，split 命令使用非常简单的命名方案。文件块将被命名为 `xaa、xab、xac` 等，并且，大概地，如果你将足够大的文件分割，你甚至可能会得到名为 xza 和 xzz 的块。
除非你要求，否则该命令将无任何反馈地运行。
```bash
$ split bigfile
```

### 3.2 `--verbose` 创建文件块时看到反馈

```bash
$ split –-verbose bigfile
creating file 'xaa'
creating file 'xab'
creating file 'xac'
```

### 3.3 原始文件分割并命名为 bigfile.aa、bigfile.ab 

```bash
$ split –-verbose bigfile bigfile.
creating file 'bigfile.aa'
creating file 'bigfile.ab'
creating file 'bigfile.ac'
```

**请注意，上述命令中显示的前缀的末尾会添加一个点。否则，文件将是 bigfileaa 之类的名称，而不是 bigfile.aa。**
请注意，split 命令不会删除你的原始文件，只是创建了文件块。

### 3.4  -b 指定文件块的大小

```bash
$ split -b100M bigfile
```

文件大小可以是 KB、MB，GB，最大可以是 YB！只需使 K、M、G、T、P、E、Z 和 Y 这些合适的字母。
### 3.5 -l 根据文件行数来分割文件
每个文件将有 1000 行

```bash
$ split --verbose -l1000 logfile log.
creating file 'log.aa'
creating file 'log.ab'
creating file 'log.ac'
creating file 'log.ad'
creating file 'log.ae'
creating file 'log.af'
creating file 'log.ag'
creating file 'log.ah'
creating file 'log.ai'
creating file 'log.aj'
```
### 3.6 cat 合并与原文件无差别
```bash
$ split --verbose -b50K zip zip.
creating file 'zip.aa'
creating file 'zip.ab'
creating file 'zip.ac'
creating file 'zip.ad'
creating file 'zip.ae'
$ cat zip.a? > zip.new
$ diff zip zip.new
$                    <== 无输出 = 无差别
```


### 3.7 -d 用数字后缀
同时可以使用-a length来指定后缀的长度：

```bash
$ split -b 10k date.file -d -a 3
$ ls
date.file  x000  x001  x002  x003  x004  x005  x006  x007  x008  x009
```

为分割后的文件指定文件名的前缀：

```bash
$ split -b 10k date.file -d -a 3 split_file
$ ls
date.file  split_file000  split_file001  split_file002  split_file003  split_file004  split_file005  split_file006  split_file007  split_file008  split_file009
```

更多阅读：

 - [Split Command in Linux with Examples](https://www.geeksforgeeks.org/split-command-in-linux-with-examples/)
 - [split(1) — Linux manual page](https://man7.org/linux/man-pages/man1/split.1.html)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/baf4cd57f4aa9e853a8408cb59b259e2.gif#pic_center)



