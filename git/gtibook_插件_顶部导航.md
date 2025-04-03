#  Gtibook 插件 顶部导航


---

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f06ae2dec0b7c4f601630ab01ed1b243.gif#pic_center)


## 1. NavBar 插件
### 1.1 安装

从源码GitHub地址中下载，放到 `node_modules` 文件夹里（GitHub地址在进入插件地址右侧的GitHub链接）.

```bash
$ cd node_modules
$ git clone https://github.com/jchamath/gitbook-plugin-navbar.git
```

### 1.2 配置

```bash
{
    "plugins": ["navbar"]
}

"pluginsConfig": {
  "navbar": {
    "logo": {
      "link": "http://example.com/",
      "image": "http://example.com/resources/static/logo.png",
      "alt": "Example"
    },
    "menu": {
      "items": [{
        "text": "Home",
        "link": "http://example.com"
      }, {
        "text": "Page",
        "link": "http://example.com/page/"
      }],
      "style": {
        "color": "black",
        "hover": "#3579e2"
      }
    },
    "style": {
      "backgroundColor": "white",
      "linkColor": "black",
      "linkHoverColor": "blue"
    }
  },
}
```
###  1.3  效果
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/bc442a369ce852134d215b8de0513803.png)
### 1.4 评价
简约，但嵌入页面有些呆板，与不能隐藏，没有下拉菜单功能。

综合指数：:star::star::star:

##  2. theme-door 插件
[theme-door](https://www.npmjs.com/package/gitbook-plugin-theme-door) 插件含有Header导航，Sidebar导航以及MiniMap导航的gitbook主题。
### 2.1 安装

```bash
npm i gitbook-plugin-theme-door
```

### 2.2 配置

```bash
{
  "plugins": [
    "splitter",
    "hide-element",
    "code",
    "theme-door",
    "-lunr",
    "-search",
    "-sharing",
    "search-pro-fixed"
  ],
  "variables": {
    "doorTheme": {
      "nav": [
        {
          "url": "xxxxx",
          "target": "_blank",
          "name": "论坛"
        },
        {
          "url": "xxxxx",
          "target": "_blank",
          "name": "登录"
        }
      ]
    }
  },
  "pluginsConfig": {
    "hide-element": {
      "elements": [".gitbook-link"]
    },
    "doorTheme": {
      "search-placeholder": "请输入关键字搜索",
      "logo": "./_media/logo.svg",
      "favicon": "./_media/favicon.ico"
    }
  }
}
```

###  2.3 效果
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d21b8f88e5484af409e186247a58555a.png)

### 2.4 评价
可在此基础二次自定义开发。目前默认主题不太友好。不能隐藏，没有下拉菜单功能。
综合指数：:star::star::star:
