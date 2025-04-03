
![](https://i-blog.csdnimg.cn/blog_migrate/deb91d22ba0e16189a189d542384e20b.png)


## 介绍
[CodeGeeX](https://codegeex.cn/)是一个具有130亿参数的多编程语言代码生成预训练模型，使用超过二十种编程语言训练得到。 基于CodeGeeX开发的插件可以实现通过描述生成代码、补全代码、代码翻译等一系列功能。 CodeGeeX同样提供可以定制的提示模式（Prompt Mode），构建专属的编程助手

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/2b566ac0925df1c687d9323ffb98166b.png)

## 特性
- 自动代码生成  
-  函数级代码完成
-  跨语言代码翻译  
-  10+编程语言  
-  支持VSCode，Jetbrains IDE

## 安装
- [https://codegeex.cn/downloadGuide#vscode](https://codegeex.cn/downloadGuide#vscode)
## 在线demo
![](https://i-blog.csdnimg.cn/blog_migrate/98fb225063151093388e0a811989fb94.png)

## 生成代码

需要保证VS Code版本 >= 1.68.0。安装插件并全局激活CodeGeeX，有以下四种使用模式：

- 隐匿模式: 保持CodeGeeX处于激活状态，当您停止输入时，会从当前光标处开始生成（右下角CodeGeeX图标转圈表示正在生成）。 生成完毕之后会以灰色显示，按Tab即可插入生成结果。
- 交互模式: 按Ctrl+Enter激活交互模式，CodeGeeX将生成X个候选，并显示在右侧窗口中（X 数量可以在设置的Candidate Num中修改）。 点击候选代码上方的use code即可插入。
- 翻译模式: 选择代码，然后按下Ctrl+Alt+T激活翻译模式，CodeGeeX会把该代码翻译成匹配您当前编辑器语言的代码。点击翻译结果上方的use code插入。您还可以在设置中选择您希望插入的时候如何处理被翻译的代码，您可以选择注释它们或者覆盖它们。
- 提示模式（实验功能）: 选择需要作为输入的代码，按Alt/Option+t触发提示模式，会显示预定义模板列表，选择其中一个模板，即可将代码插入到模板中进行生成。 这个模式高度自定义，可以在设置中 Prompt Templates修改或添加模板内容，为模型加入额外的提示。

![](https://i-blog.csdnimg.cn/blog_migrate/e1001dc7ae68971c05296446ffe89ed0.png)
### Add comment 添加注释
![](https://i-blog.csdnimg.cn/blog_migrate/541e0f709a619648a000b4bdbe384e12.png)

## 隐匿模式
在该模式中，CodeGeeX将在您停止输入时，从光标处开始生成（右下角CodeGeeX图标转圈表示正在生成）。生成完毕之后会以灰色显示，按Tab即可插入生成结果。 在生成多个候选的情况下，可以使用`Alt/Option+[ 或 ]`在几个候选间进行切换。如果你对现有建议不满意，可以使用`Alt/Option+N`去获得新的候选。可以在设置中改变Candidate Num（增加个数会导致生成速度相对变慢）。注意：生成总是从当前光标位置开始，如果您在生成结束前移动光标位置，可能会导致一些bugs。我们正在努力使生成速度变得更快以提升用户体验。

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/462809b0d02db10e6e7bf2404d51fcb1.gif#pic_center)


### interactive mode 交互模式
在该模式中，按`Ctrl+Enter`激活交互模式，CodeGeeX将生成X个候选，并显示在右侧窗口中（X 数量可以在设置的Candidate Num中修改）。 点击候选代码上方的use code即可插入结果到为当前光标位置。




login 登陆账号
Prompt mode 灵活功能
### Translation mode 代码转换
代码咨询
![](https://i-blog.csdnimg.cn/blog_migrate/18157bacd2bc356e2bbda7833a116e40.png)
			翻译模式：
贴入一段待翻译代码，`Ctrl+Alt+T`/ `ctrl+option+T`激活
翻译模式，根据提示选择该代码语言，CodeGeeX会匹配到当前编辑器语言。
点击 use code^插入代码。可以在设置中选择注释或者覆盖原来的代码。
![](https://i-blog.csdnimg.cn/blog_migrate/04ea5eed63a4ba3bc8d6d77aba4f549c.png)

提示模式：
预先写好自定义模板文件（如上）
在输入中加入额外提示例子，按Option/ Alt+T进入提示模式，选择特定模板生成代码。
## 插件配置
@ext:AMiner.codegeex
![](https://i-blog.csdnimg.cn/blog_migrate/bf5503f575e6762cce32c268b8f7367b.png)

Candidate Num，The candidate list of code.   The more the slower the inference.
候选编号，代码的候选列表。推理越慢

Completion Delay
The delay in seconds to start getting completions without new change in the editor.   You can change this value to get better experience in coding with our extension and to avoid some unuseful completion suggestions. 
延迟完成 
在没有对编辑器进行新更改的情况下开始完成的秒数延迟。您可以更改此值，以获得更好的编码经验与我们的扩展，并避免一些无用的完成建议。

Temp
Temp controls the randomness of output, range: [0.01, 1]. Higher temperature means more randomness, and the model will return creative results.
临时 
温度控制输出的随机性，范围:[0.01,1]。更高的温度意味着更多的随机性，模型将返回创造性的结果。

Topk
Top-k keeps the k candidate tokens with the highest probabilities, range [0, 40]. Top-k=0 means disabled.
Top-k保持具有最高概率的k个候选令牌，范围[0，40]。Top-k=0表示禁用。

Topp
Top-p keeps the candidate tokens whose probabilities sum to p, range [0, 1]. Top-p=0 means disabled.
Top-p保持其概率总和为p，范围[0，1]的候选令牌。Top-p=0表示禁用。

Disabled For
DisabledFor is a list of specific languages that will be disabled temporarily for the extension, you can set manully language* as the key and then set the value true to disable a language or false to re-enable it settings (this will need to restart vscode) or do this by clicking icon in the status bar. * language should be a valid vscode language id, like python, shellscript, csharp, objective-cpp and etc. You can click the language option in status bar to find the list and language ids are in the parentheses.
DisabledFor是一个特定语言的列表，将被暂时禁用的扩展，你可以设置manully language* 作为关键字，然后设置值true以禁用语言或false以重新启用它的设置（这将需要重新启动vscode）或通过单击状态栏中的图标来实现这一点。* language应该是一个有效的vscode语言id，如python、shellscript、csharp、objective-cpp等。您可以单击状态栏中的语言选项以查找列表，括号中显示语言ID。

Enable Extension
Check this if you want to enable stealth mode of the extension.
如果要启用扩展的隐藏模式，请选中此选项。

Generation Preference
You can choose the preference when generating code. If you choose automatic, the extension will generate whether a block or a line depending on your input. If you choose line by line, the extension will generate a code line, a comment line or a code line and a comment line for you each time.
您可以在生成代码时选择首选项。如果选择自动，扩展将根据您的输入生成块或线。如果您选择逐行，则扩展将每次为您生成一个代码行、一个注释行或一个代码行和一个注释行。

Only Key Control
Check this if you want to get suggestions only in need by pressing alt/option+\ in stealth mode.
如果您想在隐身模式下按alt/option+\只在需要时获取建议，请选中此选项。


## 原理
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/22c5587de1cc1671aa79cc3a6655c08f.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e7817a53475b42b480fe4498672cfe5e.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/1bd70737c4d507ceb950d55fcfbdc9f8.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/46fe6f07cc14680ff76da5f865fc4031.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/2235c037bb4d1fa63a0ed8ca719ff6b1.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c8df991f3b237019f970c27f6a66ab0e.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/fd7e51a12d833539ac933a6735d0736e.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/16837156b22f000a6ff1bdc6f618b287.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3981475e60e7d9fb9aacbe7374fe04b1.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/5d2fc8c22be46435600fefcf864b2942.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/bf850e32634eb96f20f30bf780136539.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/748a6fd53c7d71cc03741c0c64e7e70d.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3fedd10bfdbef8a5506a57df0951770d.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b070cd95b08b71e62cbede3ed0edf53a.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/43d72e17427ec7cf2d7116123cc71a15.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/5699fca0abd5b3ecb60af7c67765fa85.png)

