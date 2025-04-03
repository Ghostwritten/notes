

##  _config.yaml设置
### 改变背景颜色
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/42d3d85d86a5d08c5572f3e077cb2407.png)
_config.yaml
```bash
text_skin: forest
title   : 幽灵代笔
```

### 改变代码颜色
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b38bad953b8aa2520a7a9b47f8a99d67.png)

```bash
highlight_theme: default # "default" (default), "tomorrow", "tomorrow-night", "tomorrow-night-eighties", "tomorrow-night-blue", "tomorrow-night-bright"
```


### 改变名字描述
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/1d57417f4bd2f79d00aa97b8bab373c5.png)
_config.yaml

```bash
description: > # this means to ignore newlines until "Language & timezone"
   源于一位名叫大卫·米切尔于1999年创作的第一部小说名字
```
## 设置语言
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/9de11db8012da7913f1421ee4ecf3a95.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0dfb6270c85b902da6e4d7f058ebe9e4.png)

```bash
lang: en # the language of your site, default as "en"
lang: zh-Hans # the language of your site, default as "en"
```



### 添加博主联系方式

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/86436210ebaede851c3ca24368254685.png)
_config.yaml
```bash
author:
  type      : # "person" (default), "organization"
  name      : Your Name
  url       :
  avatar    : # path or url of avatar image (square)
  bio       : I am an amazing person.
  email     : 1zoxun1@gmail.com
  facebook  : # "user_name" the last part of your profile url, e.g. https://www.facebook.com/user_name
  twitter   :  "cnghostwritten" #the last part of your profile url, e.g. https://twitter.com/user_name
  weibo     : # "user_id"   the last part of your profile url, e.g. https://www.weibo.com/user_id/profile?...
  googleplus: # "user_id"   the last part of your profile url, e.g. https://plus.google.com/u/0/user_id
  telegram  : # "user_name" the last part of your profile url, e.g. https://t.me/user_name
  medium    : # "user_name" the last part of your profile url, e.g. https://medium.com/user_name
  zhihu     :  "cao-zhi-39-73" #the last part of your profile url, e.g. https://www.zhihu.com/people/user_name
  douban    : # "user_name" the last part of your profile url, e.g. https://www.douban.com/people/user_name
  linkedin  : # "user_name" the last part of your profile url, e.g. https://www.linkedin.com/in/user_name
  github    :  "Ghostwritten" #the last part of your profile url, e.g. https://github.com/user_name
  npm       : # "user_name" the last part of your profile url, e.g. https://www.npmjs.com/~user_name
```

### 几个文章分页
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/2d853e13b688c1d45b65f69532edc989.png)

```bash
paginate: 8
```


###  添加统计用户访问
-------------
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/90ce778382af5dc21c62860012ee2224.png)

```bash
pageview:
  provider: false # false (default), "leancloud", "custom"

  ## Leancloud
  leancloud:
    app_id    : # LeanCloud App id
    app_key   : # LeanCloud App key
    app_class : # LeanCloud App class
```
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/1a63ee9d3ffaf1d8a93b1106f1514096.png)

```bash
pageview:
  provider: leancloud
  leancloud:
    app_id: uAG3OhdcH8H4fxSqXLyBljA7-gzGzoHsz
    app_key: Mzf5m9skSwYVWVXhGiYMNyXs
    app_class: ThomasBlog
```

##  _data/navigation.yml
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/16b6a6703c14673a9c130f23e6f77f1a.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/201df81ebe3de072e691a77c46123969.png)

```bash
  - titles:
      # @start locale config
      en      : &EN       Book
      en-GB   : *EN
      en-US   : *EN
      en-CA   : *EN
      en-AU   : *EN
      zh-Hans : &ZH_HANS  书
      zh      : *ZH_HANS
      zh-CN   : *ZH_HANS
      zh-SG   : *ZH_HANS
      zh-Hant : &ZH_HANT  書
      zh-TW   : *ZH_HANT
      zh-HK   : *ZH_HANT
      ko      : &KO       책
      ko-KR   : *KO
      fr      : &FR       Book
      fr-BE   : *FR
      fr-CA   : *FR
      fr-CH   : *FR
      fr-FR   : *FR
      fr-LU   : *FR
      # @end locale config
    url: /book.html
```



