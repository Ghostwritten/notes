# 分享


---
##  1. sharing-plus 插件
[sharing-plus](https://www.npmjs.com/package/gitbook-plugin-sharing-plus) 插件：分享当前页面，比默认的 `sharing` 插件多了一些分享方式。

###  1.1 安装

```bash
npm i gitbook-plugin-sharing-plus
```

### 1.2  配置

```bash
{
    "plugins": ["-sharing", "sharing-plus"],
    "pluginsConfig": {
        "sharing": {
             "douban": false,
             "facebook": false,
             "google": true,
             "pocket": false,
             "qq": false,
             "qzone": true,
             "twitter": false,
             "weibo": true,
          "all": [
               "douban", , "weibo",  "messenger","qq", "qzone"
           ]
       }
    }
}
```

其中： 为`true`的代表直接显示在页面顶端，为`false`的不显示，不写默认为false "all"中代表点击分享符号显示出来的

支持的网站如下：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/2cb00e54519839bd541995cb451fa202.png)

###  1.3 效果
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a33fc0c9123da0967215ebf46a044a25.png)

