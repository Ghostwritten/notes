#  侧边导航



---

## 1. 简介

Gitbook 提供的书籍默认都会在左侧提供书籍目录索引导航的功能。

对于侧边导航功能、UI、布局、效果等等有一些开源的插件可供选择，插件的核心功能与差异：

 - [chapter-fold](https://www.npmjs.com/package/gitbook-plugin-chapter-fold) 插件: 默认的侧边目录是全部展开的，该插件可以让文章按照层级目录折叠起来，同时只能展开一个目录。
 - [expandable-chapters](https://www.npmjs.com/package/gitbook-plugin-expandable-chapters) 插件: 默认的侧边目录是全部展开的，该插件可以让文章按照层级目录折叠起来，展开后不会自动折叠。
 - [expandable-chapters-small](https://www.npmjs.com/package/gitbook-plugin-expandable-chapters-small) 插件: 默认的侧边目录是全部展开的，该插件可以让文章按照层级目录折叠起来，展开后不会自动折叠，箭头相比 `expandable-chapters` 会细一些。
 - [sidebar-style](https://www.npmjs.com/package/gitbook-plugin-sidebar-style) 插件: 会替换掉 Published by GitBook，并在左侧最上面显示标题。
 - [splitter](https://www.npmjs.com/package/gitbook-plugin-splitter) 插件:提供侧边栏宽度可调节功能。

本站点使用：

 1. expandable-chapters-small 插件
 2. sidebar-style 插件
 3. splitter 插件

## 1. chapter-fold 插件
安装

```bash
npm i gitbook-plugin-chapter-fold
```

默认的侧边目录是全部展开的，该插件可以使左侧导航目录默认折叠。

在点击折叠目录时，只会展开一个目录，新的目录展开时其他目录会折叠。


```bash
{
    "plugins": ["chapter-fold"]
}
```
效果图：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c966d601ea201277411f367c1313c047.png)
## 2. expandable-chapters 插件
安装

```bash
npm i gitbook-plugin-expandable-chapters
```

默认的侧边目录是全部展开的，该插件可以使左侧导航目录默认折叠。

可以支持点击展开后的目录一直保持展开的状态，这点和 chapter-fold 不同。

和 `expandable-chapters-small` 效果相同，唯一不同的是这个插件的箭头粗一些。

```bash
{
    "plugins": [
         "expandable-chapters"
    ]
}
```
效果图：

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/39947d2c5e31f886a40407afcaf7edb5.png)

## 3. expandable-chapters-small 插件
安装

```bash
npm i gitbook-plugin-expandable-chapters-small
```

默认的侧边目录是全部展开的，该插件可以使左侧导航目录默认折叠。

可以支持点击展开后的目录一直保持展开的状态，这点和 chapter-fold 不同。

和 expandable-chapters 插件 效果相同，差异是折叠展开箭头图标小一些。

```bash
{
    "plugins": [
         "expandable-chapters-small"
    ]
}
```
效果图：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/13bed2e8c0efa83a6a82253b4e3a64a7.png)
## 4. sidebar-style 插件
安装
```bash
npm i gitbook-plugin-sidebar-style
```

在左侧最上方添加标题。

在左侧导航最下方替换掉 `Published by GitBook` 提示信息

```bash
{
    "plugins": ["sidebar-style"],
    "pluginsConfig": {
        "sidebar-style": {
            "title": "《Git Demo》",
            "author": "zongxun"
        }
    }
}
```
效果图：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/751b029b4bcd55f31c6e742ae9122803.png)

## 5. splitter 插件
安装
```bash
npm i gitbook-plugin-splitter
```

侧边栏宽度可调节
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ac30a98143d4bda9d5cd6636ecdf9eaf.gif#pic_center)


