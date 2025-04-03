#  Github Package npm 应用发布实践

![](https://i-blog.csdnimg.cn/blog_migrate/048984ee94161c3c89c1d0da7aae35db.png)


## 1. 简介
[GitHub Packages](https://docs.github.com/zh/packages) 是一个用于托管和管理包的平台，包括容器和其他依赖项。 GitHub Packages 将源代码和包组合在一起，以提供集成的权限管理和计费，使你能够在 GitHub 上专注于软件开发。

您可以将 GitHub Packages 与 [GitHub API](https://docs.github.com/en/rest?apiVersion=2022-11-28)、[GitHub Actions](https://github.com/features/actions) 以及 web 挂钩集成在一起，以创建端到端的 DevOps 工作流程，其中包括您的代码、CI 和部署解决方案。

GitHub Packages 为常用的包管理器提供不同的包仓库，例如 npm、RubyGems、Apache Maven、Gradle、Docker 和 Nuget。 GitHub 的 Container registry 针对容器进行了优化，支持 Docker 和 OCI 映像。

今天，我尝试实现 Github Package npm 应用发布实践。

## 2.  创建新库 
名字：`github-packages-npm-demo`
![](https://i-blog.csdnimg.cn/blog_migrate/c8c4bc3854dd43752ca934079b931a5a.png)

## 3. 编写 index.js
创建 `index.js` 文件，并添加指示“Hello world!”的基本警报

```bash
console.log("Hello, World!");
```
## 4. npm init 初始化
使用 `npm init` 初始化 npm 包。 在包初始化向导中，输入名称为 `@YOUR-USERNAME/YOUR-REPOSITORY` 的包，并将测试脚本设置为 `exit 0`。 这将生成一个 `package.json` 文件，其中包含有关包的信息。

> 如何安装 npm，可以[参考这篇文章](https://blog.csdn.net/xixihahalelehehe/article/details/122587741)。

```bash
$ npm init
This utility will walk you through creating a package.json file.
It only covers the most common items, and tries to guess sensible defaults.

See `npm help init` for definitive documentation on these fields
and exactly what they do.

Use `npm install <pkg>` afterwards to install a package and
save it as a dependency in the package.json file.

Press ^C at any time to quit.
package name: (github-packages-npm-demo) @ghostwritten/github-packages-npm-demo
version: (1.0.0)
description: github packages npm demo
entry point: (index.js)
test command: exit 0
git repository: (https://github.com/Ghostwritten/github-packages-npm-demo.git)
keywords:
author: ghostwritten
license: (ISC)
About to write to /root/github/github-packages-npm-demo/package.json:

{
  "name": "@ghostwritten/github-packages-npm-demo",
  "version": "1.0.0",
  "description": "github packages npm demo",
  "main": "index.js",
  "scripts": {
    "test": "exit 0"
  },
  "repository": {
    "type": "git",
    "url": "git+https://github.com/Ghostwritten/github-packages-npm-demo.git"
  },
  "author": "ghostwritten",
  "license": "ISC",
  "bugs": {
    "url": "https://github.com/Ghostwritten/github-packages-npm-demo/issues"
  },
  "homepage": "https://github.com/Ghostwritten/github-packages-npm-demo#readme"
}


Is this OK? (yes) yes

$ ls
index.js  package.json
```
## 5. npm install 安装包
运行 `npm install` 以生成 `package-lock.json` 文件，然后提交更改并将其推送到 GitHub。

```bash
$ npm install
npm notice created a lockfile as package-lock.json. You should commit this file.
up to date in 3.702s
found 0 vulnerabilities



   ╭───────────────────────────────────────────────────────────────╮
   │                                                               │
   │      New major version of npm available! 6.14.12 → 9.2.0      │
   │   Changelog: https://github.com/npm/cli/releases/tag/v9.2.0   │
   │               Run npm install -g npm to update!               │
   │                                                               │
   ╰───────────────────────────────────────────────────────────────╯


$ ls
index.js  package.json  package-lock.json

$ git add index.js package.json package-lock.json
$ git commit -m "initialize npm package"
$ git push
```
## 6. 编写 release-package.yml 

创建 `.github/workflows` 目录。 在此目录中，创建名为 `release-package.yml` 的文件。

```bash
name: Node.js Package

on:
  push:
    branches:
      - main
     
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: 16
      - run: npm ci
      - run: npm test

  publish-gpr:
    needs: build
    runs-on: ubuntu-latest
    permissions:
      packages: write
      contents: read
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: 16
          registry-url: https://npm.pkg.github.com/
      - run: npm ci
      - run: npm publish
        env:
          NODE_AUTH_TOKEN: ${{secrets.GITHUB_TOKEN}}
```

> 如何生成 `TOKEN`，请[参考这篇文章](https://blog.csdn.net/xixihahalelehehe/article/details/120178280)。

## 7.  发布
告诉 NPM 使用以下方法之一发布包的范围和仓库：

- 第一种方法
在根目录中创建包含以下内容的 `.npmrc` 文件，为存储库添加 NPM 配置文件

```bash
@YOUR-USERNAME:registry=https://npm.pkg.github.com
```
- 第二种方法：
编辑 `package.json` 文件并指定 `publishConfig` 密钥：

```bash
"publishConfig": {
   "@YOUR-USERNAME:registry": "https://npm.pkg.github.com"
 }
```

提交并推送更改到 GitHub。

```bash
$ git add .github/workflows/release-package.yml
# Also add the file you created or edited in the previous step.
$ git add .npmrc or package.json
$ git commit -m "workflow to publish package"
$ git push
```
只要您的仓库中创建新版本，您创建的工作流程就会运行。 如果测试通过，则包将发布到 GitHub Packages。

## 8. 查看已发布包
workflow 构建流程![](https://i-blog.csdnimg.cn/blog_migrate/2f4b7e4b833a80d14d69bcece357bbc3.png)

发布的 npm 包
![](https://i-blog.csdnimg.cn/blog_migrate/024babbdb2e740f3307ec211cc5bfd79.png)

## 9. 管理 npm 包
将默认的 `private` 包转为 `public` 包
![](https://i-blog.csdnimg.cn/blog_migrate/7d5c664ca37393ba1d409149ac4d52a1.png)

![](https://i-blog.csdnimg.cn/blog_migrate/720252464001c93278d7c788df15fa1a.png)

参考：
- [GitHub Packages 快速入门](https://docs.github.com/zh/packages/quickstart#next-steps)
