#  windows 解决方案





## cmd：dxdiag
![在这里插入图片描述](https://img-blog.csdnimg.cn/697aa619c17e4b48905865b0b579eed8.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/c5b9971ee01c4fa58415f2741190131f.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/6600eba33414423c9beca466044850d7.png)

##  开机自启
取消开机自启动权限：

 - `Win+R`，在打开的运行程序中输入 `msconfig`，回车

 - 在打开的系统配置中，选中上面tab栏中的启动，打开任务管理器，这里面列出的程序就是开机会自启动的程序

 - 选中程序，点击禁用即可禁止开机自启动。



启动开机自启动权限：

1. `Win+R`，在打开的运行程序中输入 `shell:startup`，回车

2. 打开一个文件夹，把需要自启动的程序的exe执行文件或者快捷方式复制到该目录即可。



参考：

 - [可可西](https://www.cnblogs.com/kekec/p/3662125.html)

