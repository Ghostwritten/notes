


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
![在这里插入图片描述](https://img-blog.csdnimg.cn/9f4a48474fb140d79932873e8e337c9b.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

##  3. 安装Debugger for Chrome 插件
![在这里插入图片描述](https://img-blog.csdnimg.cn/ff7cbe8a7f1a498b9b50f0ba138c1f7b.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
##  4. 生成并配置launch.json   
按F5启动调试
生成一个launch.json   
![在这里插入图片描述](https://img-blog.csdnimg.cn/c79576db9f6045f0986ce3dd6b885fb3.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
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
![在这里插入图片描述](https://img-blog.csdnimg.cn/3927cfa3357443958cda70bc1ef5c529.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

