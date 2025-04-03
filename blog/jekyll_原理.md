


---

## 1. jekyll new

`jekyll new <PATH>`在指定的路径（相对于当前目录）安装一个新的Jekyll站点。 在上面列出的命令行的情况下，Jekyll将被安装在当前目录下的名为myblog的目录中。这里有一些额外的细节：

 - 要将Jekyll站点安装到你当前所在的目录中，请运行`jekyll new .`。 如果现有的目录不是空的，你可以使用`jekyll new . --force`传递`--force`选项。
 - `jekyll new`会自动启动`bundle install`以安装所需的依赖关系。 （如果你不想让`Bundler`安装`gem`，请使用`jekyll new myblog --skip-bundle`。）
 - 默认情况下，由`jekyll new`安装的Jekyll站点使用一个叫做为`Minima`的基于gem的主题。使用基于gem的主题，一些目录和文件存储在gem主题中，隐藏在你的即时视图中。
 - 我们建议你将`Jekyll`设置为基于`gem`的主题，但如果你想从空白的模版开始，请使用`jekyll new myblog --blank`。
 - 要了解其他参数，你可以使用`jekyll new`，输入`jekyll new --help`即可。

## 2. jekyll build

```bash
jekyll build
# => 当前文件夹中的内容将被生成到./_site

jekyll build --destination <destination>
# => 当前文件夹中的内容将被生成到<destination>

jekyll build --source <source> --destination <destination>
# => <source>文件夹中的内容将被生成到<destination>

jekyll build --watch
# => 当前文件夹中的内容将被生成到./_site,
#    检查改动，并自动重新生成。
```
## 3. 自定义开发配置
发环境的默认预览地址是`http://localhost:4000`。[3.3.0]
如果你想为你的生产环境构建：

 - 在`_config.yml`中设置你的生产环境URL。例如：`https://example.com`。
 - 运行`JEKYLL_ENV=production bundle exec jekyll build`。

> 注意: `_config.yml`**的更改并不会在自动生成更新过程中生效。**
> `_config.yml`主要配置文件会在执行时读取一次全局配置和变量定义。在下次执行之前，不会加载在自动生成更新期间对_config.yml所做的更改。注意，自动生成更新过程中会对[Data Files](https://jekyllrb.com/docs/datafiles/)重新加载。

> 警告：目标文件夹在网站构建的时候会被清空
> 默认情况下，当网站构建的时候，`<destination>`中的内容会被自动的清空。不是被你的网站构建时所创建的文件和文件夹都会被删除。可以在`<keep_files>`配置指令中指定你希望保留在中的文件和文件夹。
> 不要设置为重要本地路径；相反，应该将其用作暂存区域并将文件从那里复制到你的Web服务器。

##  4. Jekyll serve
还附带了一个内置的开发服务器，可以让你在本地浏览中浏览生成的网站。
jekyll serve
```bash
# => 开发服务将会运行在http://localhost:4000/
# 自动生成更新会被开启，如果不想开启请使用`--no-watch`。

jekyll serve --no-watch
# => 等同于`jekyll serve`，但是内容更改时不会自动生成新的。


jekyll serve --livereload
# LiveReload将在更新后刷新浏览器页面。

jekyll serve --incremental
# Incremental将会匹配更改部分，执行部分构建以减少自动生成更新时间。

jekyll serve --detach
# => 等同于`jekyll serve`，但是不会再当前终端中显示运行状态，而是转为后台模式。
#    如果你需要关闭服务，你可以`kill -9 1234`，这里的"1234"是PID。
#    如果你不知道PID，那么就执行`ps aux | grep jekyll`并关闭这个实例。
```

这些只是几个可用的配置选项。许多配置选项可以在命令行中指定为`flags`，或者（更常见）它们可以在源目录的根目录下的`_config.yml`文件中指定。运行时，Jekyll会自动使用该文件中的选项。例如，如果将以下行放在`_config.yml`文件中：

```bash
source:      _source
destination: _deploy
```
那么以下两个命令将是等效的：

```bash
jekyll build
# 上面的命令添加过配置后等于下面的命令没加配置
jekyll build --source _source --destination _deploy
```

##  5. 目录结构
`Jekyll`的核心是一个文本转换引擎。这个系统的大概是：你给它的内容可以用你最喜欢的标记语言编写，可以是`Markdown`，是`Textile`或者是纯粹的`HTML`，然后Jekyll通过一个或一系列布局文件混合它们。在整个过程中，你可以调整网站URL的样式，布局中显示的数据等等。这一切都是通过编辑文本文件完成的，而静态网站就是它的最终产品。
基本的Jekyll站点通常看起来像这样：

```bash
.
├── _config.yml
├── _data
|   └── members.yml
├── _drafts
|   ├── begin-with-the-crazy-ideas.md
|   └── on-simplicity-in-technology.md
├── _includes
|   ├── footer.html
|   └── header.html
├── _layouts
|   ├── default.html
|   └── post.html
├── _posts
|   ├── 2007-10-29-why-every-programmer-should-play-nethack.md
|   └── 2009-04-26-barcamp-boston-4-roundup.md
├── _sass
|   ├── _base.scss
|   └── _layout.scss
├── _site
├── .jekyll-metadata
└── index.html # can also be an 'index.md' with valid YAML Frontmatter
```

> 注意：使用基于gem的主题的Jekyll网站的目录结构 始于`Jekyll 3.2`，一个使用`jekyll new`创建的基于`gem`的主题来定义网站外观的新的Jekyll项目，会有一个比较简单的默认目录结构：默认情况下，`_layouts`，`_includes`和`_sass`会被存储在`theme-gem`中，而不是放在项目目录中。minima是当前的默认主题，`bundle show minima`会告诉你`minima`主题的文件存储在计算机上的哪个位置。

目录结构说明：
|文件/文件夹|说明|
|--|--|
|_config.yml|存储配置数据。这些配置中的许多选项都可以从命令行中指定，但在这里指定它们更加容易，并且你不必记住它们。|
|_drafts|草稿是未发布的文章。这些文件的命名格式是没有日期的：`title.MARKUP`。了解[如何使用草稿](https://jekyllrb.com/docs/posts/)。|
|_includes|这些是你的布局和帖子可以混合和匹配的部分，以促进重用。液体标签 `{% include file.ext %}` 可以用来包括在 `_includes/file.ext.`|
|_layouts|这些是包装文章的模板。在[YAML Front Matter](https://jekyllrb.com/docs/front-matter/)中逐层选择布局，这将在下一节中介绍。 The liquid tag `{{ content }}`用于将内容注入网页。|
|_posts|可以这么说，这里是你的动态内容。这些文件的命名约定很重要，并且必须遵循以下格式：`YEAR-MONTH-DAY-title.MARKUP`。可以为每篇文章指定[固定链接](https://jekyllrb.com/docs/permalinks/)，但日期和MARKUP语言完全由文件名决定。|
|_data|格式良好的网站数据应该放在这里。Jekyll引擎将自动加载该目录中的所有数据文件（使用`.yml`，`.yaml`，`.json`或`.csv`格式和扩展名），并且可以通过`site.data`访问它们。如果目录下有文件`members.yml`，则可以通过`site.data.members`访问该文件的内容。|
|_sass|是可以导入到`main.scss`中的sass部分，然后将它们处理成一个样式表`main.css`，该样式表定义了你的网站使用的样式。|
|_site|这是Jekyll完成转换后，生成的网站将被存放的（默认）位置。建议将它添加到`.gitignore`文件中。|
|.jekyll-metadata|临时文件，这些将帮助Jekyll追踪自上次构建站点后哪些文件未被修改，以及哪些文件需要在下一个版本中重新生成。该文件不会包含在生成的网站中。建议将它添加到`.gitignore`文件中。|
|index.html或index.md或其他HTML、Markdown文件|假设该文件具有YAML Front Matter部分，它将由Jekyll进行转换。网站根目录中的任何`.html`，`.markdown`，`.md`或`.textile`文件或上面未列出的目录也会发生同样的情况。|
|其他文件/文件夹|除了上面列出之外的其他文件夹和文件（例如`css`和`images`文件夹，`favicon.ico`文件等），将会被逐字复制到生成的网站中。如果你想知道它们是如何布置的，有[很多网站](https://jekyllrb.com/showcase/)已经在使用Jekyll。|

---

 - [https://jekyllrb.com/docs/](https://jekyllrb.com/docs/)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/21a6ed38c88a74a8ed49d5bb4f6ca822.gif#pic_center)

