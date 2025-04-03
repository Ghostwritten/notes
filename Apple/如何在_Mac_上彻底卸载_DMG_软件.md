
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/9be02d6fde104f6a95a963be4d3306e6.jpeg)




## 1. 前言
在 macOS 上，许多应用程序是通过 **DMG（磁盘映像文件）** 安装的。虽然删除应用通常只需拖动到废纸篓，但这并不能彻底清理应用残留文件。本教程介绍如何**干净地卸载 DMG 安装的应用程序**。

## 2. 常规卸载方法

### 方法 1：手动删除应用程序
1. 打开 **访达（Finder）**。
2. 进入 **应用程序（Applications）** 文件夹。
3. 找到要卸载的应用，右键点击并选择 **移动到废纸篓**。
4. 清空废纸篓：右键点击废纸篓图标，选择 **清倒废纸篓**。





### 方法 2：使用 Launchpad 卸载（适用于 Mac App Store 安装的应用）
1. 打开 **Launchpad**（点击 Dock 栏上的火箭图标）。
2. 找到要删除的应用。
3. 长按应用图标，直到出现 **X** 按钮。
4. 点击 **X**，然后选择 **删除**。

> **注意**：此方法仅适用于 Mac App Store 下载的应用，DMG 安装的应用需要手动删除。


### 方法3

首先，commad + 空格，打开finder功能，搜索关键词
下拉到底部，使用访达显示全部

在右上角点击 +

选择“系统文件”，选择“包括”，就会显示全部关于你搜索的应用程序的全部文件，然后把它们移动到废纸篓即可。

在移动到废纸篓之前，要把应用程序退出。


## 3. 清理应用残留文件
即使删除了应用，仍然可能会有 **缓存、配置文件和日志文件** 残留在系统中。可以手动删除这些文件。

### 手动清理残留文件
1. 打开 **访达（Finder）**。
2. 在顶部菜单栏点击 **前往** > **前往文件夹**，输入以下路径并删除相关文件：
   - `~/Library/Application Support/<应用名称>`
   - `~/Library/Caches/<应用名称>`
   - `~/Library/Preferences/com.<应用名称>.plist`
   - `~/Library/Logs/<应用名称>`
   - `~/Library/Containers/<应用名称>`（部分应用）
   - `/Library/Application Support/<应用名称>`（部分应用）
3. 清空废纸篓。

### 使用终端删除残留文件
可以使用 `rm -rf` 命令删除相关目录，例如：
```bash
rm -rf ~/Library/Application\ Support/<应用名称>
rm -rf ~/Library/Caches/<应用名称>
rm -rf ~/Library/Preferences/com.<应用名称>.plist
```

> **⚠️ 注意**：`rm -rf` 命令无法撤销，请确认路径正确后再执行。

## 4. 使用第三方工具卸载（推荐）
如果不想手动删除残留文件，可以使用以下工具：

### 1. AppCleaner（免费）
1. 下载 AppCleaner（https://freemacsoft.net/appcleaner/）。
2. 打开 AppCleaner，将要卸载的应用拖入窗口。
3. 勾选所有相关文件，点击 **删除**。

### 2. CleanMyMac X（付费）
1. 下载并安装 CleanMyMac X（https://macpaw.com/cleanmymac）。
2. 打开软件，选择 **卸载**。
3. 找到要删除的应用，点击 **卸载**。

## 5. 结论
正确卸载 DMG 应用不仅可以**释放磁盘空间**，还可以**避免系统垃圾堆积**。如果你希望彻底删除应用，建议结合 **手动清理** 和 **AppCleaner 等工具**，确保没有残留文件。









