Alt + Enter -- 全屏
Alt + B --  快速打开新的连接
Alt + 1/2/3/4/5.../9 -- 切换到第1/2/3/4/5.../9个标签
Ctrl + A | Alt+A  -- 光标移至行首，windows复制功能应用后，使用Alt+A
Ctrl + E -- 光标移至行末  
Ctrl + U -- 清除当前行和缓存的输入，删除光标至行首所有字符
Ctrl + K -- 删除当前光标至行末的字符
Ctrl + B -- 光标前移1个字符
Ctrl + F -- 光标后移1个字符
Ctrl + H -- 删除光标前的1个字符
Ctrl + D -- 删除光标后1个字符
Ctrl + J -- 回车
Ctrl + M -- 回车

Ctrl + P -- 显示前一条命令
Ctrl + N -- 下一条命令
Ctrl + T -- 交换光标前最后两个字符（思科路由器可用）
Ctrl + V -- 输入ctrl字符
Ctrl + W -- 删除当前光标至行首的字符

 SecureCRT 常用技巧


快捷键：
1、 ctrl + a :  移动光标到行首
2、 ctrl + e ：移动光标到行尾
3、 ctrl + d ：删除光标之后的一个字符
4、 ctrl + w ： 删除行首到当前光标所在位置的所有字符
5、 crtl + k ： 删除当前光标到行尾的所有字符
6、 alt + b ： 打开快速启动栏
7、 alt + 1/2/3... ： 在多个不同的session标签之间切换
 
鼠标复制：
options -> global options ->  Terminal  钩上Copy on select，并钩上paste on middle button
这样在secrecrt中用鼠标选中一段字符，就可以直接复制到剪切板，按鼠标中间的齿轮完成复制粘贴。
 
双击复制并打开新session:
options -> global options -> Terminal -> Tabs 选择Double-click action的下拉框为Clone tab，这样就可以在已经打开的session标签中鼠标双击，打开一个完全一样的新session标签。
 
用sftp与windows之间上传现在文件：
在一个已经打开的session中按alt + p组合键，打开一个该session的sftp，通过cd,ls查看远程服务器的文件，通过lcd,lls可以查看windows上面的的文件，通过put,get命令可以进行文件的上传下载，用习惯之后比rz,sz效率高。
 
键盘映射：
options ->  global options -> General -> Default Session,点击Edit default settings按钮，再Terminal -> Mapped Keys，在这里面用map a key按钮来设定键盘映射，对于经常需要输入的字符串，可以在这里设置，如密码。
 
保持连接：
options -> global options -> General -> Default Session,点击Edit default settings按钮，在Terminal中钩上Send protocol NO-OP, every 30 seconds，这样可以保证securecrt不会因为一段时间没有操作，而丢掉连接。


