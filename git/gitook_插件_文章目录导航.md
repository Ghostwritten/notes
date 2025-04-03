# 文章目录导航


---

## 1.  anchor-navigation-ex插件

[anchor-navigation-ex](https://www.npmjs.com/package/gitbook-plugin-anchor-navigation-ex)插件功能是：悬浮目录和回到顶部
安装：

```bash
npm i gitbook-plugin-anchor-navigation-ex
```

添加 `Toc` 到侧边悬浮导航以及回到顶部按钮。需要注意以下两点：

本插件只会提取 `h[1-3]` 标签作为悬浮导航

只有按照以下顺序嵌套才会被提取

> 必须要以 h1 开始，直接写 h2 不会被提取 `anchor-navigation-ex` 和 会互相叠加影响，应选择其中一种即可

在 `books.json` 添加配置

```bash
{
    plugins: ["anchor-navigation-ex"],
    pluginsConfig: [
        "anchor-navigation-ex": {
            "tocLevel1Icon": "fa fa-hand-o-right",
            "tocLevel2Icon": "fa fa-hand-o-right",
            "tocLevel3Icon": "fa fa-hand-o-right",
            "multipleH1": false,
            "multipleH2": false,
            "multipleH3": false,
            "multipleH4": false,
            "showLevelIcon": false,
            "showLevel": false
        },
}
```
在文章中使用1/2/3级标题

```bash
# h1
## h2
### h3
```
右上角目录图标预览效果
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/46efd14b791fe028e1e45686def4940e.png)

##  2. ancre-navigation 插件
[ancre-navigation](https://www.npmjs.com/package/gitbook-plugin-ancre-navigation) 插件功能是：右上角悬浮导航和回到顶部按钮

安装

```bash
npm i gitbook-plugin-anchor-navigation-ex
```

方法：

```bash
{
    plugins: ["ancre-navigation"]
}
```
效果图：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/73194f63c6c2d9c0b4683a60a23fe7ce.png)
##  3. back-to-top-button 插件
[back-to-top-button](https://www.npmjs.com/package/gitbook-plugin-back-to-top-button) 插件功能：当页面超过一屏幕时，会显示一个 回到顶部按钮
安装

```bash
npm i gitbook-plugin-anchor-navigation-ex
```

地址：

[gitbook-plugin-back-to-top-button](https://github.com/stuebersystems/gitbook-plugin-back-to-top-button)
book.json配置：

```bash
{
    "plugins": [
         "back-to-top-button"
    ]
}
```
效果图
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/56052da4cf52c1f5c610e908b23b923e.png)

## 4. page-toc-button 插件
[page-toc-button](https://www.npmjs.com/package/gitbook-plugin-page-toc-button) 插件功能：悬浮目录
安装

```bash
npm i gitbook-plugin-anchor-navigation-ex
```

方法

```bash
{
    "plugins" : [ "page-toc-button" ]
}
```
可选配置项：

```bash
"pluginsConfig": {
        "page-toc-button": {
            "maxTocDepth": 2,
            "minTocSize": 2
       }
}
```

 - `maxTocDept` 标题的最大深度（2 = h1 + h2 + h3）。不支持值> 2。 默认2
 - `minTocSize` 显示toc按钮的最小toc条目数。 默认 2

效果图：
页面toc按钮:
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d12511077c131279422d9b69b01e8219.png)
页面toc菜单:
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f2fd36ab5f8afd9f9ad245b7f263b507.png)



