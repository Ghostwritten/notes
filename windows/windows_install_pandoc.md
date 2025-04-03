


## 下载
- [https://github.com/jgm/pandoc/releases](https://github.com/jgm/pandoc/releases)






##  安装
`pandoc-3.1.3-windows-x86_64.msi `直接双击一路默认即可。

但安装后并没有得到该命令，我们需要配置环境变量，找到安装的命令位置

```bash
C:\Users\XH\AppData\Local\Pandoc\pandoc.exe
```
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/44a37c7cf374e46a9ec5291c7e054a21.png)

##  测试
```bash
C:\Users\XH>pandoc  --version
pandoc 3.1.2
Features: +server +lua
Scripting engine: Lua 5.4
User data directory: C:\Users\XH\AppData\Roaming\pandoc
Copyright (C) 2006-2023 John MacFarlane. Web:  https://pandoc.org
This is free software; see the source for copying conditions. There is no
warranty, not even for merchantability or fitness for a particular purpose.

```

## 使用
- [https://pandoc.org/MANUAL.html](https://pandoc.org/MANUAL.html)

```bash
 pandoc -s .\k8s-certs-update-fail.md -o k8s-certs-update-fail.docx
```

