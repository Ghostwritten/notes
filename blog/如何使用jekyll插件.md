## 插件配置
_config.yml

```bash
plugins:
  - jekyll-sitemap # Create a sitemap using the official Jekyll sitemap gem
  - jekyll-feed # Create an Atom feed using the official Jekyll feed gem
  - jekyll-paginate
  - jekyll-admin
  - jekyll-gist
  - jekyll-coffeescript
  - jekyll-seo-tag
  - jekyll-remote-theme
```

## 插件安装

```bash
$ gem install jekyll-gist jekyll-coffeescript jekyll-remote-theme jekyll-sitemap jekyll-feed jekyll-paginate jekyll-admin j
ekyll-seo-tag
Successfully installed jekyll-gist-1.5.0
Parsing documentation for jekyll-gist-1.5.0
Installing ri documentation for jekyll-gist-1.5.0
Done installing documentation for jekyll-gist after 0 seconds
Fetching jekyll-coffeescript-2.0.0.gem
Successfully installed jekyll-coffeescript-2.0.0
Parsing documentation for jekyll-coffeescript-2.0.0
Installing ri documentation for jekyll-coffeescript-2.0.0
Done installing documentation for jekyll-coffeescript after 0 seconds
Fetching jekyll-remote-theme-0.4.3.gem
Fetching rubyzip-2.3.2.gem
RubyZip 3.0 is coming!
**********************

The public API of some Rubyzip classes has been modernized to use named
parameters for optional arguments. Please check your usage of the
following classes:
  * `Zip::File`
  * `Zip::Entry`
  * `Zip::InputStream`
  * `Zip::OutputStream`

Please ensure that your Gemfiles and .gemspecs are suitably restrictive
to avoid an unexpected breakage when 3.0 is released (e.g. ~> 2.3.0).
See https://github.com/rubyzip/rubyzip for details. The Changelog also
lists other enhancements and bugfixes that have been implemented since
version 2.3.0.
Successfully installed rubyzip-2.3.2
Successfully installed jekyll-remote-theme-0.4.3
Parsing documentation for rubyzip-2.3.2
Installing ri documentation for rubyzip-2.3.2
Parsing documentation for jekyll-remote-theme-0.4.3
Installing ri documentation for jekyll-remote-theme-0.4.3
Done installing documentation for rubyzip, jekyll-remote-theme after 1 seconds
Successfully installed jekyll-sitemap-1.4.0
Parsing documentation for jekyll-sitemap-1.4.0
Done installing documentation for jekyll-sitemap after 0 seconds
Fetching jekyll-feed-0.16.0.gem
Successfully installed jekyll-feed-0.16.0
Parsing documentation for jekyll-feed-0.16.0
Installing ri documentation for jekyll-feed-0.16.0
Done installing documentation for jekyll-feed after 0 seconds
Successfully installed jekyll-paginate-1.1.0
Parsing documentation for jekyll-paginate-1.1.0
Done installing documentation for jekyll-paginate after 0 seconds
Fetching jekyll-admin-0.11.0.gem
Successfully installed jekyll-admin-0.11.0
Parsing documentation for jekyll-admin-0.11.0
Installing ri documentation for jekyll-admin-0.11.0
Done installing documentation for jekyll-admin after 4 seconds
Fetching jekyll-seo-tag-2.8.0.gem
Successfully installed jekyll-seo-tag-2.8.0
Parsing documentation for jekyll-seo-tag-2.8.0
Installing ri documentation for jekyll-seo-tag-2.8.0
Done installing documentation for jekyll-seo-tag after 0 seconds
9 gems installed
```



### 插件大全
### jekyll admin
可以管理文档
官方文档：[https://jekyll.github.io/jekyll-admin/](https://jekyll.github.io/jekyll-admin/)
github文档：[https://github.com/jekyll/jekyll-admin](https://github.com/jekyll/jekyll-admin)


Genfile添加

```bash
gem 'jekyll-admin', group: :jekyll_plugins
```
运行

```bash
bundle install
```
 测试

```bash
bundle exec jekyll serve
```
访问`http://127.0.0.1:4000/admin`

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/305bd250fa60c7deafc82f77681d5f86.png)
### 配置隐藏专栏
在_config.yml配置

```bash
jekyll_admin:
  hidden_links:
    - posts
    - pages
    - staticfiles
    - datafiles
    - configuration
```

重新执行：
```bash
bundle exec jekyll serve
```
访问`http://127.0.0.1:4000/admin`
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d4814f8aec3bd9ba099c1ef13e901960.png)
### 在线编写博客
#### 标题

```bash
#  一级标题
##  二级标题
###  三级标题
####  四级标题
```
##### 无序分行

```bash
- 第一行
- 第二行
```
##### 有序分行

```bash
 1. 第一行
 2. 第二行
```
#### 特殊字符显色

```bash
`hello world`
```
#### 代码插入


 
```
#！/bin/bash
......
```
`

#### 代码高亮

```bash
{% highlight ruby %}
#!/bin/bash
.........
{% endhighlight %}
```
#### 添加 New metadata field
添加layout 分列布局
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/5c736b8f04f51ab8390ace6bc79476cd.png)
效果显示；否则，会以网页的形式直接显示文章内容。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4ef6d7f5870fca2d5626ac06bd97f361.png)
添加时间轴，可以自定义发行日期。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/376486182e42b6f0d595a7f2ff56a137.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4bfa5d5dc9cef94a8099788cae91c398.png)
##  jekyll-paginate
官网文档：[https://jekyllcn.com/docs/pagination/](https://jekyllcn.com/docs/pagination/)
github：[https://github.com/jekyll/jekyll-paginate](https://github.com/jekyll/jekyll-paginate)

Genfile添加

```bash
gem 'jekyll-paginate'
```
_config.yml添加

```bash
plugins:
  - jekyll-paginate

# Pagination
paginate                     : 5    #在生成的站点中每页展示博客数目的最大值
paginate_path                : '/page-:num/'    #定分页页面的目标路径
```

安装

```bash
 gem install jekyll-paginate
```

**注意：**
分页功能只支持 HTML 文件
Jekyll 的分页功能不支持 Jekyll site 中的 Markdown 或 Textile 文件。分页功能从名为 index.html 的 HTML 文件中被调用时，才能工作。分页功能是可选的，可能通过 paginate_path 配置的值，驻留和生成在子目录中。


创建`index.html`,并添加如下内容：

```css
---

layout: default
title: My Blog
---

<!-- 遍历分页后的文章 -->
{% for post in paginator.posts %}
  <h1><a href="{{ post.url }}">{{ post.title }}</a></h1>
  <p class="author">
    <span class="date">{{ post.date }}</span>
  </p>
  <div class="content">
    {{ post.content }}
  </div>
{% endfor %}

<!-- 分页链接 -->
<div class="pagination">
  {% if paginator.previous_page %}
    <a href="/page{{ paginator.previous_page }}" class="previous">Previous</a>
  {% else %}
    <span class="previous">Previous</span>
  {% endif %}
  <span class="page_number ">Page: {{ paginator.page }} of {{ paginator.total_pages }}</span>
  {% if paginator.next_page %}
    <a href="/page{{ paginator.next_page }}" class="next">Next</a>
  {% else %}
    <span class="next ">Next</span>
  {% endif %}
</div>
```
界面暂时并没什么变化。
