#  shell 批量修改文件名
tags: 任务



---
## 1. 添加

```bash
$ ls
file1  file2  file3  file4
$ cat mv1.sh
#!/bin/bash
for file in `ls file*`
do
   mv $file `echo "${file}.txt" `
done

$ bash mv1.sh 
$ ls
file1.txt  file2.txt  file3.txt  file4.txt
```
```bash
$ ls
file.1  file.2  file.3  file.4
$ cat mv1.sh
#!/bin/bash
for file in `ls file*`
do 
   #mv $file  `echo ${file}.txt|sed 's/\.//1' `
   mv $file $(echo ${file}.txt|sed 's/\.//1')
done


$ bash mv1.sh 
$ ls
file1.txt  file2.txt  file3.txt  file4.txt
```
## 2. 修改

```bash
$ ls
file1.txt  file2.txt  file3.txt  file4.txt 

$ cat mv3.sh
#!/bin/bash
for file in `ls file*`
do
   mv $file ${file%.txt}.sh    #第一种方法
done

$ bash mv3.sh 
$ ls
file1.sh  file2.sh  file3.sh  file4.sh

$ cat mv4.sh
#!/bin/bash
for file in `ls file*`
do
   mv $file `echo $file |sed 's/\.sh/\.pdf/'`   #第二种方法
done

$ bash mv4.sh
$  ls
file1.pdf  file3.pdf  file2.pdf  file4.pdf 

```

另外一种方式
文件格式

```bash
_copr:copr.fedorainfracloud.org:frostyx:modulemd-tools-epel.repo.original  epel-testing.repo.original     Rocky-Devel.repo.original             Rocky-Plus.repo.original
docker-ce.repo.original                                                    offline.repo                   Rocky-Extras.repo.original            Rocky-PowerTools.repo.original
epel-modular.repo.original                                                 Rocky-AppStream.repo.original  Rocky-HighAvailability.repo.original  Rocky-ResilientStorage.repo.original
epel.repo.original                                                         Rocky-BaseOS.repo.original     Rocky-Media.repo.original             Rocky-RT.repo.original
epel-testing-modular.repo.original                                         Rocky-Debuginfo.repo.original  Rocky-NFV.repo.original               Rocky-Sources.repo.original
```

```bash
for file in *.repo.original; do mv "$file" "${file%.repo.original}.repo"; done
```
修改后

```bash
_copr:copr.fedorainfracloud.org:frostyx:modulemd-tools-epel.repo  epel-testing-modular.repo  Rocky-BaseOS.repo     Rocky-HighAvailability.repo  Rocky-PowerTools.repo
docker-ce.repo                                                    epel-testing.repo          Rocky-Debuginfo.repo  Rocky-Media.repo             Rocky-ResilientStorage.repo
epel-modular.repo                                                 offline.repo               Rocky-Devel.repo      Rocky-NFV.repo               Rocky-RT.repo
epel.repo                                                         Rocky-AppStream.repo       Rocky-Extras.repo     Rocky-Plus.repo              Rocky-Sources.repo

```

## 3. 删除

```bash
$  ls
file1.pdf  file3.pdf  file2.pdf  file4.pdf 

$ cat mv5.sh
#!/bin/bash
for file in `ls file*`
do
   mv $file `echo $file |sed 's/\.pdf//'`
done

bash mv5.sh
$ ls
file1  file2  file3  file4
```


