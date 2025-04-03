![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e751ea77db13408ebf9784fba1dece13.png)





## AI
###  CodeGeeX
[CodeGeeX](https://codegeex.cn/zh-CN) 可以根据自然语言注释描述的功能自动生成代码，也可以根据已有的代码自动生成后续代码，补全当前行或生成后续若干行，帮助你提高编程效率。

-  代码翻译
基于AI大模型对代码进行语义级翻译，支持多种编程语言互译，准确率高。 
- 支持多种语言
支持Python、Java、C++/C、JavaScript、Go等多种语言。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/76d6372b7953e02248c3295ca1ce7e98.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a68fa8fae9cbdd45c83b6cc77852df88.png)

###  GitHub Copilot 
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/9dd766a37923d94b67aea73966b74e06.png)



##  项目管理
###  Project Manager

以下是项目管理器提供的一些功能：

- 将任何文件夹或工作区保存为项目
- 自动检测 Git、Mercurial 或 SVN 存储库
- 使用标签组织您的项目
- 在同一窗口或新窗口中打开项目
- 识别已删除/重命名的项目
- 标识当前项目的状态栏
- 专门的侧边栏



![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/1673b956300266fa45aa7be417ba8b92.png)


## markdown




###  markdown-pdf 
[markdown-pdf](https://marketplace.visualstudio.com/items?itemName=yzane.markdown-pdf)  扩展将Markdown文件转换为PDF，HTML，PNG或JPEG文件。


![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/22953e1c4ab6f4524a3f6387f9226939.png)

输出的 pdf 可以自定义 

自定义css，同样在setting.json里面加入如下配置

```bash
// 路径和css文件名以你的文件为准
"markdown-pdf.styles": ["E:\\css\\markdowncss.css"],
```

###  Paste Image

[Paste Image](https://marketplace.visualstudio.com/items?itemName=mushan.vscode-paste-image) 插件将图像直接从剪贴板粘贴到 markdown/asciidoc（或其他文件）！

支持Mac/Windows/Linux！并支持配置目标文件夹。

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d23924a2afe3b67273b635fd39fa5d1a.png)

将图片保存到当前文档的`img/文件名/`路径下

```bash
"pasteImage.path": "${currentFileDir}/imgs",
"pasteImage.basePath": "${currentFileDir}/imgs/${currentFileNameWithoutExt}",
"pasteImage.forceUnixStyleSeparator": true,
"pasteImage.insertPattern": "${imageSyntaxPrefix}./img/${currentFileNameWithoutExt}/${imageFilePath}${imageSyntaxSuffix}",
"pasteImage.defaultName": "${currentFileNameWithoutExt}-Y-MM-DD-HH-mm-ss"
```




### Markdown Preview Enhanced
 [Markdown Preview Enhanced](https://marketplace.visualstudio.com/items?itemName=shd101wyy.markdown-preview-enhanced) 是一个扩展，为您提供了许多有用的功能，如自动滚动同步，数学排版，美人鱼，PlantUML，pandoc，PDF导出，代码块，演示文稿编写器等。它的很多想法都受到了Markdown Preview Plus和RStudio Markdown的启发。

插件使用说明：[https://shd101wyy.github.io/markdown-preview-enhanced/#/zh-cn/](https://shd101wyy.github.io/markdown-preview-enhanced/#/zh-cn/)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/cbe5a82e0b108bae712bf6d8af109011.png)
###  Markdown All in One

[Markdown All in One](https://marketplace.visualstudio.com/items?itemName=yzhang.markdown-all-in-one) 插件具备所有你需要的Markdown（键盘快捷键，目录，自动预览等）

command + shift + p ：搜索指定功能
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d0dcb9b98060b01e3cb7755f37b3a69b.png)

markdown 更多参考：
- [将 VS Code 打造成一个体验舒适的 Markdown 编辑器](https://blog.cxplay.org/works/vscode-to-markdown-editor/#%E8%BD%AF%E4%BB%B6%E5%AE%89%E8%A3%85)
##  yaml
###  yaml
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/295c706c795618bf19fb55b6c70de2f5.png)

## 笔记 

###  Notes

安装完后右下角会弹出选择存储位置，否则当你点击create netnote 会报 `Notes.newnote not found` 错误，因此，第一次从“活动栏”或通过“命令托盘”访问扩展时，“注释”将提示您输入存储位置。如果要更改存储位置，稍后可以从命令托盘访问Notes设置。选择存储位置后，可以从活动栏中的“便笺”图标或通过命令托盘访问便笺。

Notes.notesLocation: 设置存储位置

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/bf89d4c291df51bf66125ba3f8085c49.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/9330aaf831a4f8d5063e663dd409d731.png)

##  自动化

###  Run on Save

[Run on Save](https://marketplace.visualstudio.com/items?itemName=emeraldwalk.RunOnSave) 扩展允许配置命令，只要文件保存在vscode中，这些命令就会运行。
>注意：命令仅在保存现有文件时运行。创建新文件，并保存为...别触发指令。


- 配置保存文件时运行的多个命令
- 触发命令运行的文件的正则表达式模式匹配
- 同步和异步支持
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/1afb77fbf50fc8de3842b233ec8955f4.png)

- [Using VS Code for notes taking](https://dev.to/theagilemonkeys/using-vs-code-for-notes-taking-3eof)
## 思维导图

###  vscode-mindmap
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ec1c0108de5aad3d135aee1a14353bc8.png)


##  开发

###  Visual Studio IntelliCode
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/25f68cfd3c1ff43a188195b5adeddb72.png)

###  GitHub Repositories
 [GitHub Repositories](https://marketplace.visualstudio.com/items?itemName=GitHub.remotehub) 插件可以远程打开 github 仓库，浏览十分方便。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/37daacb9d3c20e9ef4b1cf04400cebd6.png)

## 云原生

###  docker

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/08f6ff6ccc660f3b4716fc9713fcf4b4.png)



## 数据库

###  mysql

[mysql](https://marketplace.visualstudio.com/items?itemName=cweijan.vscode-mysql-client2) 插件是VSCode的数据库客户端，支持管理器MySQL/MariaDB、PostgreSQL、SQLite、Redis、ClickHouse、达梦、ElasticSearch，同时作为SSH客户端，让你的生产力发挥到极致！
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d98a8a71348dcbe5c4aa4ac8c7a0356d.png)

连接数据库

- 打开数据库资源管理器面板，然后单击+按钮。
- 选择你的数据库类型，输入连接配置，然后点击连接按钮.
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8d68521b9320b00d04a27e71a83b2346.png)
创建两个面板是因为在某些情况下需要同时查看SQL和NoSQL数据，您可以通过长按将面板拖到另一个面板。

管理库表

- 单击表以打开表视图。
- 单击表格旁边的按钮打开新表格视图。
- 然后您可以在表视图上进行数据修改。

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6bdea6b6c466b31ca912970335fed303.png)


##  娱乐

###  超越鼓励师
[vscode-ycy](https://github.com/formulahendry/vscode-ycy) 插件除了每过一小时会自动弹出提醒页面，也可以按 F1, 然后输入 ycy: 打开提醒页面来打开提醒页面。


![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/220593610ca476fc6b0fd0b2ba341d27.png)

- [VSocde 中文社区](https://github.com/vscodecc)
- [《Visual Studio Code 权威指南》](https://github.com/formulahendry/awesome-vscode-cn)




