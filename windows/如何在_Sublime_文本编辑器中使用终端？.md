

---
## 第 1 步
打开 `Sublime Text`
## 第 2 步
使用命令 `Ctrl+shift+p` 打开命令面板

键入以下内容，直到出现该选项并选择它。`Package Control: Install Package`

注意：如果您是第一次使用包控制，则需要安装它。

输入入“`Terminus`”并选择它。等待它完成安装并重新启动 `sublime text`。

## 第 3 步
现在转到`Preferences >Package Settings > Terminus > Command Palette`
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6802107f501128fd1949836150738643.png)
代码粘贴至文件

```bash
[
   {
        "caption": "Terminal (panel)",
        "command": "terminus_open",
        "args"   : {
           "cmd": "bash",
           "cwd": "${file_path:${folder}}",
           "title": "Command Prompt",
           "panel_name": "Terminus"
        }
   },
]  
```

> 注意：以上代码适用于 `Linux` 用户，对于 `Windows` 用户，您必须输入“`cmd.exe`”代替“`bash`”

##  第 4 步
现在转到`Preferences >Package Settings > Terminus > Key Bindings`
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/fc5cd36d46c65431f5295c9700506b99.png)
将此代码粘贴到 `Default sublime Keymap` 并保存：

```bash
[
   {
       "keys": ["alt+1"],
       "command": "terminus_open",
       "args" : {
           "cmd": "bash",
           "cwd": "${file_path:${folder}}",
           "panel_name": "Terminus"
       }
   }
] 
```

> 注意：以上代码适用于`Linux`用户，`Windows`用户需要输入“`cmd.exe`”代替“`bash`”，这里我们保留快捷键为“`alt+1`”，您可以使用自己的键。

##  第 5 步
每当您想使用终端时，请按`alt+1`并在终端中关闭终端类型 `exit` 并按 `Enter`。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0107daeec1fdd849bc31488f815004fc.png)
✈<font color=	#FF4500 size=4 style="font-family:Courier New">推荐阅读：</font>

 - [How to Use Terminal in Sublime Text Editor ?](https://www.geeksforgeeks.org/how-to-use-terminal-in-sublime-text-editor/)


