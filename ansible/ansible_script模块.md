

## 1. script介绍

script 模块可以帮助我们在远程主机上执行 ansible 管理主机上的脚本，也就是说，脚本一直存在于 ansible 管理主机本地，不需要手动拷贝到远程主机后再执行。 

## 2. 常用参数

free_form参数 ：必须参数，指定需要执行的脚本，脚本位于 ansible 管理主机本地，并没有具体的一个参数名叫 free_form。
chdir参数 : 此参数的作用就是指定一个远程主机中的目录，在执行对应的脚本之前，会先进入到 chdir 参数指定的目录中。

creates参数 ：使用此参数指定一个远程主机中的文件，当指定的文件存在时，就不执行对应脚本。

removes参数 ：使用此参数指定一个远程主机中的文件，当指定的文件不存在时，就不执行对应脚本。

## 3. 实例

```bash
$ ansible test1 -m script -a "chdir=/opt /testdir/testscript.sh"
test1 | SUCCESS => {
    "changed": true, 
    "rc": 0, 
    "stderr": "Shared connection to test1 closed.\r\n", 
    "stdout": "testscript\r\n", 
    "stdout_lines": [
        "testscript"
    ]
}
$ ansible test1 -m script -a "chdir=/opt /testdir/testscript.sh"
$ ansible test1 -m script -a "creates=/testdir/testfile1 /testdir/testscript.sh"
$ ansible  test1 -m script -a "removes=/testdir/testfile1 /testdir/testscript.sh"
```

