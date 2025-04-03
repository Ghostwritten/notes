

##  安装插件

1. chrome 添加 [Tampermonkey 插件](https://chrome.google.com/webstore/detail/tampermonkey/dhdgffkkebhmkfjojejmpbldmpobfkfo/related?hl=zh-CN)
2. 打开管理面板
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/21de2642d5263c8003bf41bc20920ac9.png)


3. 打开 [mdn web 文档](https://developer.mozilla.org/en-US/)   学习 [contenteditable](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Global_attributes/contenteditable)
4. 打开[csdn 任意博客界面](https://ghostwritten.blog.csdn.net/article/details/126298557?spm=1001.2014.3001.5502)，未登录状态会发现无法复制代码
5. 编写代码

CsdnCopy.js
```bash
//获取所有代码块
let codes = document。querySelectorALL("code");
//循环遍历所有代码块
codes.forEach(c => {
    //设置代码块可以编辑，从而实现可赋值
    c.contentEditable = "true";
});
```
6.Tampermonkey 添加脚本
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/85ce4dba2c31d7e2ef2f490828056ed4.png)
##  编写注释
匹配URL

```bash
// @match        https://blog.csdn.net/*/artile/details/*
```


获取图标，在网页按F12，
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/678d8513a750653da9e69601d1198346.png)
注释内容：


```bash
// ==UserScript==
// @name         CSDN 免登陆复制
// @version      1.0
// @description  CSDN 免登陆状态实现复制代码
// @author       ghostwritten
// @match        https://blog.csdn.net/*/artile/details/*
// @icon         https://g.csdnimg.cn/static/logo/favicon32.ico
// ==/UserScript==
```
按`CTRL + s`（保存），效果展示
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/9d3970348b27461c94cab9dbc71b5246.png)
## 编写代码
代码美化
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/1757c32fc763e60e2709450ad400e6b6.png)
最终代码：

```bash
// ==UserScript==
// @name         CSDN 免登陆复制
// @version      1.0
// @description  CSDN 免登陆状态实现复制代码
// @author       ghostwritten
// @match        https://blog.csdn.net/*/artile/details/*
// @icon         https://g.csdnimg.cn/static/logo/favicon32.ico
// ==/UserScript==

(function() {
    //获取所有代码块
    let codes = document.querySelectorALL("code");
    //循环遍历所有代码块
    codes.forEach(c => {
        //设置代码块可以编辑，从而实现可赋值
        c.contentEditable = "true";
    });
})();
```

##  测试效果
免登陆复制了
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/18b001b39e30f223fa92047e0147abee.png)
`WIN + v`（显示复制面板）

参考：

 - [手写油猴脚本，几分钟学会新技能](https://www.bilibili.com/video/BV1yT411L7n7?spm_id_from=333.999.0.0&vd_source=ae7b192be069682aabc96350ba419fc5)

