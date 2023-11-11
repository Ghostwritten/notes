

##  _config.yaml设置
### 改变背景颜色
![在这里插入图片描述](https://img-blog.csdnimg.cn/8635e073edfa4cd788683359ad062d46.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
_config.yaml
```bash
text_skin: forest
title   : 幽灵代笔
```

### 改变代码颜色
![在这里插入图片描述](https://img-blog.csdnimg.cn/97d860d7716a4147b8e144a13d1bb781.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

```bash
highlight_theme: default # "default" (default), "tomorrow", "tomorrow-night", "tomorrow-night-eighties", "tomorrow-night-blue", "tomorrow-night-bright"
```


### 改变名字描述
![在这里插入图片描述](https://img-blog.csdnimg.cn/3a55fba82a3b445592b2ceb1c4d9ad31.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
_config.yaml

```bash
description: > # this means to ignore newlines until "Language & timezone"
   源于一位名叫大卫·米切尔于1999年创作的第一部小说名字
```
## 设置语言
![在这里插入图片描述](https://img-blog.csdnimg.cn/e224e5c3744048aa96b0bf4667071a04.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/5dd9f2a561024f9b80f006d611a48743.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

```bash
lang: en # the language of your site, default as "en"
lang: zh-Hans # the language of your site, default as "en"
```



### 添加博主联系方式

![在这里插入图片描述](https://img-blog.csdnimg.cn/34e7f5eb64884616b64b2844f158db59.png)
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
![在这里插入图片描述](https://img-blog.csdnimg.cn/68fc4e9461c941c3bc358913789a9051.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

```bash
paginate: 8
```


###  添加统计用户访问
-------------
![在这里插入图片描述](https://img-blog.csdnimg.cn/049729565e72443c89a32a521fd8b7e6.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

```bash
pageview:
  provider: false # false (default), "leancloud", "custom"

  ## Leancloud
  leancloud:
    app_id    : # LeanCloud App id
    app_key   : # LeanCloud App key
    app_class : # LeanCloud App class
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/f9acb739368b4806a6a611e1691803e9.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

```bash
pageview:
  provider: leancloud
  leancloud:
    app_id: uAG3OhdcH8H4fxSqXLyBljA7-gzGzoHsz
    app_key: Mzf5m9skSwYVWVXhGiYMNyXs
    app_class: ThomasBlog
```

##  _data/navigation.yml
![在这里插入图片描述](https://img-blog.csdnimg.cn/a5437ef14e0d4becad5cfb8f6b59f7c1.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/a99dbb37625146369902e8a5fb79db42.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

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



