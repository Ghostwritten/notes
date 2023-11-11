![在这里插入图片描述](https://img-blog.csdnimg.cn/21bc16a65d2041e4ba1be2ab204da10e.png)







##  1. 背景
很多不注重commit ，导致其他开发人员无法看懂

##  2. 优秀的 commit
- 可以使自己或者其他开发人员能够清晰地知道每个 commit 的变更内容，方便快速浏览变更历史，比如可以直接略过文档类型或者格式化类型的代码变更。
- 可以基于这些 Commit Message 进行过滤查找，比如只查找某个版本新增的功能：`git log --oneline --grep "^feat|^fix|^perf"`。
- 可以基于规范化的 Commit Message 生成 Change Log。
- 可以依据某些类型的 Commit Message 触发构建或者发布流程，比如当 `type` 类型为 `feat`、`fix` 时我们才触发 CI 流程。
- 确定语义化版本的版本号。比如 `fix` 类型可以映射为 `PATCH` 版本，`feat` 类型可以映射为 `MINOR` 版本。带有 `BREAKING CHANGE` 的 commit，可以映射为 `MAJOR` 版本。我就是通过这种方式来自动生成版本号。


## 3. Commit Message 的规范
我们可以根据需要自己来制定 Commit Message 规范，但是我更建议你采用开源社区中比较成熟的规范。
- 一方面，可以避免重复造轮子，提高工作效率。
- 另一方面，这些规范是经过大量开发者验证的，是科学、合理的。

目前，社区有多种 Commit Message 的规范，例如 [jQuery](https://jquery.com/)、[Angular](https://angular.io/) 等。我将这些规范及其格式绘制成下面一张图片，供你参考：

![在这里插入图片描述](https://img-blog.csdnimg.cn/d9e4b58e4bab49adae006b7af39c2b7f.png)
在这些规范中，Angular 规范在功能上能够满足开发者 commit 需求，在格式上清晰易读，目前也是用得最多的。Angular 规范其实是一种语义化的提交规范（Semantic Commit Messages），所谓语义化的提交规范包含以下内容：
- `Commit Message` 是语义化的：Commit Message 都会被归为一个有意义的类型，用来说明本次 commit 的类型。
- `Commit Message` 是规范化的：Commit Message 遵循预先定义好的规范，比如 Commit Message 格式固定、都属于某个类型，这些规范不仅可被开发者识别也可以被工具识别。

为了方便你理解 Angular 规范，我们直接看一个遵循 Angular 规范的 commit 历史记录，见下图：
![在这里插入图片描述](https://img-blog.csdnimg.cn/f79679618f9440c099bacde1c8dfc272.png)
再来看一个完整的符合 Angular 规范的 Commit Message，如下图所示：
![在这里插入图片描述](https://img-blog.csdnimg.cn/7a44fe54a3904a11a1a9bf5ee146b452.png)
通过上面 2 张图，我们可以看到符合 `Angular Commit Message` 规范的 `commit` 都是有一定格式，有一定语义的。

## 4. 怎么写出符合 Angular 规范的 Commit Message
在 Angular 规范中，Commit Message 包含三个部分，分别是 `Header`、`Body` 和 `Footer`，格式如下：

```bash

<type>[optional scope]: <description>
// 空行
[optional body]
// 空行
[optional footer(s)]
```
其中，`Header` 是必需的，`Body` 和 `Footer` 可以省略。在以上规范中，`<scope>`必须用括号 `()` 括起来， `<type>[<scope>]` 后必须紧跟冒号 ，冒号后必须紧跟空格，2 个空行也是必需的。

在实际开发中，为了使 Commit Message 在 GitHub 或者其他 Git 工具上更加易读，我们往往会限制每行 message 的长度。根据需要，可以限制为 `50/72/100` 个字符，这里我将长度限制在 72 个字符以内（也有一些开发者会将长度限制为 100，你可根据需要自行选择）。

以下是一个符合 Angular 规范的 Commit Message：

```bash

fix($compile): couple of unit tests for IE9
# Please enter the Commit Message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
# On branch master
# Changes to be committed:
# ...

Older IEs serialize html uppercased, but IE9 does not...
Would be better to expect case insensitive, unfortunately jasmine does
not allow to user regexps for throw expectations.

Closes #392
Breaks foo.bar api, foo.baz should be used instead
```
接下来，我们详细看看 Angular 规范中 Commit Message 的三个部分。

### 4.1 Header
`Header` 部分只有一行，包括三个字段：`type`（必选）、`scope`（可选）和 `subject`（必选）。我们先来说 `type`，它用来说明 commit 的类型。为了方便记忆，我把这些类型做了归纳，它们主要可以归为 `Development` 和 `Production` 共两类。它们的含义是：

- `Development`：这类修改一般是项目管理类的变更，不会影响最终用户和生产环境的代码，比如 CI 流程、构建方式等的修改。遇到这类修改，通常也意味着可以免测发布。
- `Production`：这类修改会影响最终的用户和生产环境的代码。所以对于这种改动，我们一定要慎重，并在提交前做好充分的测试。
#### 4.1.1 type
我在这里列出了 Angular 规范中的常见 type 和它们所属的类别，你在提交 Commit Message 的时候，一定要注意区分它的类别。举个例子，我们在做 Code Review 时，如果遇到 Production 类型的代码，一定要认真 Review，因为这种类型，会影响到现网用户的使用和现网应用的功能。

![在这里插入图片描述](https://img-blog.csdnimg.cn/2360a892b779458e956a4f5e6d3b280b.png)
有这么多 type，我们该如何确定一个 commit 所属的 type 呢？这里我们可以通过下面这张图来确定。
![在这里插入图片描述](https://img-blog.csdnimg.cn/ff4995c04e4c4f32b9191c6f2d66a17f.png)
如果我们变更了应用代码，比如某个 Go 函数代码，那这次修改属于代码类。在代码类中，有 4 种具有明确变更意图的类型：feat、fix、perf 和 style；如果我们的代码变更不属于这 4 类，那就全都归为 refactor 类，也就是优化代码。

如果我们变更了非应用代码，例如更改了文档，那它属于非代码类。在非代码类中，有 3 种具有明确变更意图的类型：test、ci、docs；如果我们的非代码变更不属于这 3 类，那就全部归入到 chore 类。

Angular 的 Commit Message 规范提供了大部分的 type，在实际开发中，我们可以使用部分 type，或者扩展添加我们自己的 type。但无论选择哪种方式，我们一定要保证一个项目中的 type 类型一致。

#### 4.1.2 scope

scope 是用来说明 commit 的影响范围的，它必须是名词。显然，不同项目会有不同的 scope。在项目初期，我们可以设置一些粒度比较大的 scope，比如可以按组件名或者功能来设置 scope；后续，如果项目有变动或者有新功能，我们可以再用追加的方式添加新的 scope。

这里想强调的是，scope 不适合设置太具体的值。太具体的话，一方面会导致项目有太多的 scope，难以维护。另一方面，开发者也难以确定 commit 属于哪个具体的 scope，导致错放 scope，反而会使 scope 失去了分类的意义。

当然了，在指定 scope 时，也需要遵循我们预先规划的 scope，所以我们要将 scope 文档化，放在类似 devel 这类文档中。

####  4.1.3 subject
subject 是 commit 的简短描述，必须以动词开头、使用现在时。比如，我们可以用 change，却不能用 changed 或 changes，而且这个动词的第一个字母必须是小写。通过这个动词，我们可以明确地知道 commit 所执行的操作。此外我们还要注意，subject 的结尾不能加英文句号。

### 4.2 Body

Header 对 commit 做了高度概括，可以方便我们查看 Commit Message。那我们如何知道具体做了哪些变更呢？答案就是，可以通过 Body 部分，它是对本次 commit 的更详细描述，是可选的。Body 部分可以分成多行，而且格式也比较自由。不过，和 Header 里的一样，它也要以动词开头，使用现在时。此外，它还必须要包括修改的动机，以及和跟上一版本相比的改动点。

范例

```bash

The body is mandatory for all commits except for those of scope "docs". When the body is required it must be at least 20 characters long.
```
### 4.3 Footer
Footer 部分不是必选的，可以根据需要来选择，主要用来说明本次 commit 导致的后果。在实际应用中，`Footer` 通常用来说明不兼容的改动和关闭的 `Issue` 列表，格式如下：

```bash

BREAKING CHANGE: <breaking change summary>
// 空行
<breaking change description + migration instructions>
// 空行
// 空行
Fixes #<issue number>
```
接下来，我给你详细说明下这两种情况：

- 兼容的改动：如果当前代码跟上一个版本不兼容，需要在 Footer 部分，以 BREAKING CHANG: 开头，后面跟上不兼容改动的摘要。Footer 的其他部分需要说明变动的描述、变动的理由和迁移方法，例如：

```bash

BREAKING CHANGE: isolate scope bindings definition has changed and
    the inject option for the directive controller injection was removed.

    To migrate the code follow the example below:

    Before:

    scope: {
      myAttr: 'attribute',
    }

    After:

    scope: {
      myAttr: '@',
    }
    The removed `inject` wasn't generaly useful for directives so there should be no code using it.
```

- 关闭的 Issue 列表：关闭的 Bug 需要在 Footer 部分新建一行，并以 `Closes` 开头列出，例如：`Closes #123`。如果关闭了多个 Issue，可以这样列出：Closes #123, #432, #886。例如:

```bash

 Change pause version value to a constant for image
    
    Closes #1137
```
### 5. Revert Commit
除了 `Header`、`Body` 和 `Footer` 这 3 个部分，Commit Message 还有一种特殊情况：如果当前 commit 还原了先前的 commit，则应以 `revert`: 开头，后跟还原的 commit 的 Header。而且，在 Body 中必须写成 `This reverts commit <hash>` ，其中 hash 是要还原的 commit 的 `SHA` 标识。例如：

```bash

revert: feat(iam-apiserver): add 'Host' option

This reverts commit 079360c7cfc830ea8a6e13f4c8b8114febc9b48a.
```
了更好地遵循 Angular 规范，建议你在提交代码时养成不用 `git commit -m`，即不用 `-m` 选项的习惯，而是直接用 `git commit` 或者 `git commit -a` 进入交互界面编辑 Commit Message。这样可以更好地格式化 Commit Message。但是除了 Commit Message 规范之外，在代码提交时，我们还需要关注 3 个重点内容：**提交频率**、**合并提交**和 **Commit Message 修改**。


## 6. 提交频率
在实际项目开发中，如果是个人项目，随意 commit 可能影响不大，但如果是多人开发的项目，随意 commit 不仅会让 Commit Message 变得难以理解，还会让其他研发同事觉得你不专业。因此，我们要规定 commit 的提交频率。
**那到底什么时候进行 commit 最好呢？**

- 一种情况是，只要我对项目进行了修改，一通过测试就立即 commit。比如修复完一个 bug、开发完一个小功能，或者开发完一个完整的功能，测试通过后就提交。
- 另一种情况是，我们规定一个时间，定期提交。这里我建议代码下班前固定提交一次，并且要确保本地未提交的代码，延期不超过 1 天。这样，如果本地代码丢失，可以尽可能减少丢失的代码量。

按照上面 2 种方式提交代码，你可能会觉得代码 commit 比较多，看起来比较随意。或者说，我们想等开发完一个完整的功能之后，放在一个 commit 中一起提交。这时候，我们可以在最后合并代码或者提交 Pull Request 前，执行 `git rebase -i` 合并之前的所有 commit。

**那么如何合并 commit 呢？**


## 7. 合并提交
合并提交，就是将多个 commit 合并为一个 commit 提交。这里，我建议你把新的 commit 合并到主干时，只保留 2~3 个 commit 记录。那具体怎么做呢？在 Git 中，我们主要使用 `git rebase` 命令来合并。`git rebase` 也是我们日后开发需要经常使用的一个命令，所以我们一定要掌握好它的使用方法。

###  7.1 git rebase 命令介绍
git rebase 的最大作用是它可以重写历史。
我们通常会通过 `git rebase -i <commit ID>`使用 git rebase 命令，-i 参数表示交互（interactive），该命令会进入到一个交互界面中，其实就是 Vim 编辑器。在该界面中，我们可以对里面的 commit 做一些操作，交互界面如图所示：
![在这里插入图片描述](https://img-blog.csdnimg.cn/829bd15a4d7f4e71b8912636b80c6b12.png)
这个交互界面会首先列出给定`<commit ID>`之前（不包括，越下面越新）的所有 commit，每个 commit 前面有一个操作命令，默认是 `pick`。我们可以选择不同的 commit，并修改 commit 前面的命令，来对该 commit 执行不同的变更操作。

`git rebase` 支持的变更操作如下：
![在这里插入图片描述](https://img-blog.csdnimg.cn/52b472cb035d41f59fa5b910576d9305.png)
在上面的 7 个命令中，`squash` 和 `fixup` 可以用来合并 commit。例如用 squash 来合并，我们只需要把要合并的 commit 前面的动词，改成 squash（或者 s）即可。你可以看看下面的示例：


```bash
pick 07c5abd Introduce OpenPGP and teach basic usage
s de9b1eb Fix PostChecker::Post#urls
s 3e7ee36 Hey kids, stop all the highlighting
pick fa20af3 git interactive rebase, squash, amend
```
`rebase` 后，第 2 行和第 3 行的 `commit` 都会合并到第 1 行的 `commit`。这个时候，我们提交的信息会同时包含这三个 commit 的提交信息：

```bash
# This is a combination of 3 commits.
# The first commit's message is:
Introduce OpenPGP and teach basic usage

# This is the 2ndCommit Message:
Fix PostChecker::Post#urls

# This is the 3rdCommit Message:
Hey kids, stop all the highlighting
```
如果我们将第 3 行的 `squash` 命令改成 `fixup` 命令：

```bash

pick 07c5abd Introduce OpenPGP and teach basic usage
s de9b1eb Fix PostChecker::Post#urls
f 3e7ee36 Hey kids, stop all the highlighting
pick fa20af3 git interactive rebase, squash, amend
```
rebase 后，第 2 行和第 3 行的 commit 都会合并到第 1 行的 commit。这个时候，我们提交的信息会同时包含这三个 commit 的提交信息：

```bash

# This is a combination of 3 commits.
# The first commit's message is:
Introduce OpenPGP and teach basic usage

# This is the 2ndCommit Message:
Fix PostChecker::Post#urls

# This is the 3rdCommit Message:
Hey kids, stop all the highlighting
```
如果我们将第 3 行的 squash 命令改成 fixup 命令：

```bash

pick 07c5abd Introduce OpenPGP and teach basic usage
s de9b1eb Fix PostChecker::Post#urls
f 3e7ee36 Hey kids, stop all the highlighting
pick fa20af3 git interactive rebase, squash, amend
```
rebase 后，还是会生成两个 commit，第 2 行和第 3 行的 commit，都合并到第 1 行的 commit。但是，新的提交信息里面，第 3 行 commit 的提交信息会被注释掉：

```bash

# This is a combination of 3 commits.
# The first commit's message is:
Introduce OpenPGP and teach basic usage

# This is the 2ndCommit Message:
Fix PostChecker::Post#urls

# This is the 3rdCommit Message:
# Hey kids, stop all the highlighting
```
- 删除某个 commit 行，则该 commit 会丢失掉。
- 删除所有的 commit 行，则 rebase 会被终止掉。
- 可以对 commits 进行排序，git 会从上到下进行合并。

### 7.2 合并提交操作示例
假设我们需要研发一个新的模块：user，用来在平台里进行用户的注册、登录、注销等操作，当模块完成开发和测试后，需要合并到主干分支，具体步骤如下。

首先，我们新建一个分支。我们需要先基于 master 分支新建并切换到 feature 分支：

```bash

$ git checkout -b feature/user
Switched to a new branch 'feature/user'
```
这是我们的所有 commit 历史：

```bash

$ git log --oneline
7157e9e docs(docs): append test line 'update3' to README.md
5a26aa2 docs(docs): append test line 'update2' to README.md
55892fa docs(docs): append test line 'update1' to README.md
89651d4 docs(doc): add README.md
```
接着，我们在 `feature/user`分支进行功能的开发和测试，并遵循规范提交 commit，功能开发并测试完成后，Git 仓库的 commit 记录如下：

```bash

$ git log --oneline
4ee51d6 docs(user): update user/README.md
176ba5d docs(user): update user/README.md
5e829f8 docs(user): add README.md for user
f40929f feat(user): add delete user function
fc70a21 feat(user): add create user function
7157e9e docs(docs): append test line 'update3' to README.md
5a26aa2 docs(docs): append test line 'update2' to README.md
55892fa docs(docs): append test line 'update1' to README.md
89651d4 docs(doc): add README.md
```
可以看到我们提交了 5 个 commit。接下来，我们需要将 `feature/user`分支的改动合并到 master 分支，但是 5 个 commit 太多了，我们想将这些 commit 合并后再提交到 master 分支。

接着，我们合并所有 commit。在上一步中，我们知道 `fc70a21`是 feature/user分支的第一个 commit ID，其父 commit ID 是 `7157e9e`，我们需要将7157e9e之前的所有分支 进行合并，这时我们可以执行：

```bash

$ git rebase -i 7157e9e
```
执行命令后，我们会进入到一个交互界面，在该界面中，我们可以将需要合并的 4 个 commit，都执行 squash 操作，如下图所示：
![在这里插入图片描述](https://img-blog.csdnimg.cn/85dac6886fd045feabb2022892acb87e.png)
修改完成后执行:wq 保存，会跳转到一个新的交互页面，在该页面，我们可以编辑 Commit Message，编辑后的内容如下图所示：
![在这里插入图片描述](https://img-blog.csdnimg.cn/fb6db662526649849283a765aa4478dd.png)
#开头的行是 git 的注释，我们可以忽略掉，在 `rebase` 后，这些行将会消失掉。修改完成后执行`:wq` 保存，就完成了合并提交操作。

除此之外，这里有 2 个点需要我们注意：

- `git rebase -i <commid ID>`这里的一定要是需要合并 commit 中最旧 commit 的父 commit ID。
- 我们希望将 `feature/user` 分支的 5 个 commit 合并到一个 commit，在 `git rebase` 时，需要保证其中最新的一个 commit 是 `pick` 状态，这样我们才可以将其他 4 个 commit 合并进去。

然后，我们用如下命令来检查 commits 是否成功合并。可以看到，我们成功将 5 个 commit 合并成为了一个 commit：`d6b17e0`。

```bash

$ git log --oneline
d6b17e0 feat(user): add user module with all function implements
7157e9e docs(docs): append test line 'update3' to README.md
5a26aa2 docs(docs): append test line 'update2' to README.md
55892fa docs(docs): append test line 'update1' to README.md
89651d4 docs(doc): add README.md
```
最后，我们就可以将 feature 分支 `feature/user` 的改动合并到主干分支，从而完成新功能的开发。

```bash

$ git checkout master
$ git merge feature/user
$ git log --oneline
d6b17e0 feat(user): add user module with all function implements
7157e9e docs(docs): append test line 'update3' to README.md
5a26aa2 docs(docs): append test line 'update2' to README.md
55892fa docs(docs): append test line 'update1' to README.md
89651d4 docs(doc): add README.md
```
这里给你一个小提示，如果你有太多的 `commit` 需要合并，那么可以试试这种方式：先撤销过去的 commit，然后再建一个新的。

## 8. 修改 Commit Message
即使我们有了 Commit Message 规范，但仍然可能会遇到提交的 Commit Message 不符合规范的情况，这个时候就需要我们能够修改之前某次 commit 的 Commit Message。具体来说，我们有两种修改方法，分别对应两种不同情况：

- `git commit --amend`：修改最近一次 commit 的 message；
- `git rebase -i`：修改某次 commit 的 message。

接下来，我们分别来说这两种方法。

### 8.1 git commit --amend：**修改最近一次 commit 的 message**

有时候，我们刚提交完一个 commit，但是发现 commit 的描述不符合规范或者需要纠正，这时候，我们可以通过 `git commit --amend` 命令来修改刚刚提交 commit 的 Commit Message。具体修改步骤如下：

1. 查看当前分支的日志记录。

```bash

$ git log –oneline
418bd4 docs(docs): append test line 'update$i' to README.md
89651d4 docs(doc): add README.md
```
可以看到，最近一次的 Commit Message 是 `docs(docs): append test line 'update$i' to README.md`，其中 `update$i` 正常应该是 `update1`。


2. 更新最近一次提交的 Commit Message
在当前 Git 仓库下执行命令：`git commit --amend`，后会进入一个交互界面，在交互界面中，修改最近一次的 Commit Message，如下图所示：

![在这里插入图片描述](https://img-blog.csdnimg.cn/ecc8b24c6f614b3e98aed10426646df7.png)
修改完成后执行`:wq` 保存，退出编辑器之后，会在命令行显示，该 commit 的 message 的更新结果如下：

```bash

[master 55892fa] docs(docs): append test line 'update1' to README.md
 Date: Fri Sep 18 13:40:42 2020 +0800
 1 file changed, 1 insertion(+)
```
查看最近一次的 Commit Message 是否被更新

```bash

$ git log --oneline
55892fa docs(docs): append test line 'update1' to README.md
89651d4 docs(doc): add README.md
```
可以看到最近一次 commit 的 message 成功被修改为期望的内容。


### 8.2 git rebase -i：修改某次 commit 的 message
如果我们想修改的 Commit Message 不是最近一次的 Commit Message，可以通过 `git rebase -i <父 commit ID>`命令来修改。这个命令在实际开发中使用频率比较高，我们一定要掌握。具体来说，使用它主要分为 4 步。

1. 查看当前分支的日志记录。

```bash
$ git log --oneline
1d6289f docs(docs): append test line 'update3' to README.md
a38f808 docs(docs): append test line 'update$i' to README.md
55892fa docs(docs): append test line 'update1' to README.md
89651d4 docs(doc): add README.md
```
可以看到倒数第 3 次提交的 Commit Message 是：`docs(docs): append test line 'update$i' to README.md，其中 update$i` 正常应该是 `update2`。

2. 修改倒数第 3 次提交 commit 的 message。
在 Git 仓库下直接执行命令 `git rebase -i 55892fa`，然后会进入一个交互界面。在交互界面中，修改最近一次的 Commit Message。这里我们使用 `reword` 或者 `r`，保留倒数第 3 次的变更信息，但是修改其 message，如下图所示：
![在这里插入图片描述](https://img-blog.csdnimg.cn/d8fcaa159e0141408197fecb23a5e017.png)
修改完成后执行`:wq` 保存，还会跳转到一个新的交互页面，如下图所示：
![在这里插入图片描述](https://img-blog.csdnimg.cn/bf0aea620c3f4d7389b4a822d49a6861.png)
修改完成后执行`:wq` 保存，退出编辑器之后，会在命令行显示该 commit 的 message 的更新结果：


```bash
[detached HEAD 5a26aa2] docs(docs): append test line 'update2' to README.md
 Date: Fri Sep 18 13:45:54 2020 +0800
 1 file changed, 1 insertion(+)
Successfully rebased and updated refs/heads/master.
```
`Successfully rebased and updated refs/heads/master.`说明 `rebase` 成功，其实这里完成了两个步骤：更新 message，更新该 commit 的 HEAD 指针。

> 注意：这里一定要传入想要变更 Commit Message 的父 commit ID：git rebase -i <父 commit ID>。

3.  查看倒数第 3 次 commit 的 message 是否被更新。

这里有两点需要你注意：

- Commit Message 是 commit 数据结构中的一个属性，如果 Commit Message 有变更，则 commit ID 一定会变，`git commit --amend` 只会变更最近一次的 commit ID，但是 git rebase -i 会变更父 commit ID 之后所有提交的 commit ID。
- 如果当前分支有未 commit 的代码，需要先执行 `git stash` 将工作状态进行暂存，当修改完成后再执行 `git stash pop` 恢复之前的工作状态。

## 9. Commit Message 规范自动化
其实，到这里我们也就意识到了一点：Commit Message 规范如果靠文档去约束，就会严重依赖开发者的代码素养，并不能真正保证提交的 commit 是符合规范的。那么，有没有一种方式可以确保我们提交的 Commit Message 一定是符合规范的呢？有的，我们可以通过一些工具，来自动化地生成和检查 Commit Message 是否符合规范。另外，既然 Commit Message 是规范的，那么我们能不能利用这些规范来实现一些更酷的功能呢？答案是有的，我将可以围绕着 Commit Message 实现的一些自动化功能绘制成了下面一张图。

![在这里插入图片描述](https://img-blog.csdnimg.cn/d5712a0d3dca49719f3b2e1fa53b8e08.png)
这些自动化功能可以分为以下 2 类：

- Commit Message 生成和检查功能：生成符合 Angular 规范的 Commit Message、Commit Message 提交前检查、历史 Commit Message 检查。
- 基于 Commit Message 自动生成 `CHANGELOG` 和 `SemVer` 的工具。

我们可以通过下面这 5 个工具自动的完成上面的功能：

- [commitizen-go](https://github.com/lintingzhen/commitizen-go)：使你进入交互模式，并根据提示生成 Commit Message，然后提交。
- commit-msg：githooks，在 commit-msg 中，指定检查的规则，commit-msg 是个脚本，可以根据需要自己写脚本实现。这门课的 commit-msg 调用了 go-gitlint 来进行检查。
- [go-gitlint](https://github.com/llorllale/go-gitlint)：检查历史提交的 Commit Message 是否符合 Angular 规范，可以将该工具添加在 CI 流程中，确保 Commit Message 都是符合规范的。
- [gsemver](https://github.com/arnaud-deprez/gsemver)：语义化版本自动生成工具。
- [git-chglog](https://github.com/git-chglog/git-chglog)：根据 Commit Message 生成 CHANGELOG。


## 10. 总结

今天我向你介绍了 Commit Message 规范，主要讲了业界使用最多的 Angular 规范。

Angular 规范中，Commit Message 包含三个部分：`Header`、`Body` 和 `Footer`。Header 对 commit 做了高度概括，Body 部分是对本次 commit 的更详细描述，Footer 部分主要用来说明本次 commit 导致的后果。格式如下：

```bash

<type>[optional scope]: <description>
// 空行
[optional body]
// 空行
[optional footer(s)]
```
另外，我们也需要控制 commit 的提交频率，比如可以在开发完一个功能、修复完一个 bug、下班前提交 commit。

后，我们也需要掌握一些常见的提交操作，例如通过 `git rebase -i` 来合并提交 commit，通过 `git commit --amend` 或 `git rebase -i` 来修改 commit message。


## 11.  练习
- 新建一个 `git repository`，提交 4 个符合 Angular 规范的 Commit Message，并合并前 2 次提交。
- 使用 [git-chglog](https://github.com/git-chglog/git-chglog) 工具来生成 CHANGEOG，使用 [gsemver](https://github.com/arnaud-deprez/gsemver) 工具来生成语义化版本号。

### 11.1 第一题
1. 创建git-test项目
![在这里插入图片描述](https://img-blog.csdnimg.cn/b2e80e22670c4e16b52112191b1c4d86.png)
2. 本地安装git
配置yum源
```bash
# mv /etc/yum.repos.d /etc/yum.repos.d.bak # 先备份原有的 Yum 源
# mkdir /etc/yum.repos.d
# wget -O /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-vault-8.5.2111.repo
# yum clean all && yum makecache
```
安装依赖

```bash
$ sudo yum -y install make autoconf automake cmake perl-CPAN libcurl-devel libtool gcc gcc-c++ glibc-headers zlib-devel git-lfs telnet lrzsz jq expat-devel openssl-devel
```
安装 Git

```bash

$ cd /tmp
$ wget --no-check-certificate https://mirrors.edge.kernel.org/pub/software/scm/git/git-2.36.1.tar.gz
$ tar -xvzf git-2.36.1.tar.gz
$ cd git-2.36.1/
$ ./configure
$ make
$ sudo make install
$ git --version          # 输出 git 版本号，说明安装成功
git version 2.36.1
```

```bash

tee -a $HOME/.bashrc <<'EOF'
# Configure for git
export PATH=/usr/local/libexec/git-core:$PATH
EOF
```

3. 配置互信

```bash

$ git config --global user.name "ghostwritten"    # 用户名改成自己的
$ git config --global user.email "1zoxun1@gmail.com"    # 邮箱改成自己的
$ git config --global credential.helper store    # 设置 Git，保存用户名和密码
$ git config --global core.longpaths true # 解决 Git 中 'Filename too long' 的错误
$ git config --list

$ git config --global core.quotepath off
$ git lfs install --skip-repo
ssh-keygen -t rsa -C "1zoxun1@gmail.com"

$ cat /root/.ssh/id_rsa.pub
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDjE+yheOYdN0YfgqzuS1PyTqwdUAbP3C7iP1zuWFutdaS5RCRNaljusudgE2BQ//ttuAg/CjecNFq9HoM320pU9jO6kq1ZpK6limdZUz7a5LWMMpfP6/i+oJYf8okwajbVXz80zkGN0P/I6F+/uiThOeN0fKboYHiRLvI44AdvUvO0nhzqUxC05umprIVfsf/p/JnS2FUsMd5rfzkLonCeMzfW19dhf1wqV5MOD+JsTdoog32Ir9Iwlpgu+o2TiuVVI/BMUqv92wJyGKngeOaAmKmf6zeqGfWvJpjcorgVZiLiOyomGw3AbwLSntjGq9KrSviULZWGLcigKFr7oyvcOE2Qv4F2wv2cVJBrXA3wMjpCt4wOPksk8v5HGOIsG7F5AQcIV/dLn01z/RIrdVzUrr9g8EXtwyNfpJ+15lKu5+de3tWb2ZHWncqoY50G7oeBnC5S9aKRwitkPhuodfPcaPmYPbfysimY9GMbeUEX8ynnZMkTzpDXXD4KP3G1YQ0= 1zoxun1@gmail.com
```
拷贝公钥至github ssh
![在这里插入图片描述](https://img-blog.csdnimg.cn/2a3a67b3590d4740bbe410cce2d7d3d5.png)

4.  测试添加第一个 commit

```bash
$ echo "# git-test" >> README.md
$ git add .
$  git commit -m "create README.md"
[main (root-commit) 9927ec0] create README.md
 1 file changed, 1 insertion(+)
 create mode 100644 README.md

$ git log --oneline
9927ec0 (HEAD -> main) create README.md
```


5. 添加第二个commit

```bash
$ echo "this is a test1!" >> README.md
$ cat README.md
# git-test
this is a test1!
$ git add .
$ git commit -m "append test line 'test1' to README.md"
[main 6a79e97] append test line 'test1' to README.md
 1 file changed, 2 insertions(+)
$ git log --oneline
6a79e97 (HEAD -> main) append test line 'test1' to README.md
9927ec0 create README.md
```
6. 添加第三个commit，我们尝试故意写错commit，然后通过`git commit amend`改正。

```bash
$ echo "this is a test2!" >> README.md
$ cat README.md
# git-test
this is a test1!
this is a test2!
$ git add .
$ git commit -m "append test line 'test$' to README.md"  # 把test2 错写成了test$
$ git log --oneline
0834d4a (HEAD -> main) append test line 'test$' to README.md
6a79e97 append test line 'test1' to README.md
9927ec0 create README.md


$ git commit --amend
#append test line 'test$' to README.md   #把test$ 改成test1
append test line 'test1' to README.md

# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
#
# Date:      Fri Nov 11 15:59:44 2022 +0800
#
# On branch main
# Your branch is based on 'origin/main', but the upstream is gone.
#   (use "git branch --unset-upstream" to fixup)
#
# Changes to be committed:
#       modified:   README.md
#
```
wq!保存退出，会自动显示：

```bash
[main dd07581] append test line 'test1' to README.md
 Date: Fri Nov 11 15:59:44 2022 +0800
 1 file changed, 1 insertion(+)
```
查看git log

```bash
4 git log --oneline
dd07581 (HEAD -> main) append test line 'test1' to README.md
6a79e97 append test line 'test1' to README.md
9927ec0 create README.md
```

7. 添加第四个commit，我们通过git commit 交互写commit

```bash
$ echo "this is a test3!" >> README.md
$ git add .
$  git commit
# 第一行添加commit 内容
append test line 'test3' to README.md  

# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
#
# Date:      Fri Nov 11 15:59:44 2022 +0800
#
# On branch main
# Your branch is based on 'origin/main', but the upstream is gone.
#   (use "git branch --unset-upstream" to fixup)
#
# Changes to be committed:
#       modified:   README.md
#

```
wq！保存退出，会自动显示“

```bash
[main ce9b6fb] append test line 'test3' to README.md
 1 file changed, 1 insertion(+)
```
查看日志

```bash
$ git log --oneline
ce9b6fb (HEAD -> main) append test line 'test3' to README.md
dd07581 append test line 'test1' to README.md
6a79e97 append test line 'test1' to README.md
9927ec0 create README.md
```

8.  我们通过`git rebase -i` 修改第三次添加的内容，test2写成了`test1`

> 注意，这里一定要传入想要变更 Commit Message 的父 commit ID：`git rebase -i <父 commit ID>`。

```bash
pick 6a79e97 append test line 'test1' to README.md
r dd07581 append test line 'test2' to README.md   # 把pick改成r（意思是保留commit信息，并修改commit信息）
pick ce9b6fb append test line 'test3' to README.md

# Rebase 9927ec0..ce9b6fb onto 9927ec0 (3 commands)
#
# Commands:
# p, pick <commit> = use commit
# r, reword <commit> = use commit, but edit the commit message
# e, edit <commit> = use commit, but stop for amending
# s, squash <commit> = use commit, but meld into previous commit
# f, fixup [-C | -c] <commit> = like "squash" but keep only the previous
#                    commit's log message, unless -C is used, in which case
#                    keep only this commit's message; -c is same as -C but
#                    opens the editor
# x, exec <command> = run command (the rest of the line) using shell
# b, break = stop here (continue rebase later with 'git rebase --continue')
# d, drop <commit> = remove commit
# l, label <label> = label current HEAD with a name
# t, reset <label> = reset HEAD to a label
# m, merge [-C <commit> | -c <commit>] <label> [# <oneline>]
# .       create a merge commit using the original merge commit's
# .       message (or the oneline, if no original merge commit was
# .       specified); use -c <commit> to reword the commit message
#
# These lines can be re-ordered; they are executed from top to bottom.
#
# If you remove a line here THAT COMMIT WILL BE LOST.
#
```
：wq保存，继续弹出

```bash
append test line 'test2' to README.md  #修改此行

# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
#
# Date:      Fri Nov 11 15:59:44 2022 +0800
#
# interactive rebase in progress; onto 9927ec0
# Last commands done (2 commands done):
#    pick 6a79e97 append test line 'test1' to README.md
#    reword 50b6bb0 append test line 'test2' to README.md
# Next command to do (1 remaining command):
#    pick 2bc13f2 append test line 'test3' to README.md
# You are currently editing a commit while rebasing branch 'main' on '9927ec0'.
#
# Changes to be committed:
#       modified:   README.md
```
最后`:wq`保存退出,会自动弹出

```bash
[detached HEAD 45a5d2b] append test line 'test2' to README.md
 Date: Fri Nov 11 15:59:44 2022 +0800
 1 file changed, 1 insertion(+)
Successfully rebased and updated refs/heads/main.
```
查看git日志

```bash
 git log --oneline
f8300a8 (HEAD -> main) append test line 'test3' to README.md
45a5d2b append test line 'test2' to README.md   #已经改过来了。
6a79e97 append test line 'test1' to README.md
9927ec0 create README.md
```

8.  我们进行合并

创建一个分支
```bash
$ git checkout -b feature/user
Switched to a new branch 'feature/user'

#这是我们当前的提交历史
$ git log --oneline
cdc05c6 (HEAD -> feature/user, main) append test line 'test3' to README.md
053fa91 append test line 'test2' to README.md
6a79e97 append test line 'test1' to README.md
9927ec0 create README.md
```
接着，我们在 `feature/user`分支进行功能的开发和测试，并遵循规范提交 commit，功能开发并测试完成后，Git 仓库的 commit 记录如下：

```bash
$ git log --oneline
efa9b1c (HEAD -> feature/user) docs(user): update user/README.md
bec209f docs(user): update user/README.md
9f69521 docs(user): add README.md for user
2fc667e feat(user): add delete user function
c2fda9f feat(user): add create user function
cdc05c6 (main) append test line 'test3' to README.md
053fa91 append test line 'test22' to README.md
6a79e97 append test line 'test1' to README.md
9927ec0 create README.md
```
可以看到我们提交了 5 个 commit。接下来，我们需要将 feature/user分支的改动合并到 master 分支，但是 5 个 commit 太多了，我们想将这些 commit 合并后再提交到 master 分支。

接着，我们合并所有 commit。在上一步中，我们知道`c2fda9f`是 feature/user分支的第一个 commit ID，其父 commit ID 是`cdc05c6`，我们需要将`cdc05c6`之前的所有分支 进行合并，这时我们可以执行：


```bash
$ git rebase -i cdc05c6
```

执行命令后，我们会进入到一个交互界面，在该界面中，我们可以将需要合并的 4 个 commit，都执行 squash 操作，如下图所示：

```bash
pick c2fda9f feat(user): add create user function
s 2fc667e feat(user): add delete user function  #pick 改成s，squash 操作，保留该commit，但会和上一个commit合并
s 9f69521 docs(user): add README.md for user  #pick 改成s，squash 操作，保留该commit，但会和上一个commit合并
s bec209f docs(user): update user/README.md  #pick 改成s，squash 操作，保留该commit，但会和上一个commit合并
s efa9b1c docs(user): update user/README.md #pick 改成s，squash 操作，保留该commit，但会和上一个commit合并

# Rebase cdc05c6..efa9b1c onto cdc05c6 (5 commands)
```
修改完成后执行`:wq` 保存，会跳转到一个新的交互页面，在该页面，我们可以编辑 Commit Message，

```bash
# This is a combination of 5 commits.
# This is the 1st commit message:

feat(user): add create user function

# This is the commit message #2:

feat(user): add delete user function

# This is the commit message #3:

docs(user): add README.md for user

# This is the commit message #4:

docs(user): update user/README.md

# This is the commit message #5:

docs(user): update user/README.md
```

编辑后的内容如下图所示：

```bash
# This is the 1st commit message:

feat(user): add create user module with all function implements

do the following updates:
1. create User go struct
2. (u *User) create() function
3. add (u *User) Delete() function
4. add README.md for user module
```
开头的行是 git 的注释，我们可以忽略掉，在 rebase 后，这些行将会消失掉。修改完成后执行:wq 保存，就完成了合并提交操作。

查看git 日志

```bash
$ git log --oneline
3650b6c (HEAD -> feature/user) feat(user): add create user module with all function implements
cdc05c6 (main) append test line 'test3' to README.md
053fa91 append test line 'test22' to README.md
6a79e97 append test line 'test1' to README.md
9927ec0 create README.md
```
这样就完成了五次commit 的合并。


### 11.2 第二题

```bash
get https://github.com/git-chglog/git-chglog/releases/download/v0.15.1/git-chglog_0.15.1_linux_amd64.tar.gz
tar -zxvf git-chglog_0.15.1_linux_amd64.tar.gz
mv git-chglog /usr/local/bin/
git-chglog -version
```

```bash
git-chglog --init
? What is the URL of your repository? https://github.com/Ghostwritten/git-test
? What is your favorite style? github
? Choose the format of your favorite commit message <type>(<scope>): <subject>
? What is your favorite template style? keep-a-changelog
? Do you include Merge Commit in CHANGELOG? Yes
? Do you include Revert Commit in CHANGELOG? Yes
? In which directory do you output configuration files and templates? .chglog

✨  Configuration file and template generation completed!
  ✔ .chglog/config.yml
  ✔ .chglog/CHANGELOG.tpl.md

$ ls .chglog/
CHANGELOG.tpl.md  config.yml
```

```bash
$ cat .chglog/CHANGELOG.tpl.md
{{ if .Versions -}}
<a name="unreleased"></a>
## [Unreleased]

{{ if .Unreleased.CommitGroups -}}
{{ range .Unreleased.CommitGroups -}}
### {{ .Title }}
{{ range .Commits -}}
- {{ if .Scope }}**{{ .Scope }}:** {{ end }}{{ .Subject }}
{{ end }}
{{ end -}}
{{ end -}}
{{ end -}}

{{ range .Versions }}
<a name="{{ .Tag.Name }}"></a>
## {{ if .Tag.Previous }}[{{ .Tag.Name }}]{{ else }}{{ .Tag.Name }}{{ end }} - {{ datetime "2006-01-02" .Tag.Date }}
{{ range .CommitGroups -}}
### {{ .Title }}
{{ range .Commits -}}
- {{ if .Scope }}**{{ .Scope }}:** {{ end }}{{ .Subject }}
{{ end }}
{{ end -}}

{{- if .RevertCommits -}}
### Reverts
{{ range .RevertCommits -}}
- {{ .Revert.Header }}
{{ end }}
{{ end -}}

{{- if .MergeCommits -}}
### Pull Requests
{{ range .MergeCommits -}}
- {{ .Header }}
{{ end }}
{{ end -}}

{{- if .NoteGroups -}}
{{ range .NoteGroups -}}
### {{ .Title }}
{{ range .Notes }}
{{ .Body }}
{{ end }}
{{ end -}}
{{ end -}}
{{ end -}}

{{- if .Versions }}
[Unreleased]: {{ .Info.RepositoryURL }}/compare/{{ $latest := index .Versions 0 }}{{ $latest.Tag.Name }}...HEAD
{{ range .Versions -}}
{{ if .Tag.Previous -}}
[{{ .Tag.Name }}]: {{ $.Info.RepositoryURL }}/compare/{{ .Tag.Previous.Name }}...{{ .Tag.Name }}
{{ end -}}
{{ end -}}
{{ end -}}


$ cat .chglog/config.yml
style: github
template: CHANGELOG.tpl.md
info:
  title: CHANGELOG
  repository_url: https://github.com/Ghostwritten/git-test
options:
  commits:
    # filters:
    #   Type:
    #     - feat
    #     - fix
    #     - perf
    #     - refactor
  commit_groups:
    # title_maps:
    #   feat: Features
    #   fix: Bug Fixes
    #   perf: Performance Improvements
    #   refactor: Code Refactoring
  header:
    pattern: "^(\\w*)(?:\\(([\\w\\$\\.\\-\\*\\s]*)\\))?\\:\\s(.*)$"
    pattern_maps:
      - Type
      - Scope
      - Subject
  notes:
    keywords:
      - BREAKING CHANGE
```

