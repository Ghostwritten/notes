#  trivy 工具漏洞扫描
tags: trivy,安全
<!-- catalog:trivy 扫描:-->



---

[![在这里插入图片描述](https://img-blog.csdnimg.cn/f4d46ead81ff47a58fb0c860cfee7b62.png#pic_center)](https://www.rottentomatoes.com/tv/for_all_mankind)

上一篇我们[了解 trivy 安装教程](https://blog.csdn.net/xixihahalelehehe/article/details/126019559)，接下来我们熟悉 trviy 命令的场景运用。

[Trivy](https://aquasecurity.github.io/trivy/dev/) 是一个简单而全面的漏洞/错误配置/秘密扫描器，用于容器和其他工件。 检测操作系统包（Alpine、RHEL、CentOS 等）和特定语言包（Bundler、Composer、npm、yarn 等）的漏洞。此外，扫描Terraform 和 Kubernetes 等基础架构即代码 (IaC) 文件，以检测使您的部署面临攻击风险的潜在配置问题。 还扫描硬编码的秘密vyTrivyTrivyTrivy比如密码、API 密钥和令牌。 Trivy易于使用。



##  1. 扫描镜像

```bash
trivy image nginx:1.18.0
trivy image --severity CRITICAL nginx:1.18.0
trivy image --severity CRITICAL, HIGH nginx:1.18.0
trivy image --ignore-unfixed nginx:1.18.0

# Scanning image tarball
docker save nginx:1.18.0 > nginx.tar
trivy image --input archive.tar

# Scan and output results to file
trivy image --output python_alpine.txt python:3.10.0a4-alpine
trivy image --severity HIGH --output /root/python.txt python:3.10.0a4-alpine

# Scan image tarball
trivy image --input alpine.tar --format json --output /root/alpine.json
```

扫描解压镜像文件系统

```bash
$ docker export $(docker create alpine:3.10.2) | tar -C /tmp/rootfs -xvf -
$ trivy rootfs /tmp/rootfs
```
## 2. 嵌入 Dockerfile 扫描
通过将 Trivy 嵌入 Dockerfile 来扫描您的图像作为构建过程的一部分。这种方法可用于更新当前使用 Aqua 的Microscanner的 Dockerfile 
```bash
$ cat Dockerfile
FROM alpine:3.7

RUN apk add curl \
    && curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh | sh -s -- -b /usr/local/bin \
    && trivy rootfs --exit-code 1 --no-progress /

$ docker build -t vulnerable-image .
```
或者，您可以在多阶段构建中使用 Trivy。从而避免了不安全curl | sh。图像也没有改变。

```bash
[...]
# Run vulnerability scan on build image
FROM build AS vulnscan
COPY --from=aquasec/trivy:latest /usr/local/bin/trivy /usr/local/bin/trivy
RUN trivy rootfs --exit-code 1 --no-progress /
[...]
```

##  3. 扫描文件系统
###  3.1 独立模式

```bash
本地项目
trivy fs /path/to/project
trivy fs ~/src/github.com/aquasecurity/trivy-ci-test
单个文件
trivy fs ~/src/github.com/aquasecurity/trivy-ci-test/Pipfile.lock
```

###  3.2 client/server

```bash
trivy server
trivy fs --server http://localhost:4954 --severity CRITICAL ./integration/testdata/fixtures/fs/pom/
```

## 3. 扫描 Rootfs
扫描根文件系统（例如主机、虚拟机映像或未打包的容器映像文件系统）
```bash
$ trivy rootfs /path/to/rootfs
$ docker run --rm -it alpine:3.11
/ # curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh | sh -s -- -b /usr/local/bin
/ # trivy rootfs /
```

##  4. 扫描 git 仓库
### 4.1 扫描您的远程 git 存储库
```bash
trivy repo https://github.com/knqyf263/trivy-ci-test
```
### 4.2 扫描分支
在提供的远程存储库上传递--branch具有有效分支名称的 agrument：

```bash
$ trivy repo --branch <branch-name> <repo-name>
```
### 4.3 扫描到 Commit
在提供的远程存储库上传递--commit具有有效提交哈希的 agrument：

```bash
$ trivy repo --commit <commit-hash> <repo-name>
```
### 4.4 扫描标签
在提供的远程存储库上传递--tag带有有效标签的 agrument：

```bash
$ trivy repo --tag <tag-name> <repo-name>
```
###  4.5 扫描私有存储库
为了扫描私有 GitHub 或 GitLab 存储库，必须分别设置环境变量`GITHUB_TOKEN`或，并使用有权访问正在扫描的私有存储库的有效令牌：`GITLAB_TOKEN`
环境变量将GITHUB_TOKEN优先于GITLAB_TOKEN，因此如果要扫描私有 GitLab 存储库，则GITHUB_TOKEN必须取消设置。

例如：


```bash
$ export GITHUB_TOKEN="your_private_github_token"
$ trivy repo <your private GitHub repo URL>
$
$ # or
$ export GITLAB_TOKEN="your_private_gitlab_token"
$ trivy repo <your private GitLab repo URL>
```
##  5. 扫描错误配置
只需指定一个包含 IaC 文件的目录，例如 Terraform、CloudFormation 和 Dockerfile。
格式：`trivy config [YOUR_IaC_DIRECTORY]`

实例

```bash
$ ls build/
Dockerfile
$ trivy config ./build
2022-05-16T13:29:29.952+0100    INFO    Detected config files: 1

Dockerfile (dockerfile)
=======================
Tests: 23 (SUCCESSES: 22, FAILURES: 1, EXCEPTIONS: 0)
Failures: 1 (UNKNOWN: 0, LOW: 0, MEDIUM: 1, HIGH: 0, CRITICAL: 0)

MEDIUM: Specify a tag in the 'FROM' statement for image 'alpine'
══════════════════════════════════════════════════════════════════════════════════════════════════════════════════════
When using a 'FROM' statement you should use a specific tag to avoid uncontrolled behavior when the image is updated.

See https://avd.aquasec.com/misconfig/ds001
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
Dockerfile:1
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
1 [ FROM alpine:latest
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
```
您还可以通过`--security-checks config.`

```bash
$ trivy image --security-checks config IMAGE_NAME

$ trivy fs --security-checks config /path/to/dir
```
与`config`子命令不同`image`，`fs`和`repo`子命令还可以同时扫描漏洞和秘密。您可以指定`--security-checks vuln,config,secret`启用漏洞和秘密检测以及错误配置检测。

```bash
$ ls myapp/
Dockerfile Pipfile.lock
$ trivy fs --security-checks vuln,config,secret --severity HIGH,CRITICAL myapp/
2022-05-16T13:42:21.440+0100    INFO    Number of language-specific files: 1
2022-05-16T13:42:21.440+0100    INFO    Detecting pipenv vulnerabilities...
2022-05-16T13:42:21.440+0100    INFO    Detected config files: 1

Pipfile.lock (pipenv)
=====================
Total: 1 (HIGH: 1, CRITICAL: 0)

┌──────────┬────────────────┬──────────┬───────────────────┬───────────────┬───────────────────────────────────────────────────────────┐
│ Library  │ Vulnerability  │ Severity │ Installed Version │ Fixed Version │                           Title                           │
├──────────┼────────────────┼──────────┼───────────────────┼───────────────┼───────────────────────────────────────────────────────────┤
│ httplib2 │ CVE-2021-21240 │ HIGH     │ 0.12.1            │ 0.19.0        │ python-httplib2: Regular expression denial of service via │
│          │                │          │                   │               │ malicious header                                          │
│          │                │          │                   │               │ https://avd.aquasec.com/nvd/cve-2021-21240                │
└──────────┴────────────────┴──────────┴───────────────────┴───────────────┴───────────────────────────────────────────────────────────┘

Dockerfile (dockerfile)
=======================
Tests: 17 (SUCCESSES: 16, FAILURES: 1, EXCEPTIONS: 0)
Failures: 1 (HIGH: 1, CRITICAL: 0)

HIGH: Last USER command in Dockerfile should not be 'root'
════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════
Running containers with 'root' user can lead to a container escape situation. It is a best practice to run containers as non-root users, which can be done by adding a 'USER' statement to the Dockerfile.

See https://avd.aquasec.com/misconfig/ds002
────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
Dockerfile:3
────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
3 [ USER root
────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────

```
在上面的示例中，Trivy 检测到了 Python 依赖项的漏洞和 Dockerfile 中的错误配置。

##  6. 类型检测
指定目录可以包含混合类型的 IaC 文件。Trivy 自动检测配置类型并应用相关策略。

例如，以下示例将 `Terraform`、`CloudFormation`、`Kubernetes`、`Helm Charts` 和 `Dockerfile` 的 `IaC` 文件保存在同一目录中。

```bash
$ ls iac/
Dockerfile  deployment.yaml  main.tf mysql-8.8.26.tar
$ trivy conf --severity HIGH,CRITICAL ./iac
```
输出：

```bash
Dockerfile (dockerfile)
=======================
Tests: 23 (SUCCESSES: 22, FAILURES: 1, EXCEPTIONS: 0)
Failures: 1 (HIGH: 1, CRITICAL: 0)

...

deployment.yaml (kubernetes)
============================
Tests: 28 (SUCCESSES: 15, FAILURES: 13, EXCEPTIONS: 0)
Failures: 13 (MEDIUM: 4, HIGH: 1, CRITICAL: 0)

...

main.tf (terraform)
===================
Tests: 23 (SUCCESSES: 14, FAILURES: 9, EXCEPTIONS: 0)
Failures: 9 (HIGH: 6, CRITICAL: 1)

...

bucket.yaml (cloudformation)
============================
Tests: 9 (SUCCESSES: 3, FAILURES: 6, EXCEPTIONS: 0)
Failures: 6 (UNKNOWN: 0, LOW: 0, MEDIUM: 2, HIGH: 4, CRITICAL: 0)

...

mysql-8.8.26.tar:templates/primary/statefulset.yaml (helm)
==========================================================
Tests: 20 (SUCCESSES: 18, FAILURES: 2, EXCEPTIONS: 0)
Failures: 2 (MEDIUM: 2, HIGH: 0, CRITICAL: 0)
```

下一篇我们将介绍 trivy 利用rego语言编写自定义策略。

参考：

 - [Trivy 官方](https://aquasecurity.github.io/trivy/v0.30.4/)


