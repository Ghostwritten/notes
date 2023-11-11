#  python 模块 filecmp 文件差异对比



-----

> 环境：Python3.6，Windwos10 RS1
> Pycharm 参考书籍—《Python自动化运维--技术与最佳实践》刘天斯著

## 1.介绍

Python2.3以上的版本默认自带了filecmp模块，无需额外安装。我们可以用这个模块来检查原式与目标文件的一致性，filecmp可以实现文件、目录、遍历子目录的差异对比功能。

模块使用
filecmp提供了三个操作方法。

 - cmp：单文件对比
 - cmpfiles：多文件对比
 - dircmp：目录对比

## 2. 单文件对比

使用的是filecmp.cmp(f1,f2[,shallow])。其中f1、f2为文件，当两个文件相同时返回True，不同返回False，(shallow默认为True，其意思是只根据os.stat()方法返回的文件基本信息进行对比[最后访问时间、修改时间、状态改变时间等，而不考虑文件内容])。当shallow为False时，则os.stat()与文件内容同时进行校验。
现在我在我代码的当前路径下有三个分别名为File1.txt、File2.txt、File3.txt的文件，File1.txt和File3.txt两个文件是单独创建的文本文件，而File2.txt是File1.txt的副本文件，即直接由复制文件File1.txt并改名File2.txt得到。所以按理来说File1.txt、File2.txt两个文件完全相同、而与File3.txt文件不同。
下面使用代码进行验证：

```python
Python 3.6.2 (v3.6.2:5fd33b5, Jul  8 2017, 04:14:34) [MSC v.1900 32 bit (Intel)] on win32
>>> import filecmp 
>>> filecmp.cmp('./File1.txt', './File2.txt') True # 说明两个文件相同，返回True 
>>>> filecmp.cmp('./File1.txt', './File3.txt') False # 说明两个文件不同，返回False
```

## 3. 多文件对比

比较两个文件夹内指定文件是否相等。参数dir1, dir2指定要比较的文件夹，参数common指定要比较的文件名列表。函数返回包含3个list元素的元组，分别表示匹配、不匹配以及错误的文件列表。错误的文件指的是不存在的文件，或文件被琐定不可读，或没权限读文件，或者由于其他原因访问不了该文件。

下列文件是dir1文件夹下所有文件的MD5值

```powershell
$ md5sum dir1/*
FBAAF8CC3C1DF70B183CB8E8FBDFC57C *File1.txt
FBAAF8CC3C1DF70B183CB8E8FBDFC57C *File2.txt
16EC37C499F64FC60E95650B500E30A4 *File3.txt
202CB962AC59075B964B07152D234B70 *File4.txt
```

下列文件是dir1文件夹下所有文件的MD5值

```powershell
$ md5sum dir2/*
FBAAF8CC3C1DF70B183CB8E8FBDFC57C *File1.txt
FBAAF8CC3C1DF70B183CB8E8FBDFC57C *File2.txt
D41D8CD98F00B204E9800998ECF8427E *File3.txt
CB30FC9CEC9A2D04EF49B22E2066C264 *File5.txt
```

根据MD5值可以知道，两个文件夹下的File1.txt、File2.txt相匹配，

而File3.txt不匹配，另外对于File4.txt和File5.txt两个文件只在各自的目录下存在。

使用示例代码验证：

```python
>>> filecmp.cmpfiles('./dir1', './dir2', ['File1.txt', 'File2.txt', 'File3.txt', 'File4.txt', 'File5.txt'])
(['File1.txt', 'File2.txt'], ['File3.txt'], ['File4.txt', 'File5.txt'])       # 返回为列表

列表[0] == 匹配
列表[1] == 不匹配
列表[2] == 不存在，无法比较
```

## 4. 目录对比

目录对比，通过filecmp（a,b[,ignore[,hide]]）类创建一个目录比较对象用于比较文件夹，通过该类比较两个文件夹，可以获取一些详细的比较结果（如只在A文件夹存在的文件列表），并支持子文件夹的递归比较。

dircmp提供了三个方法用于报告比较的结果：

```markup
report()：只比较指定文件夹中的内容（文件与文件夹）
report_partial_closure()：比较文件夹及第一级子文件夹的内容
report_full_closure()：递归比较所有的文件夹的内容
另外为了输出更加详细的信息，dircmp类还提供了以下属性：

left_list：左边文件夹中的文件与文件夹列表；
right_list：右边文件夹中的文件与文件夹列表；
common：两边文件夹中都存在的文件或文件夹；
left_only：只在左边文件夹中存在的文件或文件夹；
right_only：只在右边文件夹中存在的文件或文件夹；
common_dirs：两边文件夹都存在的子文件夹；
common_files：两边文件夹都存在的子文件；
common_funny：两边文件夹都存在的子文件夹；
same_files：匹配的文件；
diff_files：不匹配的文件；
funny_files：两边文件夹中都存在，但无法比较的文件；
subdirs：将common_dirs 目录映射到新的dircmp对象，格式为字典的类型。
```

示例代码，比较dir1和dir2文件夹目录的差异

```python
-*- coding: utf-8 -*

import filecmp

a = "./dir1"  # 定义左目录
b = "./dir2"  # 定义右目录

dirobj = filecmp.dircmp(a, b, ['EXT.txt'])  # 目录比较，忽略EXT.txt文件

输出对比结果数据报表，详细说明请参考filecmp类方法及属性信息


dirobj.report()
dirobj.report_partial_closure()
dirobj.report_full_closure()
print("left_list:" + str(dirobj.left_list))
print("right_list:" + str(dirobj.right_list))
print("common:" + str(dirobj.common))
print("left_only:" + str(dirobj.left_only))
print("right_only:" + str(dirobj.right_only))
print("common_dirs:" + str(dirobj.common_dirs))
print("common_files:" + str(dirobj.common_files))
print("common_funny:" + str(dirobj.common_funny))
print("same_file:" + str(dirobj.same_files))
print("diff_files:" + str(dirobj.diff_files))
print("funny_files:" + str(dirobj.funny_files))
```

运行结果下：

```powershell
C:\Users\imwoo\AppData\Local\Programs\Python\Python36-32\python.exe E:/Python_Test/Code/Auto_Operations/filecmpTest.py
diff ./dir1 ./dir2
Only in ./dir1 : ['File4.txt']
Only in ./dir2 : ['File5.txt']
Identical files : ['File1.txt', 'File2.txt']
Differing files : ['File3.txt']
diff ./dir1 ./dir2
Only in ./dir1 : ['File4.txt']
Only in ./dir2 : ['File5.txt']
Identical files : ['File1.txt', 'File2.txt']
Differing files : ['File3.txt']
diff ./dir1 ./dir2
Only in ./dir1 : ['File4.txt']
Only in ./dir2 : ['File5.txt']
Identical files : ['File1.txt', 'File2.txt']
Differing files : ['File3.txt']
left_list:['File1.txt', 'File2.txt', 'File3.txt', 'File4.txt']
right_list:['File1.txt', 'File2.txt', 'File3.txt', 'File5.txt']
common:['File1.txt', 'File2.txt', 'File3.txt']
left_only:['File4.txt']
right_only:['File5.txt']
common_dirs:[]
common_files:['File1.txt', 'File2.txt', 'File3.txt']
common_funny:[]
same_file:['File1.txt', 'File2.txt']
diff_files:['File3.txt']
funny_files:[]
```

其实个人觉得上面的信息不太方便理解，所以附上我的一张目录tree图：

```bash
$ tree
├─dir1
│      EXT.txt
│      File1.txt
│      File2.txt
│      File3.txt
│      File4.txt
│
├─dir2
│      EXT.txt
│      File1.txt
│      File2.txt
│      File3.txt
│      File5.txt
│
└─__pycache__
```
源文件与备份文件保持同步


```python
#---coding:utf8----
import os, sys
import filecmp
import re
import shutil
holderlist = []

def compareme(dir1, dir2):
    dircomp=filecmp.dircmp(dir1,dir2)
    only_in_one = dircomp.left_only  #源目录新闻界或目录
    diff_in_one = dircomp.diff_files #不匹配文件， 源目录文件已经发生变化
    dirpath = os.path.abspath(dir1) #定义源目录绝对路径
    #将更新文件名或目录追加到holderlist
    [holderlist.append(os.path.abspath(os.path.join(dir1,x))) for x in only_in_one]
    [holderlist.append(os.path.abspath(os.path.join(dir1,x))) for x in diff_in_one]
    if len(dircomp.common_dirs) > 0:
        for item in dircomp.common_dirs:
            compareme(os.path.abspath(os.path.join(dir1, item)),\
                      os.path.abspath(os.path.join(dir2, item)))
    return holderlist

def main():
    if len(sys.argv) > 2:
        dir1 = sys.argv[1]
        dir2 = sys.argv[2]
    else:
        print "Usage: ", sys.argv[0], "datadir backupdir"
        sys.exit()
    source_files = compareme(dir1, dir2)
    dir1 = os.path.abspath(dir1)

    if not dir2.endswith('/'): dir2 = dir2 + '/'
    dir2 = os.path.abspath(dir2)
    destination_files = []
    createdir_bool = False

    for item in source_files:   #遍历返回的差异文件或目录清单
        destination_dir = re.sub(dir1, dir2, item)  #将源目录差异路径清单对应替换

        destination_files.append(destination_dir)
        if os.path.isdir(item):    #如果差异路径为目录且不存在，则在备份目录创建
            if not os.path.exists(destination_dir):
                os.makedirs(destination_dir)
                createdir_bool = True  #再次调用compareme函数标记

    if createdir_bool:
        destination_files = []
        source_files = []
        source_files = compareme(dir1, dir2)
        for item in source_files:
            destination_dir = re.sub(dir1, dir2, item)
            destination_files.append(destination_dir)
    print "update item:"
    print source_files
    copy_pair = zip(source_files, destination_files)
    for item in copy_pair:
        if os.path.isfile(item[0]):
            shutil.copyfile(item[0], item[1])

if __name__ == '__main__':
    main()
```

参考：

 - [filecmp --- 文件及目录的比较](https://docs.python.org/zh-cn/3.7/library/filecmp.html)
 - [Python: filecmp.cmp() method](https://www.geeksforgeeks.org/python-filecmp-cmp-method/)
