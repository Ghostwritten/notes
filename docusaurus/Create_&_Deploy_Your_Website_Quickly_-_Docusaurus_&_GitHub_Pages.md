![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/bb33df66dadf4bcabe96373c0b79c567.png)





## 1. 简介

Docusaurus 是一个开源框架，用于快速构建、部署和托管技术文档。它支持 Markdown 格式，提供了主题和插件，支持版本控制，适合开发者编写和托管项目文档。在这篇博客中，我们将介绍如何本地安装 Docusaurus 并将其托管到 GitHub Pages 上。

## 2. 准备
本地安装 Docusaurus请参考：

- [Npm Install Docusaurus Demo](https://ghostwritten.blog.csdn.net/article/details/138565544)

或者参考官网：[https://docusaurus.io/docs](https://docusaurus.io/docs)

安装npm git 工具参考：[https://blog.csdn.net/xixihahalelehehe/article/details/138565544](https://blog.csdn.net/xixihahalelehehe/article/details/138565544)


## 3. 创建项目

- github创建 demo 项目。
- 本地vscode 终端执行：`npx create-docusaurus@latest demo classic`

安装依赖包。
```bash
$ cd demo
$ npm i
```


配置`docusaurus.config.js`文件

```bash
  url: 'https://github.com',
  baseUrl: '/demo/',
  organizationName: 'Ghostwritten', // Usually your GitHub org/user name.
  projectName: 'demo', // Usually your repo name.
  deploymentBranch: 'gh-pages',
```

本地启动测试

```bash
npm start
```
访问`http://localhost:3000`

推送代码至仓库

```bash
git init
git add .
git commit -m "website demo"
git branch -M main
git remote add origin https://github.com/Ghostwritten/demo.git
git push -u origin main
```

触发部署到github托管服务。

```bash
GIT_USER=Ghostwritten yarn deploy
```
调试输出执行：

```bash
Edit
GIT_TRACE=1 GIT_CURL_VERBOSE=1 GIT_USER=Ghostwritten yarn deploy
```

访问：`https://ghostwritten.github.io/demo`


## 4. 问题

### 4.1 GIT_USER=Ghostwritten yarn deploy 卡住
认证github问题，创建token

禁用 VS Code 的 askpass 脚本
你可以临时取消 GIT_ASKPASS 环境变量的设置，让 Git 使用标准终端提示来输入密码。执行下面的命令后再运行部署命令：


```bash
unset GIT_ASKPASS
```
使用 GitHub 个人访问令牌（PAT）
由于 GitHub 已经不再支持使用账户密码进行认证，建议生成一个个人访问令牌，并在提示输入密码时用令牌替换：

- 登录 GitHub，进入 `Settings > Developer settings > Personal access tokens`。
- 生成一个新令牌，确保勾选了 repo 等相关权限。
- 在终端提示输入密码时，粘贴你的令牌即可

GIT_USER=Ghostwritten yarn deploy 执行后，交互输入新生成的token。
