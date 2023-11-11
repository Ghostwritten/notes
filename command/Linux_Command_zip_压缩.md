#  Linux Command zip 压缩
![](https://img-blog.csdnimg.cn/0f5e605ef19b48e294135941117e3fa3.gif#pic_center)



##  1. 安装

```bash
$ yum -y install zip
```
## 2. 格式
```c
zip [-AcdDfFghjJKlLmoqrSTuvVwXyz$][-b <工 作目录>][-ll][-n <字 尾字符串>][-t <日 期时间>][-<压 缩效率>][压 缩文件][文件...][-i <范本样式>][-x <范本样式>]
```

补充说明：zip是个使用广泛的压缩程序，文件经它压缩后会另外产生具 有".zip"扩展名 的压缩文件。
## 3. 参数

```c
-A   调 整可执行的自动解压缩文件。
-b   <工作目录>   指 定暂时存放文件的目录。
-c   替 每个被压缩的文件加上注释。
-d   从 压缩文件内删除指定的文件。
-D   压 缩文件内不建立目录名称。
-f   此 参数的效果和指定"-u"参 数类似，但不仅更新既有文件，如果某些文件原本不存在于压缩文件内，使用本参数会一并将其加入压缩文件中。
-F   尝 试修复已损坏的压缩文件。
-g   将 文件压缩后附加在既有的压缩文件之后，而非另行建立新的压缩文件。
-h   在 线帮助。
-i<范本样式>   只 压缩符合条件的文件。
-j   只 保存文件名称及其内容，而不存放任何目录名称。
-J   删 除压缩文件前面不必要的数据。
-k   使 用MS-DOS兼容格 式的文件名称。
-l   压 缩文件时，把LF字符 置换成LF+CR字 符。
-ll   压 缩文件时，把LF+CR字 符置换成LF字符。
-L   显 示版权信息。
-m   将 文件压缩并加入压缩文件后，删除原始文件，即把文件移到压缩文件中。
-n<字尾字符串>   不 压缩具有特定字尾字符串的文件。
-o   以 压缩文件内拥有最新更改时间的文件为准，将压缩文件的更改时间设成和该文件相同。
-q   不显 示指令执行过程。
-r   递 归处理，将指定目录下的所有文件和子目录一并处理。
-S   包 含系统和隐藏文件。
-t<日期时间>   把 压缩文件的日期设成指定的日期。
-T   检 查备份文件内的每个文件是否正确无误。
-u   更 换较新的文件到压缩文件内。
-v   显 示指令执行过程或显示版本信息。
-V   保 存VMS操作系统的文 件属性。
-w   在 文件名称里假如版本编号，本参数仅在VMS操 作系统下有效。
-x<范本样式>   压 缩时排除符合条件的文件。
-X   不 保存额外的文件属性。
-y   直 接保存符号连接，而非该连接所指向的文件，本参数仅在UNIX之 类的系统下有效。
-z   替 压缩文件加上注释。
-$   保 存第一个被压缩文件所在磁盘的卷册名称。
-<压缩效率>   压 缩效率是一个介于1-9的 数值。
```
## 4. 举例
-d -g -u
```bash
[root@localhost dir1]$ ls
1.txt  2.txt  3.txt  4.txt  5.txt  6.txt
[root@localhost dir1]$ zip new.zip *.txt  //压缩多个文件
  adding: 1.txt (stored 0%)
  adding: 2.txt (stored 0%)
  adding: 3.txt (stored 0%)
  adding: 4.txt (stored 0%)
  adding: 5.txt (stored 0%)
  adding: 6.txt (stored 0%)
[root@localhost dir1]$ ls
1.txt  2.txt  3.txt  4.txt  5.txt  6.txt  new.zip
 $ rm -rf *.txt
[root@localhost dir1]$ zip -d new.zip 1.txt     //删除new.zip包中1.txt文件
deleting: 1.txt
[root@localhost dir1]$ unzip new.zip 
Archive:  new.zip
 extracting: 2.txt                   
 extracting: 3.txt                   
 extracting: 4.txt                   
 extracting: 5.txt                   
 extracting: 6.txt                   
[root@localhost dir1]$ ls
2.txt  3.txt  4.txt  5.txt  6.txt  new.zip
[root@localhost dir1]$ touch 1.txt
[root@localhost dir1]$ zip -g new.zip 1.txt     //将1.txt添加到new.zip包中
  adding: 1.txt (stored 0%)
[root@localhost dir1]$ vim 1.txt   //随意输入一些内容
sss test
[root@localhost dir1]$ touch 7.txt //添加一个文件
[root@localhost dir1]$ zip -u new.zip 1.txt 
updating: 1.txt (stored 0%)
[root@localhost dir1]$ zip -u new.zip 1.txt //查看1.txt文件是否有更改
updating: 1.txt (stored 0%)
[root@localhost dir1]$ zip -u new.zip 1.txt //再次执行无效
[root@localhost dir1]$ zip -u new.zip * //查看当前目录下与new.zip中所有文件是否有更改或添加
  adding: 7.txt (stored 0%)
[root@localhost dir1]$ zip -u new.zip * //再次执行无效
[root@localhost dir1]$ zip -u new.zip 2.txt //无改动
```

1.将 /home/html/ 这个目录下所有文件和文件夹打包为当前目录下的 html.zip：
```bash
zip -q -r html.zip /home/html
```
2.在 /home/html 目录下，可以执行以下命令：
```bash
zip -q -r html.zip *
```
3.从压缩文件 cp.zip 中删除文件 a.c

```bash
zip -dv cp.zip a.c
```

更多阅读：
 1. [Linux Command lz4 压缩](https://blog.csdn.net/xixihahalelehehe/article/details/125201619)
 2. [Linux Command tar 压缩](https://ghostwritten.blog.csdn.net/article/details/106056841)
 3. [Linux Command gzip 压缩](https://blog.csdn.net/xixihahalelehehe/article/details/104724596)
 4. [Linux Command zip 压缩](https://blog.csdn.net/xixihahalelehehe/article/details/125203106)

![](https://img-blog.csdnimg.cn/7e96eec11002402d90ea6a40320b9b1d.png)




