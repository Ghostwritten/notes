
#  Windows Terminal 命令面板

命令调色板让您看到哪些操作可以在Windows终端中运行。有关如何定义操作的详细信息，[请参阅动作页面](https://learn.microsoft.com/en-us/windows/terminal/customize-settings/actions)。

##  1. 调用命令调色板
您可以通过键入以下命令来调用命令面板`Ctrl+Shift+P`。这可以通过添加`commandPalette` 命令添加到键绑定中。

```bash
{ "command": "commandPalette", "keys": "ctrl+shift+p" }
```

##  2. 命令行模式
如果您要输入wt 命令插入到命令调色板中，可以通过删除> 字符。这将运行wt 命令。有关wt 命令可以在[命令行参数页面](https://learn.microsoft.com/en-us/windows/terminal/command-line-arguments?tabs=windows)。

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/655c71fd42dd236e350d093f79ad8ea6.gif#pic_center)
您可以添加自定义键绑定，以便直接在命令行模式下调用命令调色板。

```bash
{ "command": "commandPalette", "launchMode": "commandLine", "keys": "" }
```

##  3. 将图标添加到命令
您可以选择将图标添加到在settings.json ，可以通过添加icon 属性添加到操作。图标可以是图像的路径、[Segoe MDL2资产](https://learn.microsoft.com/en-us/windows/apps/design/style/segoe-ui-symbol-font)，或任何字符，包括表情符号。

```bash
{ "icon": "C:\\Images\\my-icon.png", "name": "New tab", "command": "newTab", "keys": "ctrl+shift+t" },
{ "icon": "\uE756", "name": "New tab", "command": "newTab", "keys": "ctrl+shift+t" },
{ "icon": "⚡", "name": "New tab", "command": "newTab", "keys": "ctrl+shift+t" }
```

##  4. 嵌套指令
嵌套命令允许您将多个命令分组到命令调色板中的一个项目下。下面的示例将字体大小调整命令分组到名为“更改字体大小..."的命令调色板项目下。

```bash
{
    "name": "Change font size...",
    "commands": [
        { "command": { "action": "adjustFontSize", "delta": 1 } },
        { "command": { "action": "adjustFontSize", "delta": -1 } },
        { "command": "resetFontSize" },
    ]
}
```
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/bf72b09a490e8ff3c0238212f24f4577.gif#pic_center)
##  5. 可迭代命令
可迭代命令允许您同时创建多个命令，这些命令是从您的设置中定义的其他对象生成的。目前，您可以为您的配置文件和配色方案创建可迭代的命令。在运行时，这些命令将扩展为针对给定类型的每个对象的一个命令。

您目前可以迭代以下属性：
|iterateOn	|Property|	属性语法|
|--|--|--|
|profiles|	name|	"name": "${profile.name}"
|profiles|	icon	|"icon": "${profile.icon}"
|schemes	|name|	"name": "${scheme.name}"

例子
为每个配置文件创建一个新的选项卡命令。

```bash
{
    "iterateOn": "profiles",
    "icon": "${profile.icon}",
    "name": "${profile.name}",
    "command": { "action": "newTab", "profile": "${profile.name}" }
}
```
在上面的例子中：
- `"iterateOn": "profiles"`将为每个配置文件生成一个命令。
- 在运行时，终端将替换`${profile.icon}`为每个配置文件的图标和`${profile.name}`每个配置文件的名称。

如果您有三个配置文件：

```bash
"profiles": [
	{ "name": "Command Prompt", "icon": null },
	{ "name": "PowerShell", "icon": "C:\\path\\to\\icon.png" },
	{ "name": "Ubuntu", "icon": null },
]
```
上述命令的行为类似于以下三个命令：

```bash
{
    "icon": null,
    "name": "Command Prompt",
    "command": { "action": "newTab", "profile": "Command Prompt" }
},
{
    "icon": "C:\\path\\to\\icon",
    "name": "PowerShell",
    "command": { "action": "newTab", "profile": "PowerShell" }
},
{
    "icon": null,
    "name": "Ubuntu",
    "command": { "action": "newTab", "profile": "Ubuntu" }
}
```
也可以组合嵌套和可迭代的命令。例如，您可以将上面的三个“新建选项卡”命令组合在命令面板中的单个“新建选项卡”条目下，如上图所示，方法如下：

```bash
{
    "name": "New tab",
    "commands": [
        {
            "iterateOn": "profiles",
            "icon": "${profile.icon}",
            "name": "${profile.name}",
            "command": { "action": "newTab", "profile": "${profile.name}" }
        }
    ]
}
```
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/10520b344f8bb61cca2e0b1020c993be.gif#pic_center)
##  6. 隐藏命令
如果你想在你的键绑定列表中保留一个命令但不让它出现在命令面板中，你可以通过将其设置name为隐藏它null。下面的示例隐藏了命令面板中的“新建选项卡”操作。

```bash
{ "name": null, "command": "newTab", "keys": "ctrl+shift+t" }
```
参考：
- [How to use the command palette in Windows Terminal](https://learn.microsoft.com/en-us/windows/terminal/command-palette)
