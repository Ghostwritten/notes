#  Github | 制作您的第一个开源合并请求

![](https://i-blog.csdnimg.cn/blog_migrate/0f841413d163111372505eaf1a07057c.png)





## 1. 背景
开源软件是原始源代码可免费获得并可重新分发和修改的软件。作为一名程序员，我们更感兴趣的是如何为他们的代码库做出贡献。
许多新手发现开源是可怕和令人生畏的。但不要担心，每个伟大的开源贡献者都曾经在你现在所在的地方。


## 2. 前提
以下是深入开源之前所需的先决条件：

- 对您选择的至少一种编程语言有很好的理解
- 版本控制：Git/SVN 和 [Github](https://github.com/)、[Bitbucket](https://bitbucket.org/product/)、[Gitlab](https://about.gitlab.com/)
- 学习阅读大的源代码，这样它就不会显得乱七八糟。这篇文章可能会有所帮助。
- 了解如何使用错误/问题跟踪器

## 3. 上手贡献开源
- [Up-for-grads](https://up-for-grabs.net/#/)
- [Open Hatch](https://openhatch.org/)
- [Github: Great for new contributors](https://github.com/showcases/great-for-new-contributors)
- [Firsttimersonly](https://www.firsttimersonly.com/)

给定的资源包括不同组织的项目列表，可以根据使用的编程语言、项目类别（例如 Web、数据库等）和难度进行过滤。

发起 Pull Request 的步骤：

一旦您决定了要贡献的存储库或要处理的问题，请按照以下步骤发出您的第一个拉取请求：

1. 阅读 CONTRIBUTING.md 指南（如果存在）
![](https://i-blog.csdnimg.cn/blog_migrate/f5b7c4824f1d6e25d213fec1347e276a.png)
2. 与维护者讨论这个问题，提出问题（如果有的话）并清除疑虑。他们是可爱的人，随时准备提供帮助。您还可以通过他们的 IRC 或邮件列表 ping 他们。
3. 继续并 Fork 存储库
![](https://i-blog.csdnimg.cn/blog_migrate/acc5acb20729c189e888db73135b1a95.png)
![](https://i-blog.csdnimg.cn/blog_migrate/52b3967ddb57836b84cadb93e120aac9.png)
4. Clone the repo：`git clone https://github.com/YOUR_USERNAME/PROJECT.git`

![](https://i-blog.csdnimg.cn/blog_migrate/05922072e6a57037c94290ed55f0a08d.png)

5. Add Upstream: `git remote add upstream https://github.com/PROJECT_USERNAME/PROJECT.git`
6. Create new branch: `git checkout -b BRANCH_NAME`

![](https://i-blog.csdnimg.cn/blog_migrate/fd68a5bfd64f79b0a3666fbe03b1df1e.png)

7. 代码代码代码：进行必要的更改
8. 推送更改：`git push origin BRANCH_NAME`
9. 通过 Github 创建拉取请求
 ![](https://i-blog.csdnimg.cn/blog_migrate/bffd44dda8d6bda25a98b1a6b0c5d550.png)
其他一些有用的命令：

- 检查远程链接：`git remote -v`
- 检查分支：`git branch`
- 删除分支`：git branch -D BRANCH_NAME`
- 删除 Github 上的分支：`git push origin --delete BRANCH_NAME`

现在您所要做的就是等待您的更改被维护者审查并合并（或丢弃）。

当您发现您编写的一段代码每天都被世界各地的人们使用时，您会感觉很好。
另外，您可能想查看[GSOC](https://summerofcode.withgoogle.com/)。如果这不能激励您从开源开始，那么没有别的可能。

另请参阅 –[如何开始使用开源](https://www.geeksforgeeks.org/contributing-to-open-source-getting-started/)。

参考：
- [Making your first Open Source Pull Request | Github](https://www.geeksforgeeks.org/making-first-open-source-pull-request/?ref=rp)

更多阅读：
- [git 、github、gitlab、gitbook、gitOps学习手册](https://ghostwritten.blog.csdn.net/article/details/121107739)
- [Gitbook Docs](https://smoothies.com.cn/gitbook-docs/Overview.html)
