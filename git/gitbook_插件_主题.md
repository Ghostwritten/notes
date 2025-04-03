#  Gitbook 插件 主题


---

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/fc3162f4ca852a88a822af5c70543ae6.gif#pic_center)


##  1. theme-default 插件
[theme-default](https://www.npmjs.com/package/gitbook-plugin-theme-default) 插件是 默认主题,大多数插件针对的都是默认主题。

默认情况下,左侧菜单不显示层级属性,如果将 `showLevel` 属性设置为 true 可以显示层级数字.
###  1.1 安装

```bash
npm i gitbook-plugin-theme-default
```
###  1.2 配置

```bash

"pluginsConfig": {
    "theme-default": {
        "showLevel": true
    }
}

   "plugins": [
        "theme-default"
    ]
```

###  1.3 效果
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e5c35ca716a1daa39e6b6d85c466e50c.png)

![showLevel": true，可以显示层级数字](https://i-blog.csdnimg.cn/blog_migrate/aff76c511d7128e2b5b6d51ff425e0d7.png)

##  2. theme-comscore 插件
theme-default 插件默认主题是黑白的,而[theme-comscore](https://www.npmjs.com/package/gitbook-plugin-theme-comscore)插件 主题是彩色的,即标题和正文颜色有所区分.

### 2.1 安装

```bash
npm i gitbook-plugin-theme-comscore

npm i -g gitbook-cli // maybe need sudo permission
gitbook install
```


### 2.2 配置

```bash
{
"plugins": [
        "theme-comscore"
    ]
}
```

###  2.3 效果
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/bbfc41bbb51ba291189f92d19cbc61d4.png)

##  3. theme-api 插件
使用 GitBook 发布 API 文档的主题。

[theme-api](https://www.npmjs.com/package/gitbook-plugin-theme-api) 主题与搜索插件（如默认插件或algolia）完美配合。
### 3.1 安装

```bash
npm i gitbook-plugin-theme-api
```

### 3.2 配置

```bash
{
    "plugins": ["theme-api"],
    "pluginsConfig": {
        "theme-api": {
            "theme": "dark"
        }
    }
}
```
###  3.3 语法
该主题允许使用模板块语法轻松定义具有不同语言示例的方法。

一个方法块可以包含任意数量的嵌套`sample`和`common`块。

这些嵌套块记录在下面。

```bash
{% method %}
## Install {#install}

The first thing is to get the GitBook API client.

{% sample lang="js" %}
```bash
$ npm install gitbook-api
```

{% sample lang="go" %}
```bash
$ go get github.com/GitbookIO/go-gitbook-api
```
{% endmethod %}
```

在每个包含示例块的页面上，右上角method会自动添加一个切换器，以轻松选择要显示的语言。

可以在book.json文件中配置每种语言的名称，其lang属性对应于sample块lang参数：

```bash
{
  "plugins": ["theme-api"],
  "pluginsConfig": {
    "theme-api": {
      "languages": [
        {
          "lang": "js",          // sample lang argument
          "name": "JavaScript",  // corresponding name to be displayed
          "default": true        // default language to show
        },
        {
          "lang": "go",
          "name": "Go"
        }
      ]
    }
  }
}
```

### 3.4 效果
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/5b8f2769e7694c980a01b7b9b641c7b0.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/41181bb4f9bb54ecc38139ec65249597.png)
###  3.5 布局
该主题提供了两种布局来显示您的示例：一列或两列（拆分）。当"split": true时为拆分布局。
```bash
{
  "plugins": ["theme-api"],
  "pluginsConfig": {
    "theme-api": {
      "split": true
    }
  }
}
```
一栏布局
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/2a1a722977d87e7088c6cfe84d995c2e.png)
拆分布局
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/90f0b189d62470c7756fc4d1cad0a5df.png)

