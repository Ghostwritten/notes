
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/17567383d8c4d56b57e542697f4e77e9.jpeg#pic_center)



## 1. 简介
omebrew是一款包管理工具，目前支持macOS和linux系统。主要有四个部分组成: brew、homebrew-core 、homebrew-cask、homebrew-bottles。

|  名称| 说明 |
|--|--|
|brew	|Homebrew 源代码仓库|
|homebrew-core	|Homebrew 核心源|
|homebrew-cask	|提供 macOS 应用和大型二进制文件的安装|
|homebrew-bottles	|预编译二进制软件包## 官方安装|


## 2. 预备

- [安装并配置 git](https://ghostwritten.blog.csdn.net/article/details/105240194) 

> 注意：不仅仅是安装git ，还需要配置 git config 用户与邮箱、免密等等。否则可能报错如下：
> [error: RPC failed; curl 92 HTTP/2 stream 5 was not closed cleanly before end of the underlying stream](https://github.com/orgs/Homebrew/discussions/4069)

## 3. 安装

### 3.1 官方安装
注意： 按照官方文档安装如果在东方大陆，需要“出海行动“策略：[mac Terminal config proxy](https://ghostwritten.blog.csdn.net/article/details/134631837)

- 参考地址：[https://brew.sh/](https://brew.sh/)
```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```
输出：

```bash
$ /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
==> Checking for `sudo` access (which may request your password)...
Password:
==> This script will install:
/opt/homebrew/bin/brew
/opt/homebrew/share/doc/homebrew
/opt/homebrew/share/man/man1/brew.1
/opt/homebrew/share/zsh/site-functions/_brew
/opt/homebrew/etc/bash_completion.d/brew
/opt/homebrew

Press RETURN/ENTER to continue or any other key to abort:
==> /usr/bin/sudo /usr/sbin/chown -R zongxun:admin /opt/homebrew
==> Downloading and installing Homebrew...
remote: Enumerating objects: 251262, done.
remote: Counting objects: 100% (1043/1043), done.
remote: Compressing objects: 100% (666/666), done.
remote: Total 251262 (delta 379), reused 957 (delta 331), pack-reused 250219
Receiving objects: 100% (251262/251262), 76.11 MiB | 714.00 KiB/s, done.
Resolving deltas: 100% (182996/182996), done.
From https://github.com/Homebrew/brew
 * [new branch]          master     -> origin/master
 * [new tag]             0.1        -> 0.1
 * [new tag]             0.2        -> 0.2
 * [new tag]             0.3        -> 0.3
 * [new tag]             0.4        -> 0.4
 * [new tag]             0.5        -> 0.5
 * [new tag]             0.6        -> 0.6
 * [new tag]             0.7        -> 0.7
 * [new tag]             0.7.1      -> 0.7.1
 * [new tag]             0.8        -> 0.8
 * [new tag]             0.8.1      -> 0.8.1
 * [new tag]             0.9        -> 0.9
 * [new tag]             0.9.1      -> 0.9.1
 * [new tag]             0.9.2      -> 0.9.2
 * [new tag]             0.9.3      -> 0.9.3
 * [new tag]             0.9.4      -> 0.9.4
 * [new tag]             0.9.5      -> 0.9.5
 * [new tag]             0.9.8      -> 0.9.8
 * [new tag]             0.9.9      -> 0.9.9
 * [new tag]             1.0.0      -> 1.0.0
 * [new tag]             1.0.1      -> 1.0.1
 * [new tag]             1.0.2      -> 1.0.2
 * [new tag]             1.0.3      -> 1.0.3
 * [new tag]             1.0.4      -> 1.0.4
 * [new tag]             1.0.5      -> 1.0.5
 * [new tag]             1.0.6      -> 1.0.6
 * [new tag]             1.0.7      -> 1.0.7
 * [new tag]             1.0.8      -> 1.0.8
 * [new tag]             1.0.9      -> 1.0.9
 * [new tag]             1.1.0      -> 1.1.0
 * [new tag]             1.1.1      -> 1.1.1
 * [new tag]             1.1.10     -> 1.1.10
 * [new tag]             1.1.11     -> 1.1.11
 * [new tag]             1.1.12     -> 1.1.12
 * [new tag]             1.1.13     -> 1.1.13
 * [new tag]             1.1.2      -> 1.1.2
 * [new tag]             1.1.3      -> 1.1.3
 * [new tag]             1.1.4      -> 1.1.4
 * [new tag]             1.1.5      -> 1.1.5
 * [new tag]             1.1.6      -> 1.1.6
 * [new tag]             1.1.7      -> 1.1.7
 * [new tag]             1.1.8      -> 1.1.8
 * [new tag]             1.1.9      -> 1.1.9
 * [new tag]             1.2.0      -> 1.2.0
 * [new tag]             1.2.1      -> 1.2.1
 * [new tag]             1.2.2      -> 1.2.2
 * [new tag]             1.2.3      -> 1.2.3
 * [new tag]             1.2.4      -> 1.2.4
 * [new tag]             1.2.5      -> 1.2.5
 * [new tag]             1.2.6      -> 1.2.6
 * [new tag]             1.3.0      -> 1.3.0
 * [new tag]             1.3.1      -> 1.3.1
 * [new tag]             1.3.2      -> 1.3.2
 * [new tag]             1.3.3      -> 1.3.3
 * [new tag]             1.3.4      -> 1.3.4
 * [new tag]             1.3.5      -> 1.3.5
 * [new tag]             1.3.6      -> 1.3.6
 * [new tag]             1.3.7      -> 1.3.7
 * [new tag]             1.3.8      -> 1.3.8
 * [new tag]             1.3.9      -> 1.3.9
 * [new tag]             1.4.0      -> 1.4.0
 * [new tag]             1.4.1      -> 1.4.1
 * [new tag]             1.4.2      -> 1.4.2
 * [new tag]             1.4.3      -> 1.4.3
 * [new tag]             1.5.0      -> 1.5.0
 * [new tag]             1.5.1      -> 1.5.1
 * [new tag]             1.5.10     -> 1.5.10
 * [new tag]             1.5.11     -> 1.5.11
 * [new tag]             1.5.12     -> 1.5.12
 * [new tag]             1.5.13     -> 1.5.13
 * [new tag]             1.5.14     -> 1.5.14
 * [new tag]             1.5.2      -> 1.5.2
 * [new tag]             1.5.3      -> 1.5.3
 * [new tag]             1.5.4      -> 1.5.4
 * [new tag]             1.5.5      -> 1.5.5
 * [new tag]             1.5.6      -> 1.5.6
 * [new tag]             1.5.7      -> 1.5.7
 * [new tag]             1.5.8      -> 1.5.8
 * [new tag]             1.5.9      -> 1.5.9
 * [new tag]             1.6.0      -> 1.6.0
 * [new tag]             1.6.1      -> 1.6.1
 * [new tag]             1.6.10     -> 1.6.10
 * [new tag]             1.6.11     -> 1.6.11
 * [new tag]             1.6.12     -> 1.6.12
 * [new tag]             1.6.13     -> 1.6.13
 * [new tag]             1.6.14     -> 1.6.14
 * [new tag]             1.6.15     -> 1.6.15
 * [new tag]             1.6.16     -> 1.6.16
 * [new tag]             1.6.17     -> 1.6.17
 * [new tag]             1.6.2      -> 1.6.2
 * [new tag]             1.6.3      -> 1.6.3
 * [new tag]             1.6.4      -> 1.6.4
 * [new tag]             1.6.5      -> 1.6.5
 * [new tag]             1.6.6      -> 1.6.6
 * [new tag]             1.6.7      -> 1.6.7
 * [new tag]             1.6.8      -> 1.6.8
 * [new tag]             1.6.9      -> 1.6.9
 * [new tag]             1.7.0      -> 1.7.0
 * [new tag]             1.7.1      -> 1.7.1
 * [new tag]             1.7.2      -> 1.7.2
 * [new tag]             1.7.3      -> 1.7.3
 * [new tag]             1.7.4      -> 1.7.4
 * [new tag]             1.7.5      -> 1.7.5
 * [new tag]             1.7.6      -> 1.7.6
 * [new tag]             1.7.7      -> 1.7.7
 * [new tag]             1.8.0      -> 1.8.0
 * [new tag]             1.8.1      -> 1.8.1
 * [new tag]             1.8.2      -> 1.8.2
 * [new tag]             1.8.3      -> 1.8.3
 * [new tag]             1.8.4      -> 1.8.4
 * [new tag]             1.8.5      -> 1.8.5
 * [new tag]             1.8.6      -> 1.8.6
 * [new tag]             1.9.0      -> 1.9.0
 * [new tag]             1.9.1      -> 1.9.1
 * [new tag]             1.9.2      -> 1.9.2
 * [new tag]             1.9.3      -> 1.9.3
 * [new tag]             2.0.0      -> 2.0.0
 * [new tag]             2.0.1      -> 2.0.1
 * [new tag]             2.0.2      -> 2.0.2
 * [new tag]             2.0.3      -> 2.0.3
 * [new tag]             2.0.4      -> 2.0.4
 * [new tag]             2.0.5      -> 2.0.5
 * [new tag]             2.0.6      -> 2.0.6
 * [new tag]             2.1.0      -> 2.1.0
 * [new tag]             2.1.1      -> 2.1.1
 * [new tag]             2.1.10     -> 2.1.10
 * [new tag]             2.1.11     -> 2.1.11
 * [new tag]             2.1.12     -> 2.1.12
 * [new tag]             2.1.13     -> 2.1.13
 * [new tag]             2.1.14     -> 2.1.14
 * [new tag]             2.1.15     -> 2.1.15
 * [new tag]             2.1.16     -> 2.1.16
 * [new tag]             2.1.2      -> 2.1.2
 * [new tag]             2.1.3      -> 2.1.3
 * [new tag]             2.1.4      -> 2.1.4
 * [new tag]             2.1.5      -> 2.1.5
 * [new tag]             2.1.6      -> 2.1.6
 * [new tag]             2.1.7      -> 2.1.7
 * [new tag]             2.1.8      -> 2.1.8
 * [new tag]             2.1.9      -> 2.1.9
 * [new tag]             2.2.0      -> 2.2.0
 * [new tag]             2.2.1      -> 2.2.1
 * [new tag]             2.2.10     -> 2.2.10
 * [new tag]             2.2.11     -> 2.2.11
 * [new tag]             2.2.12     -> 2.2.12
 * [new tag]             2.2.13     -> 2.2.13
 * [new tag]             2.2.14     -> 2.2.14
 * [new tag]             2.2.15     -> 2.2.15
 * [new tag]             2.2.16     -> 2.2.16
 * [new tag]             2.2.17     -> 2.2.17
 * [new tag]             2.2.2      -> 2.2.2
 * [new tag]             2.2.3      -> 2.2.3
 * [new tag]             2.2.4      -> 2.2.4
 * [new tag]             2.2.5      -> 2.2.5
 * [new tag]             2.2.6      -> 2.2.6
 * [new tag]             2.2.7      -> 2.2.7
 * [new tag]             2.2.8      -> 2.2.8
 * [new tag]             2.2.9      -> 2.2.9
 * [new tag]             2.3.0      -> 2.3.0
 * [new tag]             2.4.0      -> 2.4.0
 * [new tag]             2.4.1      -> 2.4.1
 * [new tag]             2.4.10     -> 2.4.10
 * [new tag]             2.4.11     -> 2.4.11
 * [new tag]             2.4.12     -> 2.4.12
 * [new tag]             2.4.13     -> 2.4.13
 * [new tag]             2.4.14     -> 2.4.14
 * [new tag]             2.4.15     -> 2.4.15
 * [new tag]             2.4.16     -> 2.4.16
 * [new tag]             2.4.2      -> 2.4.2
 * [new tag]             2.4.3      -> 2.4.3
 * [new tag]             2.4.4      -> 2.4.4
 * [new tag]             2.4.5      -> 2.4.5
 * [new tag]             2.4.6      -> 2.4.6
 * [new tag]             2.4.7      -> 2.4.7
 * [new tag]             2.4.8      -> 2.4.8
 * [new tag]             2.4.9      -> 2.4.9
 * [new tag]             2.5.0      -> 2.5.0
 * [new tag]             2.5.1      -> 2.5.1
 * [new tag]             2.5.10     -> 2.5.10
 * [new tag]             2.5.11     -> 2.5.11
 * [new tag]             2.5.12     -> 2.5.12
 * [new tag]             2.5.2      -> 2.5.2
 * [new tag]             2.5.3      -> 2.5.3
 * [new tag]             2.5.4      -> 2.5.4
 * [new tag]             2.5.5      -> 2.5.5
 * [new tag]             2.5.6      -> 2.5.6
 * [new tag]             2.5.7      -> 2.5.7
 * [new tag]             2.5.8      -> 2.5.8
 * [new tag]             2.5.9      -> 2.5.9
 * [new tag]             2.6.0      -> 2.6.0
 * [new tag]             2.6.1      -> 2.6.1
 * [new tag]             2.6.2      -> 2.6.2
 * [new tag]             2.7.0      -> 2.7.0
 * [new tag]             2.7.1      -> 2.7.1
 * [new tag]             2.7.2      -> 2.7.2
 * [new tag]             2.7.3      -> 2.7.3
 * [new tag]             2.7.4      -> 2.7.4
 * [new tag]             2.7.5      -> 2.7.5
 * [new tag]             2.7.6      -> 2.7.6
 * [new tag]             2.7.7      -> 2.7.7
 * [new tag]             3.0.0      -> 3.0.0
 * [new tag]             3.0.1      -> 3.0.1
 * [new tag]             3.0.10     -> 3.0.10
 * [new tag]             3.0.11     -> 3.0.11
 * [new tag]             3.0.2      -> 3.0.2
 * [new tag]             3.0.3      -> 3.0.3
 * [new tag]             3.0.4      -> 3.0.4
 * [new tag]             3.0.5      -> 3.0.5
 * [new tag]             3.0.6      -> 3.0.6
 * [new tag]             3.0.7      -> 3.0.7
 * [new tag]             3.0.8      -> 3.0.8
 * [new tag]             3.0.9      -> 3.0.9
 * [new tag]             3.1.0      -> 3.1.0
 * [new tag]             3.1.1      -> 3.1.1
 * [new tag]             3.1.10     -> 3.1.10
 * [new tag]             3.1.11     -> 3.1.11
 * [new tag]             3.1.12     -> 3.1.12
 * [new tag]             3.1.2      -> 3.1.2
 * [new tag]             3.1.3      -> 3.1.3
 * [new tag]             3.1.4      -> 3.1.4
 * [new tag]             3.1.5      -> 3.1.5
 * [new tag]             3.1.6      -> 3.1.6
 * [new tag]             3.1.7      -> 3.1.7
 * [new tag]             3.1.8      -> 3.1.8
 * [new tag]             3.1.9      -> 3.1.9
 * [new tag]             3.2.0      -> 3.2.0
 * [new tag]             3.2.1      -> 3.2.1
 * [new tag]             3.2.10     -> 3.2.10
 * [new tag]             3.2.11     -> 3.2.11
 * [new tag]             3.2.12     -> 3.2.12
 * [new tag]             3.2.13     -> 3.2.13
 * [new tag]             3.2.14     -> 3.2.14
 * [new tag]             3.2.15     -> 3.2.15
 * [new tag]             3.2.16     -> 3.2.16
 * [new tag]             3.2.17     -> 3.2.17
 * [new tag]             3.2.2      -> 3.2.2
 * [new tag]             3.2.3      -> 3.2.3
 * [new tag]             3.2.4      -> 3.2.4
 * [new tag]             3.2.5      -> 3.2.5
 * [new tag]             3.2.6      -> 3.2.6
 * [new tag]             3.2.7      -> 3.2.7
 * [new tag]             3.2.8      -> 3.2.8
 * [new tag]             3.2.9      -> 3.2.9
 * [new tag]             3.3.0      -> 3.3.0
 * [new tag]             3.3.1      -> 3.3.1
 * [new tag]             3.3.10     -> 3.3.10
 * [new tag]             3.3.11     -> 3.3.11
 * [new tag]             3.3.12     -> 3.3.12
 * [new tag]             3.3.13     -> 3.3.13
 * [new tag]             3.3.14     -> 3.3.14
 * [new tag]             3.3.15     -> 3.3.15
 * [new tag]             3.3.16     -> 3.3.16
 * [new tag]             3.3.2      -> 3.3.2
 * [new tag]             3.3.3      -> 3.3.3
 * [new tag]             3.3.4      -> 3.3.4
 * [new tag]             3.3.5      -> 3.3.5
 * [new tag]             3.3.6      -> 3.3.6
 * [new tag]             3.3.7      -> 3.3.7
 * [new tag]             3.3.8      -> 3.3.8
 * [new tag]             3.3.9      -> 3.3.9
 * [new tag]             3.4.0      -> 3.4.0
 * [new tag]             3.4.1      -> 3.4.1
 * [new tag]             3.4.10     -> 3.4.10
 * [new tag]             3.4.11     -> 3.4.11
 * [new tag]             3.4.2      -> 3.4.2
 * [new tag]             3.4.3      -> 3.4.3
 * [new tag]             3.4.4      -> 3.4.4
 * [new tag]             3.4.5      -> 3.4.5
 * [new tag]             3.4.6      -> 3.4.6
 * [new tag]             3.4.7      -> 3.4.7
 * [new tag]             3.4.8      -> 3.4.8
 * [new tag]             3.4.9      -> 3.4.9
 * [new tag]             3.5.0      -> 3.5.0
 * [new tag]             3.5.1      -> 3.5.1
 * [new tag]             3.5.10     -> 3.5.10
 * [new tag]             3.5.2      -> 3.5.2
 * [new tag]             3.5.3      -> 3.5.3
 * [new tag]             3.5.4      -> 3.5.4
 * [new tag]             3.5.5      -> 3.5.5
 * [new tag]             3.5.6      -> 3.5.6
 * [new tag]             3.5.7      -> 3.5.7
 * [new tag]             3.5.8      -> 3.5.8
 * [new tag]             3.5.9      -> 3.5.9
 * [new tag]             3.6.0      -> 3.6.0
 * [new tag]             3.6.1      -> 3.6.1
 * [new tag]             3.6.10     -> 3.6.10
 * [new tag]             3.6.11     -> 3.6.11
 * [new tag]             3.6.12     -> 3.6.12
 * [new tag]             3.6.13     -> 3.6.13
 * [new tag]             3.6.14     -> 3.6.14
 * [new tag]             3.6.15     -> 3.6.15
 * [new tag]             3.6.16     -> 3.6.16
 * [new tag]             3.6.17     -> 3.6.17
 * [new tag]             3.6.18     -> 3.6.18
 * [new tag]             3.6.19     -> 3.6.19
 * [new tag]             3.6.2      -> 3.6.2
 * [new tag]             3.6.20     -> 3.6.20
 * [new tag]             3.6.21     -> 3.6.21
 * [new tag]             3.6.3      -> 3.6.3
 * [new tag]             3.6.4      -> 3.6.4
 * [new tag]             3.6.5      -> 3.6.5
 * [new tag]             3.6.6      -> 3.6.6
 * [new tag]             3.6.7      -> 3.6.7
 * [new tag]             3.6.8      -> 3.6.8
 * [new tag]             3.6.9      -> 3.6.9
 * [new tag]             4.0.0      -> 4.0.0
 * [new tag]             4.0.1      -> 4.0.1
 * [new tag]             4.0.10     -> 4.0.10
 * [new tag]             4.0.11     -> 4.0.11
 * [new tag]             4.0.12     -> 4.0.12
 * [new tag]             4.0.13     -> 4.0.13
 * [new tag]             4.0.14     -> 4.0.14
 * [new tag]             4.0.15     -> 4.0.15
 * [new tag]             4.0.16     -> 4.0.16
 * [new tag]             4.0.17     -> 4.0.17
 * [new tag]             4.0.18     -> 4.0.18
 * [new tag]             4.0.19     -> 4.0.19
 * [new tag]             4.0.2      -> 4.0.2
 * [new tag]             4.0.20     -> 4.0.20
 * [new tag]             4.0.21     -> 4.0.21
 * [new tag]             4.0.22     -> 4.0.22
 * [new tag]             4.0.23     -> 4.0.23
 * [new tag]             4.0.24     -> 4.0.24
 * [new tag]             4.0.25     -> 4.0.25
 * [new tag]             4.0.26     -> 4.0.26
 * [new tag]             4.0.27     -> 4.0.27
 * [new tag]             4.0.28     -> 4.0.28
 * [new tag]             4.0.3      -> 4.0.3
 * [new tag]             4.0.4      -> 4.0.4
 * [new tag]             4.0.5      -> 4.0.5
 * [new tag]             4.0.6      -> 4.0.6
 * [new tag]             4.0.7      -> 4.0.7
 * [new tag]             4.0.8      -> 4.0.8
 * [new tag]             4.0.9      -> 4.0.9
 * [new tag]             4.1.0      -> 4.1.0
 * [new tag]             4.1.1      -> 4.1.1
 * [new tag]             4.1.10     -> 4.1.10
 * [new tag]             4.1.11     -> 4.1.11
 * [new tag]             4.1.12     -> 4.1.12
 * [new tag]             4.1.13     -> 4.1.13
 * [new tag]             4.1.14     -> 4.1.14
 * [new tag]             4.1.15     -> 4.1.15
 * [new tag]             4.1.16     -> 4.1.16
 * [new tag]             4.1.17     -> 4.1.17
 * [new tag]             4.1.18     -> 4.1.18
 * [new tag]             4.1.19     -> 4.1.19
 * [new tag]             4.1.2      -> 4.1.2
 * [new tag]             4.1.20     -> 4.1.20
 * [new tag]             4.1.21     -> 4.1.21
 * [new tag]             4.1.22     -> 4.1.22
 * [new tag]             4.1.23     -> 4.1.23
 * [new tag]             4.1.24     -> 4.1.24
 * [new tag]             4.1.25     -> 4.1.25
 * [new tag]             4.1.3      -> 4.1.3
 * [new tag]             4.1.4      -> 4.1.4
 * [new tag]             4.1.5      -> 4.1.5
 * [new tag]             4.1.6      -> 4.1.6
 * [new tag]             4.1.7      -> 4.1.7
 * [new tag]             4.1.8      -> 4.1.8
remote: Enumerating objects: 18, done.
remote: Counting objects: 100% (11/11), done.
remote: Total 18 (delta 11), reused 11 (delta 11), pack-reused 7
Unpacking objects: 100% (18/18), 3.38 KiB | 314.00 KiB/s, done.
From https://github.com/Homebrew/brew
 * [new tag]             4.0.29     -> 4.0.29
 * [new tag]             4.1.9      -> 4.1.9
HEAD is now at 29d1be9e9 Merge pull request #16335 from samford/separate-stable-version-audit
fatal: cannot force update the branch 'master' checked out at '/opt/homebrew'
==> Downloading https://ghcr.io/v2/homebrew/portable-ruby/portable-ruby/blobs/sha256:d783cbeb6e6ef0d71c0b442317b54554370decd6fac66bf2d4938c07a63f67be
######################################################################### 100.0%
==> Pouring portable-ruby-3.1.4.arm64_big_sur.bottle.tar.gz
Warning: /opt/homebrew/bin is not in your PATH.
  Instructions on how to configure your shell for Homebrew
  can be found in the 'Next steps' section below.
==> Installation successful!

==> Homebrew has enabled anonymous aggregate formulae and cask analytics.
Read the analytics documentation (and how to opt-out) here:
  https://docs.brew.sh/Analytics
No analytics data has been sent yet (nor will any be during this install run).

==> Homebrew is run entirely by unpaid volunteers. Please consider donating:
  https://github.com/Homebrew/brew#donations

==> Next steps:
- Run these two commands in your terminal to add Homebrew to your PATH:
    (echo; echo 'eval "$(/opt/homebrew/bin/brew shellenv)"') >> /Users/zongxun/.zprofile
    eval "$(/opt/homebrew/bin/brew shellenv)"
- Run brew help to get started
- Further documentation:
    https://docs.brew.sh

zongxun@zongxundeMacBook-Pro ~ % brew --version
zsh: command not found: brew
zongxun@zongxundeMacBook-Pro ~ % brew version
zsh: command not found: brew
zongxun@zongxundeMacBook-Pro ~ % echo $SHELL
/bin/zsh
zongxun@zongxundeMacBook-Pro ~ % echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.zprofile
zongxun@zongxundeMacBook-Pro ~ % eval "$(/opt/homebrew/bin/brew shellenv)"
zongxun@zongxundeMacBook-Pro ~ % brew --version
Homebrew 4.1.25
```

```bash
echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.bash_profile
eval "$(/opt/homebrew/bin/brew shellenv)"
```

输出

### 3.2 安装 ARM 版 Homebrew
ARM版Homebrew需要安装在/opt/homebrew路径下，早期的时候需要手动创建目录执行命令，目前使用最新脚本不需要手动操作。

直接执行：

```bash
/bin/bash -c "$(curl -fsSL https://gitee.com/ineo6/homebrew-install/raw/master/install.sh)"
```
然后还需设置环境变量，具体操作步骤如下，一定要仔细阅读。

PS: 终端类型根据执行命令`echo $SHELL`显示的结果：

```bash
/bin/bash => bash => .bash_profile
/bin/zsh => zsh => .zprofile
```

如果遇到环境变量无效问题，建议回过头来查看终端类型，再做正确的设置。

从`macOS Catalina(10.15.x)` 版开始，Mac使用`zsh`作为默认Shell，使用`.zprofile`，所以对应命令：

```bash
echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.zprofile
eval "$(/opt/homebrew/bin/brew shellenv)"
```

如果是`macOS Mojave` 及更低版本，并且没有自己配置过zsh，使用`.bash_profile`：

```bash
echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.bash_profile
eval "$(/opt/homebrew/bin/brew shellenv)"
```

### 3.3 安装 X86 版 Homebrew
为目前很多软件包没有支持ARM架构，我们也可以考虑使用x86版的Homebrew。

在命令前面添加`arch -x86_64`，就可以按X86模式执行该命令，比如：

```bash
arch -x86_64 /bin/bash -c "$(curl -fsSL https://gitee.com/ineo6/homebrew-install/raw/master/install.sh)"
```

### 3.4 多版本共存
如果你同时安装了`ARM`和`X86`两个版本，那你需要设置别名，把命令区分开。

同样是`.zprofile`或者`.bash_profile`里面添加：

至于操作哪个文件，请参考前文关于终端类型的描述，下文如有类似文字，保持一样的操作。

```bash
alias abrew='arch -arm64 /opt/homebrew/bin/brew'
alias ibrew='arch -x86_64 /usr/local/bin/brew'
```

abrew、ibrew可以根据你的喜好自定义。

然后再执行`source ~/.zprofile`或`source ~/.bash_profile`命令更新文件。


## 4. 配置源
`brew`、`homebrew/core`是必备项目，`homebrew/cask`、`homebrew/bottles`按需设置。

通过 `brew config` 命令可以查看相关配置信息。

### 4.1 初次安装brew后配置中科大 zsh

```bash
# 1.执行安装脚本
export HOMEBREW_BREW_GIT_REMOTE="https://mirrors.ustc.edu.cn/brew.git"
export HOMEBREW_CORE_GIT_REMOTE="https://mirrors.ustc.edu.cn/homebrew-core.git"
/bin/bash -c "$(curl -fsSL https://gitee.com/ineo6/homebrew-install/raw/master/install.sh)"

# 2.安装完成后设置
echo 'export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.ustc.edu.cn/homebrew-bottles/bottles' >> ~/.zprofile
source ~/.zprofile
```

### 4.2 换源配置中科大 zsh

```bash
# 脚本
git -C "$(brew --repo)" remote set-url origin https://mirrors.ustc.edu.cn/brew.git
git -C "$(brew --repo homebrew/core)" remote set-url origin https://mirrors.ustc.edu.cn/homebrew-core.git
git -C "$(brew --repo homebrew/cask)" remote set-url origin https://mirrors.ustc.edu.cn/homebrew-cask.git
brew update

echo 'export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.ustc.edu.cn/homebrew-bottles/bottles' >> ~/.zprofile
source ~/.zprofile
```

### 4.3 换源清华大学 zsh

```bash
# 脚本
git -C "$(brew --repo)" remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/brew.git
git -C "$(brew --repo homebrew/core)" remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-core.git
git -C "$(brew --repo homebrew/cask)" remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-cask.git
brew update

echo 'export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles/bottles' >> ~/.zprofile
source ~/.zprofile
```
更多[配置镜像细节请参考](https://brew.idayer.com/guide/change-source)。


### 4.4 更新

```bash
brew update-reset 
or
brew update -v
```

## 5. 命令

```bash
#安装包
 brew install 包名
 brew install node

#卸载包
brew uninstall 包名
brew uninstall node

#查询可用包
brew search 包名


# 更新全部包
brew upgrade

# 更新指定包
brew upgrade 包名

#查看已安装包列表
brew list

#查看任意包信息
brew info 包名

# 查看哪些软件包要被清除
brew cleanup -n

# 清除指定软件包的所有老版本
brew cleanup 软件名

# 清除所有软件包的所有老版本
brew cleanup

#查看版本
brew -v

#升级 Homebrew 自身
brew update

```

帮助输出

```bash
$ brew -h
Example usage:
  brew search TEXT|/REGEX/
  brew info [FORMULA|CASK...]
  brew install FORMULA|CASK...
  brew update
  brew upgrade [FORMULA|CASK...]
  brew uninstall FORMULA|CASK...
  brew list [FORMULA|CASK...]

Troubleshooting:
  brew config
  brew doctor
  brew install --verbose --debug FORMULA|CASK

Contributing:
  brew create URL [--no-fetch]
  brew edit [FORMULA|CASK...]

Further help:
  brew commands
  brew help [COMMAND]
  man brew
  https://docs.brew.sh
```

## 6. 问题
`raw.githubusercontent.com`地址不稳定，导致无法访问官方安装脚本`install.sh`。

```bash
curl: (7) Failed to connect to raw.githubusercontent.com port 443: Operation timed out
```

解决方案就是托管到`jsdelivr`，通过`CDN`加速访问。

另外也可以采用写入`hosts`的方式，可以一定程度解决GitHub资源无法访问的问题。

设置方案请阅读 [GitHub 访问加速指南](https://mp.weixin.qq.com/s/gFNP2Pk81vg7nE1XsDingg)


参考：
- [https://brew.idayer.com/guide](https://brew.idayer.com/guide)
- [https://brew.sh/](https://brew.sh/)

