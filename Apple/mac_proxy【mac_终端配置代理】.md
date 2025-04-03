![](https://i-blog.csdnimg.cn/blog_migrate/52d2ae4e3ab40c989ddecda059c0a430.jpeg#pic_center)



```bash
vi ~/.bash_profile
alias proxy='export http_proxy=127.0.0.1:1088;export https_proxy=$http_proxy'
alias proxyOff='unset http_proxy;unset https_proxy'
```
生效

```bash
source ~/.bash_profile
```
而从 macOS Catalina 版开始，Mac 将使用 zsh 作为默认的 Shell 终端。要对其进行配置，首先执行如下命令修改用户全局配置文件：
1
利用本地的代理口地址
```bash
vi ~/.zshrc
alias proxy='export all_proxy=socks5://127.0.0.1:10805'
alias unproxy='unset all_proxy'
```

```bash
source ~/.zshrc
```

或者
利用代理服务端地址
```bash
vi ~/.zshrc
alias proxy='export http_proxy=192.168.21.101:7890;export https_proxy=192.168.21.101:7890'
alias unproxy='unset http_proxy; unset https_proxy'
```

```bash
source ~/.zshrc
```

