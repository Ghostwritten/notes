#  评论


---
##  1. Disqus 插件
[Disqus](https://www.npmjs.com/package/gitbook-plugin-disqus) 是一个非常流行的为网站集成评论系统的工具，同样，gitbook 也可以集成 disqus 以便可以和读者交流。

首先，需要在 disqus 上注册一个账号，然后添加一个 website，这会获得一个关键字，然后在集成时配置这个关键字即可。

###  1.1 安装 

```bash
npm i gitbook-plugin-disqus
```
###   1.2 配置
book.json配置：

```bash
{
    "plugins": ["disqus"],
    "pluginsConfig": {
        "disqus": {
            "shortName": "introducetogitbook"
        }
    }  
}
```

> 注意：上面的 shortName 的值就是你在 disqus 上创建的 website 获得的唯一关键字。

### 1.3 效果
![在这里插入图片描述](https://img-blog.csdnimg.cn/8d699b3d4f6c4d34b79bf0ea887f115a.png)

### 1.4 评论
略感复杂，不够简约
综合指数：⭐️⭐️⭐️⭐️

##  2. gitalk 插件
[gitalk](https://github.com/draveness/gitbook-plugin-gitalk) 插件是实现博客评论功能

### 2.1 安装
### 2.2 配置
### 2.3 效果
### 2.4 评价

## 3 gtalk 插件
[gtalk](https://www.npmjs.com/package/gitbook-plugin-gtalk) 插件是实现博客评论功能

### 3.1 安装

```bash
npm i gitbook-plugin-gtalk
```

### 3.2 配置

```bash
{
  "plugins": ["gtalk"],
  "pluginsConfig": {
    "gtalk": {
      "clientID": "GitHub Application Client ID",
      "clientSecret": "GitHub Application Client Secret",
      "repo": "GitHub repo",
      "owner": "GitHub repo owner",
      "admin": ["GitHub repo owner and collaborators, only these guys can initialize github issues"]
    }
  }
}
```
[Github 如何获取 GitHub Client ID and Client Secret](https://blog.csdn.net/xixihahalelehehe/article/details/125294535)

###  3.3 效果
略


##  4. mygitalk 插件
[mygitalk](https://www.npmjs.com/package/gitbook-plugin-mygitalk) 插件是基于 gitalk改进的插件

###  4.1 安装

```bash
npm i gitbook-plugin-mygitalk
```
or

```bash
gitbook install
```

### 4.2 配置

```bash
{
  "pluginsConfig": {
    "mygitalk": {
        "clientID": "GitHub Application Client ID",
        "clientSecret": "GitHub Application Client Secret",
        "repo": "GitHub repo",
        "owner": "GitHub repo owner",
        "admin": ["GitHub repo owner and collaborators, only these guys can initialize github issues"],
        "distractionFreeMode": false
    }
  }
}
```

###  4.3 效果
（略）
