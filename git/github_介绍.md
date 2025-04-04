# github 介绍





---
##  1. github是什么
GitHub 是一个共享虚拟主机服务，用于存放使用Git版本控制的软件代码和内容项目。它由GitHub公司（曾称Logical
Awesome）的开发者Chris Wanstrath、PJ Hyett和Tom Preston-Werner使用Ruby on Rails编写而成。

它还是什么?

 - 网站
 - 免费博客
 - 管理配置文件
 - 收集资料
 - 简历
 - 管理代码片段
 - 托管编程环境
 - 写作
 - 版本管理与软件部署

##  2. GitHub与Git关系
`Git`是一个分布式的版本控制系统，最初由`Linus Torvalds`编写，用作Linux内核代码的管理。在推出后，Git在其它项目中也取
得了很大成功，尤其是在Ruby社区中。目前，包括`Rubinius`、`Merb`和`Bitcoin`在内的很多知名项目都使用了Git。Git同样可以被
诸如`Capistrano`和`Vlad the Deployer`这样的部署工具所使用。

`GitHub`可以托管各种git库，并提供一个web界面，但与其它像 SourceForge或Google Code这样的服务不同，GitHub的独特卖
点在于从另外一个项目进行分支的简易性。为一个项目贡献代码非常简单：首先点击项目站点的“fork”的按钮，然后将代码检出并将
修改加入到刚才分出的代码库中，最后通过内建的“pull request”机制向项目负责人申请代码合并。已经有人将GitHub称为代码
玩家的MySpace。

##  3. 个人帐户的 GitHub 免费
使用个人帐户的 GitHub Free，您可以在具有完整功能集的无限公共存储库和具有有限功能集的无限私有存储库上与无限的协作者合作。

使用 GitHub Free，您的个人帐户包括：

 1. GitHub 社区支持
 2. Dependabot 警报
 3. 两因素身份验证强制执行
 4. 2,000 分钟 GitHub Actions
 5. 500MB GitHub 包存储空间

##  4. GitHub 专业版
除了 GitHub Free 为个人帐户提供的功能外，GitHub Pro 还包括：

 1. GitHub 支持 via email
 2. 3,000 分钟 GitHub Actions
 3. 2GB GitHub 包存储
 
 私有存储库中的高级工具和见解：
 
 4. Required pull request reviewers
 5. Multiple pull request reviewers
 6. Protected branches
 7. Code owners
 8. Auto-linked references
 9. GitHub Pages
 10. Wikis
 11. Repository insights graphs: Pulse, contributors, traffic, commits, code frequency, network, and forks


##  5. GitHub 免费为组织
使用适用于组织的 GitHub Free，您可以在具有完整功能集的无限公共存储库或具有有限功能集的无限私有存储库上与无限的协作者合作。

除了个人账户的 GitHub Free 提供的功能外，组织的 GitHub Free 还包括：

 1. GitHub 社区支持
 2. 团队讨论
 3. 用于管理组的团队访问控制
 4. 2,000 分钟 GitHub Actions
 5. 500MB GitHub 包存储空间


##  6. GitHub 团队
除了 GitHub Free 为组织提供的功能外，GitHub Team 还包括：

 4. Required pull request reviewers
 5. Multiple pull request reviewers
 6. Protected branches
 7. Code owners
 8. Auto-linked references
 9. GitHub Pages
 10. Wikis
 11. Repository insights graphs: Pulse, contributors, traffic, commits, code frequency, network, and forks
 12. 
启用 GitHub 代码空间的选项
组织所有者可以通过设置支出限制并为其组织成员授予用户权限来为组织启用 GitHub Codespaces。有关详细信息，请参阅“为您的组织启用代码空间”。
GitHub 团队的 GitHub 按用户计费。有关详细信息，请参阅“关于按用户定价”。

公共仓库和自托管运行器免费使用 GitHub Actions。 对于私有仓库，每个 GitHub 帐户可获得一定数量的免费记录和存储，具体取决于帐户所使用的产品。 超出包含金额的任何使用量都由支出限制控制。



##  7. 如何用好 GitHub
如何用好 GitHub，并实践一些敏捷软件开发是一个很有意思的事情.我们可以在上面做很多事情,从测试到CI,再到自动部署.， 敏捷软件开发
瀑布流是怎样的？

一个项目的组成：

 - 代码
 - CI
 - 测试
 - 自动化测试
 - 文档
 - 版本管理
 - 自动部署

代码模块化
自动化测试
代码质量与重构

## 8 git 提交规范

### 8.1 工作写法
格式

```bash
 [任务卡号] xx & xx: do something
```

 比如： `[PHODAL-0001] ladohp & phodal: update documents` ，解释如下：

 - PHODAL-0001 ，业务的任务卡号，它可以帮我们找到某个业务修改的原因，即点出相应 bug 的来源
 - ladohp & phodal ，结对编程的两个人的名字，后者（phodal）一般是写代码的人，出于礼貌就放在后面了。由于 Git的提交人只显示一个，所以写上两个的名字。当提交的人不在时，就可以问另外一个人修改的原因。
 - update documents ，我们做了什么事情

缺点：而对于采用看板的团队来说，并不存在任务卡号这种东西，因此就需要一种额外的作法。

###  8.2 常规写法
格式

```bash
[任务分类] 主要修改组件（可选）：修改内容
```
示例 1， `[T] tabs: add icons` 。其中的 `T` 表示这是一个技术卡， `tabs` 表示修改的是Tabs， `add icons` 则表示添加了图标。
示例 2， `[SkillTree] detail: add link data` 。其中的 `SkillTree` 表示修改的是技能树 Tab 下的内容， `detail` 则表示修改的是详情页， `add link data` 则表示是添加了技能的数据

这样做的主要原因是，它可以轻松也帮我 filter 出相应业务的内容。

缺点：要这样做需要团队达到一致，因此付出一些额外的成本。

###  8.3 开源应用、开源库写法
与我们日常工作稍有不同的是：工作中的 Release 计划一般都是事先安排好的，不需要一些CHANGELOG 什么的。而开源应用、开源库需要有对应的 CHANELOG，则添加了什么功能、修改了什么等等。毕竟有很多东西是由社区来维护的。

诸如： `docs(changelog): update change log to beta.5` 中：

 - docs 则对应修改的类型
 - changelog 则是影响的范围
 - subject 则是对应做的事件

对应的类型有：

 - build: 影响构建系统或外部依赖关系的更改（示例范围：gulp，broccoli，npm）
 - ci: 更改我们的持续集成文件和脚本（示例范围：Travis，Circle，BrowserStack，SauceLabs）
 - docs: 仅文档更改
 - feat: 一个新功能
 - fix: 修复错误
 - perf: 改进性能的代码更改
 - refactor: 代码更改，既不修复错误也不添加功能
 - style: 不影响代码含义的变化（空白，格式化，缺少分号等）
 - test: 添加缺失测试或更正现有测试

---

更多阅读：

 - [Gitbook Docs](https://smoothies.com.cn/gitbook-docs/About.html)

---
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/160a8d22d31c13d41ec36b8db8ceaf1e.gif#pic_center)

