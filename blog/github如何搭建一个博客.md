----

## 1注册github账户
[https://github.com/](https://github.com/)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6bbcda55441e82c39e4603a44e31afa1.png)
## 2. 创建一个博客项目
更加详细的博客创建流程可以查看[**官网**](https://help.github.com/cn/github/working-with-github-pages/creating-a-github-pages-site)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/cbd4fd5a5d07748c0a893042ad53e4ad.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/22d9314e627dcf74d4c8554999a49ce1.png)
## 3. 为博客设置一个主题
**当然，我们可以跳过这个步骤，用`hugo、jekyll、hexo`等工具设置博客得主题。**
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/9b7e846e73856df4a2d613f7135c9041.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/54dce606f37d352932f22281a5e2d1a3.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/1092613592ee841b501ca2896352294a.png)
这是访问`用户名.github.io`，就可以看到一个属于自己得博客诞生了。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/9f243ce5c9c5a917e8d45c8be440da3d.png)
## 4. 配置自定义域名并免费使用 HTTPS
在 2018 年 5 月 1 日之后，GitHub Pages 已经开始提供免费为自定义域名开启 HTTPS 的功能，并且大大简化了操作的流程，现在用户已经不再需要自己提供证书，只需要将自己的域名使用 CNAME 的方式指向自己的 `GitHub Pages` 域名即可。

### 4.1 创建一个名字为CNAME的文件
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/81668fe952c77cbca2b5dd7ac16bc64a.png)
填写自己的域名
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ec0484646c33c81cbdc017f0a1cda414.png)
### 4.2 添加域名解析
我是在腾讯云购买的域名。[如何购买域名请参考？](https://blog.csdn.net/xixihahalelehehe/article/details/106965024)
![添加 DNS 解析](https://i-blog.csdnimg.cn/blog_migrate/843b9c4730fa50f87cb0be17e01e229d.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/70862f23e7ce403dc3a449d53460e33a.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/83c1e3732b5cf5aa0d734e5b3d6bc614.png)
添加三个记录：

 - 通过 CNAME 的方式指向我刚刚自定义的 GitHub Pages 域名 ghostwritten.github.io
 - 添加能够解析github pages的地址，可查看官网找到它们地址。[请点击](https://help.github.com/cn/github/working-with-github-pages/securing-your-github-pages-site-with-https)。
[Managing a custom domain for your GitHub Pages site](https://docs.github.com/en/free-pro-team@latest/github/working-with-github-pages/managing-a-custom-domain-for-your-github-pages-site#configuring-an-apex-domain)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8582eccba5db841c6ca90dff2917cc77.png)
### 4.3 设置自己的域名
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/402decdeed815747e391eb6da98f01e5.png)
**注意**：也许你会发现下面的`Enforce HTTPS` 点不动，说明你的域名解析配置的有问题；当然，也可以多刷新一下，重新填写域名，看看有没有效果。

如果配置成功，就可以用自己的域名访问博客了。
