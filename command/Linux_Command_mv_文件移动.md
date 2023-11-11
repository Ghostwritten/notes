#  Linux Command mv 文件移动



----

![在这里插入图片描述](https://img-blog.csdnimg.cn/52df4bdd0dfd4faebd1922d758ab5507.gif#pic_center)

### 1. 移动文件

```bash
$ ls
dir1  file1
$ mv file1 dir1/
$ ls
dir1
$ ls dir1/
file1
```

### 2. 移动多个文件
第一种：

```bash
$ ls
dir1  file1  file2
$ mv file1 file2 dir1/
$ ls
dir1
$ ls dir1/
file1  file2
```

第二种

```bash
$ ls
dir1  file1  file2
$ mv file* dir1/
$ ls 
dir1
$ ls dir1/
file1  file2
```


### 3. 移动目录

```bash
$ ls
dir1
$ mkdir dir2
$ ls
dir1  dir2
$ mv dir1 dir2
$ ls
dir2
$ ls dir2/
dir1
```


### 4. 重命名文件或目录

```bash
$ ls
dir2  file1
$ mv file1 file2
$ mv dir2 dir1
$ ls
dir1  file2
```

### 5. 打印移动信息
当你移动或重命名一大堆文件或目录时，你可能会想在不去目标位置去查看的情况下知道你自己的命令是否成功地执行了。这就要用到-v选项了

```bash
$ ls
dir1  file1  file2  file3  file4
$ mv -v file* dir1/
"file1" -> "dir1/file1"
"file2" -> "dir1/file2"
"file3" -> "dir1/file3"
"file4" -> "dir1/file4"

$ ls
dir1  subdir1  subdir2  subdir3  subdir4
$ mv -v subdir*  dir1/
"subdir1" -> "dir1/subdir1"
"subdir2" -> "dir1/subdir2"
"subdir3" -> "dir1/subdir3"
"subdir4" -> "dir1/subdir4"

```


### 6. 使用交互模式
当你将文件移动到其它位置，而那个位置恰好有同样的文件，这时 mv 命令会覆盖掉原来的文件。对于mv的这一行为一般不会有什么提示。如果想产生一个关于覆盖文件的提示，我们可以使用-i选项。（译注：通常发行版会通过alias命令，将-i作为默认选项，所以会有提示。）

```bash
$ cat /root/.bashrc 
# .bashrc

# User specific aliases and functions

alias rm='rm -i'
alias cp='cp -i'
alias mv='mv -i'
```


```bash
$ ls
dir1  file1
$ ls dir1/
file1
$ mv -i file1 dir1/
mv：是否覆盖"dir1/file1"？ y
$ ls
dir1
$ ls dir1/
file1
```

### 7. 使用更新选项

 - －u 则只在源文件比目标文件新时才执行更新。默认也会询问是否覆盖。
 -  -f  为强制覆盖

```bash
$ ll
总用量 0
drwxr-xr-x 2 root root 58 6月   9 15:29 dir1
-rw-r--r-- 1 root root  0 6月   9 15:29 file1
-rw-r--r-- 1 root root  0 6月   9 15:29 file2
-rw-r--r-- 1 root root  0 6月   9 15:29 file3
-rw-r--r-- 1 root root  0 6月   9 15:29 file4
$ ll dir1/
总用量 4
-rw-r--r-- 1 root root 4 6月   9 15:29 file1
-rw-r--r-- 1 root root 0 6月   9 15:29 file2
-rw-r--r-- 1 root root 0 6月   9 15:29 file3
-rw-r--r-- 1 root root 0 6月   9 15:29 file4
$ mv -uv file* dir1/
mv：是否覆盖"dir1/file1"？ y
"file1" -> "dir1/file1"

或者
$ mv -uvf file* dir1/
"file1" -> "dir1/file1"
```



### 8. 不要覆盖任何已存在的文件

 - -i 选项询问我们是否要覆盖文件
 - -n 选项将不会允许我们覆盖任何已存在的文件

```bash
$ ls
dir1  file1  file2  file3  file4
$ ls dir1/
file2  file3  file4
$ mv -nv file* dir1/
"file1" -> "dir1/file1"
```

### 9. 复制时创建备份
-b选项。该选项会在新文件覆盖旧文件时将旧文件做备份

```bash
$ ll 
总用量 8
drwxr-xr-x 2 root root 58 6月   9 15:37 dir1
-rw-r--r-- 1 root root  5 6月   9 15:38 file1
-rw-r--r-- 1 root root  5 6月   9 15:38 file2
-rw-r--r-- 1 root root  0 6月   9 15:29 file3
-rw-r--r-- 1 root root  0 6月   9 15:29 file4
$ ll dir1/
总用量 4
-rw-r--r-- 1 root root 5 6月   9 15:30 file1
-rw-r--r-- 1 root root 0 6月   9 15:29 file2
-rw-r--r-- 1 root root 0 6月   9 15:29 file3
-rw-r--r-- 1 root root 0 6月   9 15:29 file4

$ mv -bvf file* dir1/
"file1" -> "dir1/file1" (备份："dir1/file1~")
"file2" -> "dir1/file2" (备份："dir1/file2~")
"file3" -> "dir1/file3" (备份："dir1/file3~")
"file4" -> "dir1/file4" (备份："dir1/file4~")
$ ls dir1/
file1  file1~  file2  file2~  file3  file3~  file4  file4~

or如何没有更新的文件不想被覆盖与备份则：
$ mv -ubvf file* dir1/
"file1" -> "dir1/file1" (备份："dir1/file1~")
"file2" -> "dir1/file2" (备份："dir1/file2~")

```


