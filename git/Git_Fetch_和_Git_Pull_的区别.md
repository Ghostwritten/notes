#  Git Fetch 和 Git Pull 的区别



## 1. 介绍
`Git Fetch` 是一个命令，它告诉本地存储库远程存储库中有可用的更改，而不会将更改带入本地存储库。另一方面，`Git Pull` 将远程目录更改的副本带入本地存储库。让我们借助示例分别查看 Git Fetch 和 Git Pull。

```bash
git pull = git fetch + git merge
```

## 2. Git Fetch 
让我们创建一个名为demo.txt的文件，其中包含“ Hello Geeks”内容，将目录初始化为 git 存储库，并将更改推送到远程存储库。

```bash
git init
git add <Filename>
git commit -m <Commit Message>
git remote add origin <Link to your remote repository>
git push origin <branch name>
```
![](https://i-blog.csdnimg.cn/blog_migrate/02eaaea38c19b13f2fdb6159c10ef893.png)

现在，我们在远程存储库中 有了我的`demo.txt` 。
![](https://i-blog.csdnimg.cn/blog_migrate/57586b7e9df3f95bcdf0ba5a28815f7c.png)
本地和远程存储库现在是同步的，并且在两个地方具有相同的内容。现在让我们更新远程存储库中的 demo.txt 。![](https://i-blog.csdnimg.cn/blog_migrate/fb7554f180bbf5fb90daeab01c2253ed.png)
现在，由于我们已经远程更新了demo.txt，让我们将更改带到我们的本地存储库。我们的本地存储库只有 1 次提交，而远程存储库现在有 2 次提交（观察从`4c4fcb8`开始的第二次提交）。让我们使用`git fetch`命令在本地存储库中查看远程存储库中是否有更改。在此之前，让我们使用`git log`命令查看我们之前的提交。
![](https://i-blog.csdnimg.cn/blog_migrate/341f8b773279748a9a186a8afbf2596a.png)
我们可以看到，在使用`git fetch`之后，我们得到了在远程存储库中完成了一些提交的信息。（注意`4c4fcb8`是我们在远程存储库中第二次提交的首字母）。要将这些更改合并到我们的本地存储库中，我们需要使用`git merge origin/<branch name>`命令。 
![](https://i-blog.csdnimg.cn/blog_migrate/4647fcc1b0e4aaf108349002875cf84d.png)
让我们使用`git log` 命令查看我们在本地存储库中的提交。
![](https://i-blog.csdnimg.cn/blog_migrate/fbb6b4ac6eedc0311a25c2000df8be21.png)
我们在本地存储库中提交了远程存储库。这就是 `git fetch` 的工作原理。现在让我们看一下`git pull`命令。

## 3. Git Pull
让我们对远程存储库中的 demo.txt 文件进行更多更改。
![](https://i-blog.csdnimg.cn/blog_migrate/c94b593a99d803791dedacdce523b8b3.png)
现在，我们在远程存储库中有 3 个提交，而在我们的本地存储库中有 2 个提交。（注意以09d828f开头的第三次提交）。现在让我们使用`git pull origin <branch name>`命令将此更改带到我们的本地存储库。

![](https://i-blog.csdnimg.cn/blog_migrate/5b1ae7a62bd32e9a2a03ee73c8558915.png)
我们可以看到，在git pull命令的帮助下，我们直接获取远程存储库并将其与本地存储库合并。


## 4. 结论


```bash
git pull = git fetch + git merge
```
现在我们的远程和本地存储库再次相互同步。因此，从以上例子中，我们可以得出结论：
|Git fetch |Git pull|
|--|--|
|在不合并到当前分支的情况下提供来自远程存储库的新更改的信息|	从远程存储库中获取所有更改的副本并将它们合并到当前分支中|
|存储库数据在 .git 目录中更新|	直接更新本地仓库|
|可以审查提交和更改	|立即将更改更新到本地存储库。|
|没有合并冲突的可能性。	|如果远程和本地存储库在同一位置进行了更改，则可能会发生合并冲突。|
|Git fetch 的命令是 git fetch<remote>	|Git Pull 的命令是 git pull<remote><branch>|
|Git fetch 基本上将提交导入到本地分支，以便及时了解每个人正在处理的内容。|	Git Pull 基本上使本地分支与远程副本一起更新，远程副本也将更新其他远程跟踪分支。|


参考：
- [Git – Difference Between Git Fetch and Git Pull](https://www.geeksforgeeks.org/git-difference-between-git-fetch-and-git-pull/?ref=rp)

更多阅读：
- [git 、github、gitlab、gitbook、gitOps学习手册](https://ghostwritten.blog.csdn.net/article/details/121107739)
- [Gitbook Docs](https://smoothies.com.cn/gitbook-docs/Overview.html)
