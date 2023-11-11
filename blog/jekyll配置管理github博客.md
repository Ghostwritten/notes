## 1 添加草稿
草稿即为保存起来但又暂时不想被别人看到的文章。
创建一个`_drafts`目录，把未写完的文章放在该目录下，就可以保存起来发布但别人看到，如何需要发布，只需将文章拷贝到`_post`目录即可。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200629152847692.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
## 2 目录与配置文件说明
.

```bash
|- _config.yml     #配置文件
|- index.html      #主页文件
|- pages           #和index.html处于相同地位的网站页面
|- _drafts         #草稿文件夹
|- _posts          #源文件夹
|  |- file.md
|  |- file.html
|- _sass           #样式表文件  
|- _includes       #引用文件夹，像一个挂件
|  |- themes        #主题文件夹
|  |- widgets       #插件文件夹
|- _layouts        #模板文件夹,也叫布局文件
|  |- default.html
|  |- post.html
|- assets          #资源文件
|  |- fav.ico
|  |- css
|  |- js
|- _site           #目标文件夹 ,一般不会上传代码仓库，可以在.gitgore文件忽略
|- CNAME           #拥有自已的域名时，该文件制定域名
|- LICENSE
|- README
|- changelog.md
```
##  bundle命令
生成`Gemfile`

```bash
$ bundle init
Writing new Gemfile to D:/github/ghostwritten.github.io/Gemfile
```

## 3 jekyll命令
### 3.1 jekyll new <PATH>

```c
jekyll new <PATH>      #在指定的路径（相对于当前目录）安装一个新的Jekyll站点
jekyll new .           #要将Jekyll站点安装到你当前所在的目录中
jekyll new . --force   #如果现有的目录不是空的，可以强制安装。
```

jekyll new会自动启动`bundle install`以安装所需的依赖关系。
默认情况下，由jekyll new安装的Jekyll站点使用一个叫做为Minima的基于gem的主题。

### 3.2 jekyll build
Jekyll gem在终端窗口中为你提供了一个jekyll可执行文件。

```c
$ jekyll build
or 
$ jekyll b
# => 当前文件夹中的内容将被生成到./_site

$ jekyll build --destination <destination>
# => 当前文件夹中的内容将被生成到<destination>

$ jekyll build --source <source> --destination <destination>
# => <source>文件夹中的内容将被生成到<destination>

$ jekyll build --watch
# => 当前文件夹中的内容将被生成到./_site,
#    检查改动，并自动重新生成。
```
### 3.3 jekyll serve

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


## 4 拷贝他人优秀的博客
借用[https://github.com/melangue/dactl](https://github.com/melangue/dactl)

git bash工具克隆此项目

```bash
$ git clone https://github.com/melangue/dactl
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200629172433710.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
打开终端：
执行安装一些gem包

```bash
$ bundle install 
$ bundle update
$ bundle exec jekyll serve
```

遇到的问题：[Can't install with Ruby 2.5 #598](https://github.com/ffi/ffi/issues/598)， `bundle update`解决。

界面访问：[http://127.0.0.1:4000/dactl/](http://127.0.0.1:4000/dactl/)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200629172648616.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
把`_post`目录下博客文件换成自己的博客文章，并且`CNAME`文件修改成自己的域名，复制到自己的本地github博客目录下，上传github就可以以域名的方式访问了。
