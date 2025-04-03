


------

## 1. vscode创建一个index.html

```bash
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>菜鸟教程(runoob.com)</title>
</head>
<body>
 
<h1>我的第一个标题</h1>
 
<p>我的第一个段落。</p>
 
</body>
</html>
```
## 2. 储存到一个工作区
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/95652dbee2d3bc06b0fd1dfc51e62958.png)

##  3. 安装Debugger for Chrome 插件
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f568aab1517e3d20866606146849d57e.png)
##  4. 生成并配置launch.json   
按F5启动调试
生成一个launch.json   
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a0a87969430374e18b204e5613d0c4b3.png)
修改配置

```bash
{
    // 使用 IntelliSense 了解相关属性。 
    // 悬停以查看现有属性的描述。
    // 欲了解更多信息，请访问: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [

        {
            "type": "chrome",
            "request": "launch",
            "name": "Launch Chrome against localhost",
            "file": "${workspaceRoot}/index.html", #当使用本地服务器请注释掉 url，并指定自己的文件名
            //"url": "http://localhost:8080",  #当使用外部服务器时,请注释掉 file
            "webRoot": "${workspaceFolder}"
        }
    ]
}
```

##  5. 再次调试
自动弹出chrome浏览器并指定html文件显示效果。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/476ff7a65ca271c789274bc83ced5fa3.png)

