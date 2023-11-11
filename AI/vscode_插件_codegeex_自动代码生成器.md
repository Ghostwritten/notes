
![](https://img-blog.csdnimg.cn/283519f8a4974c4a9a6a6d29470ccc13.png)


## 介绍
[CodeGeeX](https://codegeex.cn/)是一个具有130亿参数的多编程语言代码生成预训练模型，使用超过二十种编程语言训练得到。 基于CodeGeeX开发的插件可以实现通过描述生成代码、补全代码、代码翻译等一系列功能。 CodeGeeX同样提供可以定制的提示模式（Prompt Mode），构建专属的编程助手

![在这里插入图片描述](https://img-blog.csdnimg.cn/b756c58f51194f268cc2df7062cf8b40.png)

## 特性
- 自动代码生成  
-  函数级代码完成
-  跨语言代码翻译  
-  10+编程语言  
-  支持VSCode，Jetbrains IDE

## 安装
- [https://codegeex.cn/downloadGuide#vscode](https://codegeex.cn/downloadGuide#vscode)
## 在线demo
![](https://img-blog.csdnimg.cn/605bbda9d4664d17a96fb27ec912f48b.png)

## 生成代码

需要保证VS Code版本 >= 1.68.0。安装插件并全局激活CodeGeeX，有以下四种使用模式：

- 隐匿模式: 保持CodeGeeX处于激活状态，当您停止输入时，会从当前光标处开始生成（右下角CodeGeeX图标转圈表示正在生成）。 生成完毕之后会以灰色显示，按Tab即可插入生成结果。
- 交互模式: 按Ctrl+Enter激活交互模式，CodeGeeX将生成X个候选，并显示在右侧窗口中（X 数量可以在设置的Candidate Num中修改）。 点击候选代码上方的use code即可插入。
- 翻译模式: 选择代码，然后按下Ctrl+Alt+T激活翻译模式，CodeGeeX会把该代码翻译成匹配您当前编辑器语言的代码。点击翻译结果上方的use code插入。您还可以在设置中选择您希望插入的时候如何处理被翻译的代码，您可以选择注释它们或者覆盖它们。
- 提示模式（实验功能）: 选择需要作为输入的代码，按Alt/Option+t触发提示模式，会显示预定义模板列表，选择其中一个模板，即可将代码插入到模板中进行生成。 这个模式高度自定义，可以在设置中 Prompt Templates修改或添加模板内容，为模型加入额外的提示。

![](https://img-blog.csdnimg.cn/d414ccab38054f32aeb0335e033d39af.png)
### Add comment 添加注释
![](https://img-blog.csdnimg.cn/d09d0041ef9d4e4e8ce66f221c64cc67.png)

## 隐匿模式
在该模式中，CodeGeeX将在您停止输入时，从光标处开始生成（右下角CodeGeeX图标转圈表示正在生成）。生成完毕之后会以灰色显示，按Tab即可插入生成结果。 在生成多个候选的情况下，可以使用`Alt/Option+[ 或 ]`在几个候选间进行切换。如果你对现有建议不满意，可以使用`Alt/Option+N`去获得新的候选。可以在设置中改变Candidate Num（增加个数会导致生成速度相对变慢）。注意：生成总是从当前光标位置开始，如果您在生成结束前移动光标位置，可能会导致一些bugs。我们正在努力使生成速度变得更快以提升用户体验。

![在这里插入图片描述](https://img-blog.csdnimg.cn/917236e991cd4b64bfda5fda9401eeaa.gif#pic_center)


### interactive mode 交互模式
在该模式中，按`Ctrl+Enter`激活交互模式，CodeGeeX将生成X个候选，并显示在右侧窗口中（X 数量可以在设置的Candidate Num中修改）。 点击候选代码上方的use code即可插入结果到为当前光标位置。




login 登陆账号
Prompt mode 灵活功能
### Translation mode 代码转换
代码咨询
![](https://img-blog.csdnimg.cn/0bd67435e60c45059ff0b143a9d332b2.png)
			翻译模式：
贴入一段待翻译代码，`Ctrl+Alt+T`/ `ctrl+option+T`激活
翻译模式，根据提示选择该代码语言，CodeGeeX会匹配到当前编辑器语言。
点击 use code^插入代码。可以在设置中选择注释或者覆盖原来的代码。
![](https://img-blog.csdnimg.cn/410dfe1c751c4bebb0bfa2665b34638e.png)

提示模式：
预先写好自定义模板文件（如上）
在输入中加入额外提示例子，按Option/ Alt+T进入提示模式，选择特定模板生成代码。
## 插件配置
@ext:AMiner.codegeex
![](https://img-blog.csdnimg.cn/b9440848d5d444dabb950d8adf2a19f9.png)

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
![在这里插入图片描述](https://img-blog.csdnimg.cn/a1207d0de5e4471bb5ba1958a5327628.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/cb123eea3e77449b903a1af1385cc0f7.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/25a8f6671b3f43798f058c2051c8dd42.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/35b9b90593b84f63aa64c4e70c694378.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/f21752f38d07453bbaca2fe29419f567.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/b8e7680289c04eb8aa22736014d07cc0.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/5557a6d1cc9c4b5f8da4add8122d1095.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/cc864d0197d34bff9a74b5a441309fee.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/5ca0ed65d2fe4cb897c3dd4ad63174d4.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/6a02de92971146afb6e9966e9c02dd65.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/bcac1471796b40c78f030980982cc5ac.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/42dfda5b6c50441ba4f4002fbb8a866d.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/4833b8b35e344ceda77bf31850a5d242.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/0664bed4e75840c8a33cae0de12decb7.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/edf450f060bf4fa2a9778d6ea7b9a884.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/28404b5cd40048efa7a75c1533108ce3.png)

