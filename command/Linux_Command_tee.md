#  Linux Command tee 


---



## 1. 简介
Linux tee命令用于读取标准输入的数据，并将其内容输出成文件。
tee指令会从标准输入设备读取数据，将其内容输出到标准输出设备，同时保存成文件。

![在这里插入图片描述](https://img-blog.csdnimg.cn/a3de4dde357c4a8a95d3757e47d82e51.png)

## 2. 参数

```bash
-a或--append 　附加到既有文件的后面，而非覆盖它．
-i或--ignore-interrupts 　忽略中断信号。
--help 　在线帮助。
--version 　显示版本信息。
```

## 3. 实例

```bash
# 将进程信息通过管道输出到标准输出（终端）并覆盖写入到文件中。
ps -ef |tee info_a.log info_b.log

# 将进程信息通过管道输出到标准输出（终端）并追加写入到文件中。
ps -ef |tee -a info_a.log info_b.log
```

