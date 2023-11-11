
![在这里插入图片描述](https://img-blog.csdnimg.cn/960ac83868c6459d82d5037d82c8fe15.png)




## 1. 安装 Homebrew
- 安装 [homebrew](https://blog.csdn.net/xixihahalelehehe/article/details/129151854)
```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```
Add Homebrew To Path
```bash
echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> /Users/[username]/.zprofile
eval "$(/opt/homebrew/bin/brew shellenv)"
```
- [macOS 如何 GitHub 访问加速指南](https://blog.csdn.net/xixihahalelehehe/article/details/129170643)
## 2. 安装 zsh

```bash
brew install zsh
```

这应该在您的机器上安装 shell。或者，如果您使用的是 Linux，请遵循本指南；如果您使用的是 Windows，请遵循本指南。

现在，要将 ZSH 设置为默认 shell，请使用以下命令

```bash
chsh -s /usr/local/bin/zsh
```

或者，对于较旧的 Mac OS High Sierra 和之前，您可能需要运行以下命令

```bash
chsh -s /bin/zsh
```

## 3. 安装 iTerm2
命令行安装

```bash
brew install --cask iterm2
```
或者去官方 [iterm2](https://iterm2.com/) 下载安装。

安装后切换终端打开 iterm2

## 4. 安装 Git

```bash
brew install git
```
## 5. 安装 Oh My Zsh
 [官方Oh My Zsh](https://ohmyz.sh/) 
 [github Oh My Zsh](https://github.com/ohmyzsh/ohmyzsh) 

```bash
 sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```
## 6. 安装 Oh My Zsh PowerLevel10K 主题
- [github Oh My Zsh 主题](https://github.com/ohmyzsh/ohmyzsh/tree/master/themes)
- [romkatv/powerlevel10k](https://github.com/romkatv/powerlevel10k)
```bash
git clone https://github.com/romkatv/powerlevel10k.git $ZSH_CUSTOM/themes/powerlevel10k
```
现在它已经安装好了，用你喜欢的编辑器打开“`~/.zshrc`”文件，然后按如下所示修改“`ZSH_THEME`”的值：

```bash
ZSH_THEME="powerlevel10k/powerlevel10k"
```
生效

```bash
source ~/.zshrc
```
## 7. 安装 Meslo Nerd Font
按“y”安装字体，然后退出 iTerm2。

## 8. 更新 VSCode 终端字体（可选）
打开 settings.json 并添加以下行：

```bash
"terminal.integrated.fontFamily": "MesloLGS NF"
```
## 9. 配置 PowerLevel10K
重启 iTerm2。您现在应该看到 `PowerLevel10K` 配置过程。如果不这样做，请运行以下命令：

```bash
p10k configure
```
按照 PowerLevel10K 配置的说明自定义终端
## 10. 增加终端字体大小
1. 打开 iTerm2 preferences
2. 设置Profiles > Text
3. 增加 font size 至 20px

## 11. 将 iTerm2 颜色更改为我的自定义主题
打开 iTerm2
通过运行以下命令下载我的颜色配置文件（将添加到下载文件夹）：

```bash
curl https://raw.githubusercontent.com/josean-dev/dev-environment-files/main/coolnight.itermcolors --output ~/Downloads/coolnight.itermcolors
```
1. 打开 iTerm2 首选项
2. 转到配置文件 > 颜色
3. 导入下载的颜色配置文件（coolnight）
4. 选择颜色配置文件 (coolnight)

您可以在这里找到其他主题：[Iterm2 配色方案](https://iterm2colorschemes.com/)

## 12. 安装 ZSH 插件
### 12.1 安装 `zsh-autosuggestions`
替代 `shell` 是`fish`，它带有一些用于编写终端命令的下一级自动建议。
```bash
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
```
### 12.2 安装 `zsh-syntax-highlighting`
语法高亮插件
```bash
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
```
在您想要的编辑器中打开“`~/.zshrc`”文件，并将插件行修改为您在下面看到的内容。

### 12.3 加载 wd
wd 插件是我比较喜欢的一个，它的作用就是能够快速的切换到常用的目录。
比如：
cd 到一个目录

```bash
cd  /usr/nginx/www/html
```

wd 做一个标识

```bash
wd nh
```

退回至～,执行`wd nh`就可以跳转至`/usr/nginx/www/html`

```bash
cd ..
wd nh
```

```bash
plugins=(git wd zsh-autosuggestions zsh-syntax-highlighting web-search)
```
通过运行加载这些新插件：

```bash
source ~/.zshrc
```

参考：
- [My Terminal Setup: iTerm2 + ZSH + Powerlevel10k](https://medium.com/weekly-webtips/my-terminal-setu-iterm2-zsh-powerlevel10k-f7101ffc72c2)
- [How To Setup Your Mac Terminal](https://www.josean.com/posts/terminal-setup)
- [iTerm Themes](https://iterm2colorschemes.com/)
- [https://brew.idayer.com/guide](https://brew.idayer.com/guide)
