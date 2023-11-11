![在这里插入图片描述](https://img-blog.csdnimg.cn/1d96e39287444db8a9ad5a8617d7a682.png)





## AI
###  CodeGeeX
[CodeGeeX](https://codegeex.cn/zh-CN) 可以根据自然语言注释描述的功能自动生成代码，也可以根据已有的代码自动生成后续代码，补全当前行或生成后续若干行，帮助你提高编程效率。

-  代码翻译
基于AI大模型对代码进行语义级翻译，支持多种编程语言互译，准确率高。 
- 支持多种语言
支持Python、Java、C++/C、JavaScript、Go等多种语言。
![在这里插入图片描述](https://img-blog.csdnimg.cn/618ee973eb5243b7b208f32726f9cb9c.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/639951d720ed4de288a380d1f1d790ad.png)

###  GitHub Copilot 
![在这里插入图片描述](https://img-blog.csdnimg.cn/a32028d0a20a4952b736b863a03a09d8.png)



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



![在这里插入图片描述](https://img-blog.csdnimg.cn/fb36ed3887d745ce9d65dc61621ea3a5.png)


## markdown




###  markdown-pdf 
[markdown-pdf](https://marketplace.visualstudio.com/items?itemName=yzane.markdown-pdf)  扩展将Markdown文件转换为PDF，HTML，PNG或JPEG文件。


![在这里插入图片描述](https://img-blog.csdnimg.cn/eadd662250f64bdc9745182e575da831.png)

输出的 pdf 可以自定义 

自定义css，同样在setting.json里面加入如下配置

```bash
// 路径和css文件名以你的文件为准
"markdown-pdf.styles": ["E:\\css\\markdowncss.css"],
```

###  Paste Image

[Paste Image](https://marketplace.visualstudio.com/items?itemName=mushan.vscode-paste-image) 插件将图像直接从剪贴板粘贴到 markdown/asciidoc（或其他文件）！

支持Mac/Windows/Linux！并支持配置目标文件夹。

![在这里插入图片描述](https://img-blog.csdnimg.cn/68e16c4d4ce1425bb98b6684541557c3.png)

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

![在这里插入图片描述](https://img-blog.csdnimg.cn/59a622128a734fdfa59d86638683b501.png)
###  Markdown All in One

[Markdown All in One](https://marketplace.visualstudio.com/items?itemName=yzhang.markdown-all-in-one) 插件具备所有你需要的Markdown（键盘快捷键，目录，自动预览等）
![在这里插入图片描述](https://img-blog.csdnimg.cn/5bc9b3c985d247a8a4a00a3f53a853b1.png)

markdown 更多参考：
- [将 VS Code 打造成一个体验舒适的 Markdown 编辑器](https://blog.cxplay.org/works/vscode-to-markdown-editor/#%E8%BD%AF%E4%BB%B6%E5%AE%89%E8%A3%85)
##  yaml
###  yaml
![在这里插入图片描述](https://img-blog.csdnimg.cn/2200159c48604cc1b380d43d57006aaa.png)

## 笔记 

###  Notes

安装完后右下角会弹出选择存储位置，否则当你点击create netnote 会报 `Notes.newnote not found` 错误，因此，第一次从“活动栏”或通过“命令托盘”访问扩展时，“注释”将提示您输入存储位置。如果要更改存储位置，稍后可以从命令托盘访问Notes设置。选择存储位置后，可以从活动栏中的“便笺”图标或通过命令托盘访问便笺。

Notes.notesLocation: 设置存储位置

![在这里插入图片描述](https://img-blog.csdnimg.cn/f343b98be74d433e8e3afab822588f43.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/c60fefa600664c8e8ee3c7d40f5f8205.png)

##  自动化

###  Run on Save

[Run on Save](https://marketplace.visualstudio.com/items?itemName=emeraldwalk.RunOnSave) 扩展允许配置命令，只要文件保存在vscode中，这些命令就会运行。
>注意：命令仅在保存现有文件时运行。创建新文件，并保存为...别触发指令。


- 配置保存文件时运行的多个命令
- 触发命令运行的文件的正则表达式模式匹配
- 同步和异步支持
![在这里插入图片描述](https://img-blog.csdnimg.cn/2a5e97a8f4724cbdb8607183274da0c6.png)

- [Using VS Code for notes taking](https://dev.to/theagilemonkeys/using-vs-code-for-notes-taking-3eof)
## 思维导图

###  vscode-mindmap
![在这里插入图片描述](https://img-blog.csdnimg.cn/4ccf0cd7c1ee499eb0426d41f6a51cbd.png)


##  开发

###  Visual Studio IntelliCode
![在这里插入图片描述](https://img-blog.csdnimg.cn/350622c9ced84c3fac50fcec7053669f.png)

###  GitHub Repositories
 [GitHub Repositories](https://marketplace.visualstudio.com/items?itemName=GitHub.remotehub) 插件可以远程打开 github 仓库，浏览十分方便。
![在这里插入图片描述](https://img-blog.csdnimg.cn/2dbc990154944a6590126f1bddc93273.png)

## 云原生

###  docker

![在这里插入图片描述](https://img-blog.csdnimg.cn/343d905e772b4e6bb1c7ba119ee113b2.png)



## 数据库

###  mysql

[mysql](https://marketplace.visualstudio.com/items?itemName=cweijan.vscode-mysql-client2) 插件是VSCode的数据库客户端，支持管理器MySQL/MariaDB、PostgreSQL、SQLite、Redis、ClickHouse、达梦、ElasticSearch，同时作为SSH客户端，让你的生产力发挥到极致！
![在这里插入图片描述](https://img-blog.csdnimg.cn/ed797c13913f4936b4b81bd808d6c3b2.png)

连接数据库

- 打开数据库资源管理器面板，然后单击+按钮。
- 选择你的数据库类型，输入连接配置，然后点击连接按钮.
![在这里插入图片描述](https://img-blog.csdnimg.cn/7bb2d6b6155640d5ad477a9c4ef3c5ed.png)
创建两个面板是因为在某些情况下需要同时查看SQL和NoSQL数据，您可以通过长按将面板拖到另一个面板。

管理库表

- 单击表以打开表视图。
- 单击表格旁边的按钮打开新表格视图。
- 然后您可以在表视图上进行数据修改。

![在这里插入图片描述](https://img-blog.csdnimg.cn/3ac24c49947248378ce8de7b01a0c74c.png)


##  娱乐

###  超越鼓励师
[vscode-ycy](https://github.com/formulahendry/vscode-ycy) 插件除了每过一小时会自动弹出提醒页面，也可以按 F1, 然后输入 ycy: 打开提醒页面来打开提醒页面。


![在这里插入图片描述](https://img-blog.csdnimg.cn/3292f8f617de445692baf041332562dd.png)

- [VSocde 中文社区](https://github.com/vscodecc)
- [《Visual Studio Code 权威指南》](https://github.com/formulahendry/awesome-vscode-cn)




