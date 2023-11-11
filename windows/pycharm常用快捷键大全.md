## 1 常用
 Tab    鼠标选中多行代码后多行代码同时缩进，一次缩进四个字符
shift+Tab   鼠标选中多行代码同时左移一次左移四个字符
 Ctrl+  向上箭头 
 Ctrl+  向下箭头  to scroll through the history of commands.
ctrl+ Q   打印模块文档
 alt+ enter    安装选中模块  
Ctrl + Shift + F  Find则是在当前文件查找，快捷键为Ctrl + F。

## 2 编辑（Editing）

```bash
ctrl  + ？注释
Ctrl + Space 基本的代码完成（类、方法、属性）
Ctrl + Alt + Space 快速导入任意类
Ctrl + Shift + Enter 语句完成
Ctrl + P 参数信息（在方法中调用参数）
Ctrl + Q 快速查看文档
F1 Web帮助文档主页
Shift + F1 选中对象的Web帮助文档
Ctrl + 悬浮/单击鼠标左键 简介/进入代码定义
Ctrl + Z 撤销上次操作
Ctrl + Shift + Z 重做,恢复上次的撤销
Ctrl + F1 显示错误描述或警告信息
Alt + Insert 自动生成代码
Ctrl + O 重新方法
Ctrl + Alt + T 选中
Ctrl + / 行注释/取消注释
Ctrl + Shift + / 块注释
Ctrl + W 选中增加的代码块
Ctrl + Shift + W 回到之前状态
Ctrl + Shift + ]/[ 选定代码块结束、开始
Alt + Enter 快速修正
Ctrl + Alt + L 代码格式化
Ctrl + Alt + O 优化导入
Ctrl + Alt + I 自动缩进
Tab / Shift + Tab 缩进、不缩进当前行
Ctrl+X/Shift+Delete 剪切当前行或选定的代码块到剪贴板
Ctrl+C/Ctrl+Insert 复制当前行或选定的代码块到剪贴板
Ctrl+V/Shift+Insert 从剪贴板粘贴
Ctrl + Shift + V 从最近的缓冲区粘贴
Ctrl + D 复制选定的区域或行
Ctrl + Y 删除选定的行
Ctrl + Shift + J 添加智能线
Ctrl + Enter 智能线切割
Shift + Enter 另起一行
Ctrl + Shift + U 在选定的区域或代码块间切换
Ctrl + Delete 删除到字符结束
Ctrl + Backspace 删除到字符开始
Ctrl + Numpad+/- 展开/折叠代码块（当前位置：函数、注释等）
Ctrl + Shift + Numpad+/- 展开/折叠所有代码块
Ctrl + F4 关闭运行的选项卡
```

## 3 查找/替换(Search/Replace)

```bash
F3 下一个
Shift + F3 前一个
Ctrl + R 替换
Ctrl + Shift + R 全局替换
Ctrl + Shift + F 全局查找（可以在整个项目中查找某个字符串什么的，如查找某个函数名）
连续敲击两次Shift键 查找函数
```

## 4 运行(Running)

```bash
Alt + Shift + F10 运行模式配置
Alt + Shift + F9 调试模式配置
Shift + F10 运行
Shift + F9 调试
Ctrl + Shift + F10 运行编辑器配置
Ctrl + Alt + R 运行manage.py任务
```

## 5 调试(Debugging)

```bash
F8 跳过
F7 进入
Shift + F8 退出
Alt + F9 运行游标
Alt + F8 验证表达式
Ctrl + Alt + F8 快速验证表达式
F9 恢复程序
Ctrl + F8 断点开关
Ctrl + Shift + F8 查看断点
```

## 6 导航(Navigation)

```bash
Ctrl + N 跳转到类
Ctrl + Shift + N 跳转到符号
Alt + Right/Left 跳转到下一个、前一个编辑的选项卡（代码文件）
Alt + Up/Down跳转到上一个、下一个方法
F12 回到先前的工具窗口
Esc 从工具窗口回到编辑窗口
Shift + Esc 隐藏运行的、最近运行的窗口
Ctrl + Shift + F4 关闭主动运行的选项卡
Ctrl + G 查看当前行号、字符号
Ctrl + E 在当前文件弹出最近使用的文件列表
Ctrl+Alt+Left/Right 后退、前进
Ctrl+Shift+Backspace 导航到最近编辑区域（差不多就是返回上次编辑的位置）
Alt + F1 查找当前文件或标识
Ctrl+B / Ctrl+Click 跳转到声明
Ctrl + Alt + B 跳转到实现
Ctrl + Shift + I 查看快速定义
Ctrl + Shift + B 跳转到类型声明
Ctrl + U 跳转到父方法、父类
Alt + Up/Down 跳转到上一个、下一个方法
Ctrl + ]/[ 跳转到代码块结束、开始
Ctrl + F12 弹出文件结构
Ctrl + H 类型层次结构
Ctrl + Shift + H 方法层次结构
Ctrl + Alt + H 调用层次结构
F2 / Shift + F2 下一条、前一条高亮的错误
F4 / Ctrl + Enter 编辑资源、查看资源
Alt + Home显示导航条F11 书签开关
Ctrl + Shift + F11 书签助记开关
Ctrl + #[0-9] 跳转到标识的书签
Shift + F11 显示书签
```

## 7 搜索相关(Usage Search)

```bash
Alt + F7/Ctrl + F7 文件中查询用法
Ctrl + Shift + F7 文件中用法高亮显示
Ctrl + Alt + F7 显示用法
7、重构(Refactoring)
Alt + Delete 安全删除
Shift + F6 重命名文件
Ctrl + F6 更改签名
Ctrl + Alt + N 内联
Ctrl + Alt + M 提取方法
Ctrl + Alt + V 提取属性
Ctrl + Alt + F 提取字段
Ctrl + Alt + C 提取常量
Ctrl + Alt + P 提取参数
```

## 8 控制VCS/Local History

```bash
Ctrl + K 提交项目
Ctrl + T 更新项目
Alt + Shift + C 查看最近的变化
Alt + BackQuote(') VCS快速弹出
```

## 9 模版(Live Templates)

```bash
Ctrl + Alt + J 当前行使用模版
Ctrl + J 插入模版
```

## 10 基本(General)

```bash
Alt + #[0-9] 打开相应的工具窗口
Ctrl + Alt + Y 同步
Ctrl + Shift + F12 最大化编辑开关
Alt + Shift + F 添加到最喜欢
Alt + Shift + I 根据配置检查当前文件
Ctrl + BackQuote(') 快速切换当前计划
Ctrl + Alt + S 打开设置页
Ctrl + Shift + A 查找编辑器里所有的动作
Ctrl + Tab 在窗口间进行切换
```



