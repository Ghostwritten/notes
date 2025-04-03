#  trivy 自定义扫描策略
tags: trivy,安全
<!-- catalog:trivy 自定义策略:-->



---
[![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8f73d40820025efdd2a7d5cb11727fb8.png)](https://www.rottentomatoes.com/tv/for_all_mankind)


##  1. 简介
您可以在[Rego](https://www.openpolicyagent.org/docs/latest/policy-language/)中编写自定义策略。完成编写自定义策略后，您可以通过选项传递存储这些策略的目录--policy。

```bash
trivy conf --policy /path/to/custom_policies --namespaces user /path/to/config_dir
```
## 2. 文件格式
如果文件名与以下文件模式匹配，Trivy 将解析该文件并将其作为输入传递给您的 Rego 策略。
| 文件格式       | 文件模式                                             |
|------------|--------------------------------------------------|
| JSON       | *.json                                           |
| YAML       | *.yaml和*.yml                                     |
| Dockerfile | Dockerfile, Dockerfile.*, 和*.Dockerfile          |
| Containerfile       | Containerfile, Containerfile.*, 和*.Containerfile |
| Terraform         | *.tf和*.tf.json                                   |

##  3. 配置语言
在上述通用文件格式中，Trivy 会自动识别以下类型的配置文件：

 - `CloudFormation` (JSON/YAML)
 - `Kubernetes` (JSON/YAML)
 - `Helm` (YAML)
 - `Terraform Plan` (JSON)

##  4. Rego 格式
一个包必须只包含一个策略。

```bash
package user.kubernetes.ID001

import lib.result

__rego_metadata__ := {
    "id": "ID001",
    "title": "Deployment not allowed",
    "severity": "LOW",
    "description": "Deployments are not allowed because of some reasons.",
}

__rego_input__ := {
    "selector": [
        {"type": "kubernetes"},
    ],
}

deny[res] {
    input.kind == "Deployment"
    msg := sprintf("Found deployment '%s' but deployments are not allowed", [input.metadata.name])
    res := result.new(msg, input)
}

```
在此示例中，ID001“不允许部署”在 下定义`user.kubernetes.ID001`。如果添加新的自定义策略，则必须在新包下定义它，例如`user.kubernetes.ID002`.

##  5. rego 结构
- `package`（必需的）

   - 必须遵循 Rego 的规范
   - 每个政策必须是唯一的
   - 应包含唯一性的策略 ID
   - 可以包括组名，例如kubernetes为了清楚起见
   - 组名对策略评估没有影响

- `import data.lib.result`（可选的）
  如果您想用行号和代码突出显示来修饰您的结果，可以定义

- `__rego_metadata__`（可选的）
  为了清楚起见，应该定义这些值，因为这些值将显示在扫描结果中


- `__rego_input__`（可选的）
  当你想指定输入格式时可以定义

 - deny（必需的）
应该是deny或开始于deny_
 尽管`warn`, `warn_*`, `violation`,`violation_`也适用于兼容性，但deny建议使用严重性，因为可以`__rego_metadata__`.
应返回以下之一：
调用的结果`result.new(msg, cause)`。`msg`是string描述问题发生的描述，`cause`是发生问题的属性/对象。提供这一点允许 Trivy 确定行号并突出显示输出中的代码。
A string表示检测到的问题
虽然`objectwithmsg`字段被接受，但其他字段被丢弃，string如果`result.new()`不使用，建议使用。
例如`{"msg": "deny message", "details": "something"}`


###  5.1 Package

```bash
user.kubernetes.ID001
```
默认情况下，只会`builtin.*`评估包。如果您定义自定义包，则必须通过`--namespaces`选项指定包前缀。

```bash
trivy conf --policy /path/to/custom_policies --namespaces user /path/to/config_dir
```
在这种情况下，`user.*`将进行评估。允许使用任何包前缀，例如`main`和`user`

###  5.2 Metadata
元数据有助于用有用的信息丰富 Trivy 的扫描结果。

```bash
__rego_metadata__ := {
    "id": "ID001",
    "title": "Deployment not allowed",
    "severity": "LOW",
    "description": "Deployments are not allowed because of some reasons.",
    "recommended_actions": "Remove Deployment",
    "url": "https://cloud.google.com/blog/products/containers-kubernetes/kubernetes-best-practices-resource-requests-and-limits",
}
```
下面的所有字段`__rego_metadata__`都是可选的。

| 字段名称 | 允许值                        | 默认值 | 在表中 | 在 JSON 中 |
|------|----------------------------|-----|-----|----------|
| id| 任何字符                       | 不适用 |   yes  |       yes   |
| title   | 任何字符                       | 不适用 |  yes   |     yes     |
| severity  | LOW, MEDIUM, HIGH,CRITICAL | 未知  | yes    |     yes     |
| description  | 任何字符                       |     |   no  |       yes   |
| recommended_actions | 任何字符                       |     |no     |   yes       |
| url  | 任何字符                       |     |    no |     yes     |

某些字段会显示在扫描结果中。

```bash
k.yaml (kubernetes)
───────────────────

Tests: 32 (SUCCESSES: 31, FAILURES: 1, EXCEPTIONS: 0)
Failures: 1 (UNKNOWN: 0, LOW: 1, MEDIUM: 0, HIGH: 0, CRITICAL: 0)

LOW: Found deployment 'my-deployment' but deployments are not allowed
════════════════════════════════════════════════════════════════════════
Deployments are not allowed because of some reasons.
────────────────────────────────────────────────────────────────────────
 k.yaml:1-2
────────────────────────────────────────────────────────────────────────
   1 ┌ apiVersion: v1
   2 └ kind: Deployment
────────────────────────────────────────────────────────────────────────

```

###  5.3 Input
您可以通过 指定输入格式`__rego_input__`。下面的所有字段`__rego_input`都是可选的。

```bash
__rego_input__ := {
    "combine": false,
    "selector": [
        {"type": "kubernetes"},
    ],
}
```

 - `combine` (boolean)设置为 true 时，指定目录下的所有配置文件合并为一个输入数据结构。
 - `selector` (array)此选项按文件格式或配置语言过滤输入。在上面的示例中，Trivy 仅将 Kubernetes
   文件传递​​给此策略。即使指定目录中存在 Dockerfile，它也不会作为输入传递给策略。当 Kubernetes
   等配置语言未识别时，会使用 JSON 等文件格式作为type. 当识别出一种配置语言时，它将覆盖type.
 - `type`接受kubernetes, dockerfile, cloudformation, terraform，terraformplan, json, 或yaml.

在“combine”模式下，input文档变成一个数组，其中每个元素都是一个具有两个字段的对象：

 - "`path`": "path/to/file"：相应文件的相对文件路径
 - "`contents`": ...：相应文件的解析内容

现在，您可以确保重复值在整个配置文件中匹配。

###  5.4 Return value
在“combine”模式下，deny入口点必须返回一个带有两个键的对象

 - `filepath`（必需的）：正在评估的文件的相对文件路径
 - `msg`（必需的）：描述问题的消息

```bash
deny[res] {
    resource := input[i].contents
    ... some logic ...

    res := {
        "filepath": input[i].path,
        "msg": "something bad",
    }
}
```
##  6. 自定义 data
自定义策略可能需要额外的数据才能确定答案。

例如，允许创建的资源列表。Trivy 允许将路径传递到带有`--data`标志的数据文件，而不是在策略中硬编码这些信息。

给定以下 yaml 文件

```bash
$ cd examples/misconf/custom-data
$ cat data/ports.yaml                                                                                                                                                                      [~/src/github.com/aquasecurity/trivy/examples/misconf/custom-data]
services:
  ports:
    - "20"
    - "20/tcp"
    - "20/udp"
    - "23"
    - "23/tcp"
```
这可以导入您的策略：

```bash
import data.services

ports := services.ports
```
然后，您需要通过`--data`选项传递数据路径。Trivy 递归搜索 `JSON ( *.json)` 和 `YAML ( *.yaml)` 文件的指定路径。

```bash
$ trivy conf --policy ./policy --data data --namespaces user ./configs
```

## 6. rego 测试
为了帮助您验证自定义策略的正确性，OPA 为您提供了一个框架，您可以使用该框架为您的策略编写测试。通过为您的自定义策略编写测试，您可以加快新规则的开发过程，并减少随着需求的发展而修改规则所需的时间。

有关更多详细信息，请参阅[策略测试](https://www.openpolicyagent.org/docs/latest/policy-testing/)。

[Trivy 的核心库Fanal](https://github.com/aquasecurity/fanal)可以作为 Go 库导入。您可以扫描 Go 中的配置文件并使用 Go 的测试方法（例如表驱动测试）测试您的自定义策略。这允许您使用实际的配置文件作为输入，从而轻松准备测试数据并确保您的自定义策略在实践中有效。

特别是 Dockerfile 和 HCL 需要转换为结构化数据作为输入，这可能与预期的输入格式不同。

> 我们建议同时编写 OPA 和 Go 测试，因为它们具有不同的角色，例如单元测试和集成测试。

以下示例将允许和拒绝的配置文件存储在目录中。 `Successes`包含成功的结果，也`Failures`包含失败的结果。

```bash
{
    name:  "disallowed ports",
    input: "configs/",
    fields: fields{
        policyPaths: []string{"policy"},
        dataPaths:   []string{"data"},
        namespaces:  []string{"user"},
    },
    want: []types.Misconfiguration{
        {
            FileType: types.Dockerfile,
            FilePath: "Dockerfile.allowed",
            Successes: types.MisconfResults{
                {
                    Namespace: "user.dockerfile.ID002",
                    PolicyMetadata: types.PolicyMetadata{
                        ID:          "ID002",
                        Type:        "Docker Custom Check",
                        Title:       "Disallowed ports exposed",
                        Severity:    "HIGH",
                    },
                },
            },
        },
        {
            FileType: types.Dockerfile,
            FilePath: "Dockerfile.denied",
            Failures: types.MisconfResults{
                {
                    Namespace: "user.dockerfile.ID002",
                    Message:   "Port 23 should not be exposed",
                    PolicyMetadata: types.PolicyMetadata{
                        ID:          "ID002",
                        Type:        "Docker Custom Check",
                        Title:       "Disallowed ports exposed",
                        Severity:    "HIGH",
                    },
                },
            },
        },
    },
},

```
`Dockerfile.allowed`有一个成功的结果`Successes`，而`Dockerfile.denied`有一个失败的结果`Failures`。

##  7. debug 策略
在处理更复杂的查询时（或在学习 Rego 时），准确了解策略是如何应用的很有用。为此，您可以使用该--trace标志。这将从 Open Policy Agent 输出一个大的跟踪，只有失败的策略才会显示痕迹。如果要调试已通过的策略，则需要故意使其失败。如下所示：

```bash
$ trivy conf --trace configs/
2022-05-16T13:47:58.853+0100    INFO    Detected config files: 1

Dockerfile (dockerfile)
=======================
Tests: 23 (SUCCESSES: 21, FAILURES: 2, EXCEPTIONS: 0)
Failures: 2 (UNKNOWN: 0, LOW: 0, MEDIUM: 1, HIGH: 1, CRITICAL: 0)

MEDIUM: Specify a tag in the 'FROM' statement for image 'alpine'
═══════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════
When using a 'FROM' statement you should use a specific tag to avoid uncontrolled behavior when the image is updated.

See https://avd.aquasec.com/misconfig/ds001
───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
 Dockerfile:1
───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
   1 [ FROM alpine:latest
───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────


HIGH: Last USER command in Dockerfile should not be 'root'
═══════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════
Running containers with 'root' user can lead to a container escape situation. It is a best practice to run containers as non-root users, which can be done by adding a 'USER' statement to the Dockerfile.

See https://avd.aquasec.com/misconfig/ds002
───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
 Dockerfile:3
───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
   3 [ USER root
───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────



ID: DS001
File: Dockerfile
Namespace: builtin.dockerfile.DS001
Query: data.builtin.dockerfile.DS001.deny
Message: Specify a tag in the 'FROM' statement for image 'alpine'
TRACE  Enter data.builtin.dockerfile.DS001.deny = _
TRACE  | Eval data.builtin.dockerfile.DS001.deny = _
TRACE  | Index data.builtin.dockerfile.DS001.deny (matched 1 rule)
TRACE  | Enter data.builtin.dockerfile.DS001.deny
TRACE  | | Eval output = data.builtin.dockerfile.DS001.fail_latest[_]
TRACE  | | Index data.builtin.dockerfile.DS001.fail_latest (matched 1 rule)
TRACE  | | Enter data.builtin.dockerfile.DS001.fail_latest
TRACE  | | | Eval output = data.builtin.dockerfile.DS001.image_tags[_]
TRACE  | | | Index data.builtin.dockerfile.DS001.image_tags (matched 2 rules)
TRACE  | | | Enter data.builtin.dockerfile.DS001.image_tags
TRACE  | | | | Eval from = data.lib.docker.from[_]
TRACE  | | | | Index data.lib.docker.from (matched 1 rule)
TRACE  | | | | Enter data.lib.docker.from
TRACE  | | | | | Eval instruction = input.stages[_][_]
TRACE  | | | | | Eval instruction.Cmd = "from"
TRACE  | | | | | Exit data.lib.docker.from
TRACE  | | | | Redo data.lib.docker.from
TRACE  | | | | | Redo instruction.Cmd = "from"
TRACE  | | | | | Redo instruction = input.stages[_][_]
TRACE  | | | | | Eval instruction.Cmd = "from"
TRACE  | | | | | Fail instruction.Cmd = "from"
TRACE  | | | | | Redo instruction = input.stages[_][_]
TRACE  | | | | | Eval instruction.Cmd = "from"
TRACE  | | | | | Fail instruction.Cmd = "from"
TRACE  | | | | | Redo instruction = input.stages[_][_]
TRACE  | | | | Eval name = from.Value[0]
TRACE  | | | | Eval not startswith(name, "$")
TRACE  | | | | Enter startswith(name, "$")
TRACE  | | | | | Eval startswith(name, "$")
TRACE  | | | | | Fail startswith(name, "$")
TRACE  | | | | Eval data.builtin.dockerfile.DS001.parse_tag(name, __local505__)
TRACE  | | | | Index data.builtin.dockerfile.DS001.parse_tag (matched 2 rules)
TRACE  | | | | Enter data.builtin.dockerfile.DS001.parse_tag
TRACE  | | | | | Eval split(name, ":", __local504__)
TRACE  | | | | | Eval [img, tag] = __local504__
TRACE  | | | | | Exit data.builtin.dockerfile.DS001.parse_tag
TRACE  | | | | Eval [img, tag] = __local505__
TRACE  | | | | Eval output = {"cmd": from, "img": img, "tag": tag}
TRACE  | | | | Exit data.builtin.dockerfile.DS001.image_tags
TRACE  | | | Redo data.builtin.dockerfile.DS001.image_tags
TRACE  | | | | Redo output = {"cmd": from, "img": img, "tag": tag}
TRACE  | | | | Redo [img, tag] = __local505__
TRACE  | | | | Redo data.builtin.dockerfile.DS001.parse_tag(name, __local505__)
TRACE  | | | | Redo data.builtin.dockerfile.DS001.parse_tag
TRACE  | | | | | Redo [img, tag] = __local504__
TRACE  | | | | | Redo split(name, ":", __local504__)
TRACE  | | | | Enter data.builtin.dockerfile.DS001.parse_tag
TRACE  | | | | | Eval tag = "latest"
TRACE  | | | | | Eval not contains(img, ":")
TRACE  | | | | | Enter contains(img, ":")
TRACE  | | | | | | Eval contains(img, ":")
TRACE  | | | | | | Exit contains(img, ":")
TRACE  | | | | | Redo contains(img, ":")
TRACE  | | | | | | Redo contains(img, ":")
TRACE  | | | | | Fail not contains(img, ":")
TRACE  | | | | | Redo tag = "latest"
TRACE  | | | | Redo name = from.Value[0]
TRACE  | | | | Redo from = data.lib.docker.from[_]
TRACE  | | | Enter data.builtin.dockerfile.DS001.image_tags
TRACE  | | | | Eval from = data.lib.docker.from[i]
TRACE  | | | | Index data.lib.docker.from (matched 1 rule)
TRACE  | | | | Eval name = from.Value[0]
TRACE  | | | | Eval cmd_obj = input.stages[j][k]
TRACE  | | | | Eval possibilities = {"arg", "env"}
TRACE  | | | | Eval cmd_obj.Cmd = possibilities[l]
TRACE  | | | | Fail cmd_obj.Cmd = possibilities[l]
TRACE  | | | | Redo possibilities = {"arg", "env"}
TRACE  | | | | Redo cmd_obj = input.stages[j][k]
TRACE  | | | | Eval possibilities = {"arg", "env"}
TRACE  | | | | Eval cmd_obj.Cmd = possibilities[l]
TRACE  | | | | Fail cmd_obj.Cmd = possibilities[l]
TRACE  | | | | Redo possibilities = {"arg", "env"}
TRACE  | | | | Redo cmd_obj = input.stages[j][k]
TRACE  | | | | Eval possibilities = {"arg", "env"}
TRACE  | | | | Eval cmd_obj.Cmd = possibilities[l]
TRACE  | | | | Fail cmd_obj.Cmd = possibilities[l]
TRACE  | | | | Redo possibilities = {"arg", "env"}
TRACE  | | | | Redo cmd_obj = input.stages[j][k]
TRACE  | | | | Redo name = from.Value[0]
TRACE  | | | | Redo from = data.lib.docker.from[i]
TRACE  | | | Eval __local752__ = output.img
TRACE  | | | Eval neq(__local752__, "scratch")
TRACE  | | | Eval __local753__ = output.img
TRACE  | | | Eval not data.builtin.dockerfile.DS001.is_alias(__local753__)
TRACE  | | | Enter data.builtin.dockerfile.DS001.is_alias(__local753__)
TRACE  | | | | Eval data.builtin.dockerfile.DS001.is_alias(__local753__)
TRACE  | | | | Index data.builtin.dockerfile.DS001.is_alias (matched 1 rule, early exit)
TRACE  | | | | Enter data.builtin.dockerfile.DS001.is_alias
TRACE  | | | | | Eval img = data.builtin.dockerfile.DS001.get_aliases[_]
TRACE  | | | | | Index data.builtin.dockerfile.DS001.get_aliases (matched 1 rule)
TRACE  | | | | | Enter data.builtin.dockerfile.DS001.get_aliases
TRACE  | | | | | | Eval from_cmd = data.lib.docker.from[_]
TRACE  | | | | | | Index data.lib.docker.from (matched 1 rule)
TRACE  | | | | | | Eval __local749__ = from_cmd.Value
TRACE  | | | | | | Eval data.builtin.dockerfile.DS001.get_alias(__local749__, __local503__)
TRACE  | | | | | | Index data.builtin.dockerfile.DS001.get_alias (matched 1 rule)
TRACE  | | | | | | Enter data.builtin.dockerfile.DS001.get_alias
TRACE  | | | | | | | Eval __local748__ = values[i]
TRACE  | | | | | | | Eval lower(__local748__, __local501__)
TRACE  | | | | | | | Eval "as" = __local501__
TRACE  | | | | | | | Fail "as" = __local501__
TRACE  | | | | | | | Redo lower(__local748__, __local501__)
TRACE  | | | | | | | Redo __local748__ = values[i]
TRACE  | | | | | | Fail data.builtin.dockerfile.DS001.get_alias(__local749__, __local503__)
TRACE  | | | | | | Redo __local749__ = from_cmd.Value
TRACE  | | | | | | Redo from_cmd = data.lib.docker.from[_]
TRACE  | | | | | Fail img = data.builtin.dockerfile.DS001.get_aliases[_]
TRACE  | | | | Fail data.builtin.dockerfile.DS001.is_alias(__local753__)
TRACE  | | | Eval output.tag = "latest"
TRACE  | | | Exit data.builtin.dockerfile.DS001.fail_latest
TRACE  | | Redo data.builtin.dockerfile.DS001.fail_latest
TRACE  | | | Redo output.tag = "latest"
TRACE  | | | Redo __local753__ = output.img
TRACE  | | | Redo neq(__local752__, "scratch")
TRACE  | | | Redo __local752__ = output.img
TRACE  | | | Redo output = data.builtin.dockerfile.DS001.image_tags[_]
TRACE  | | Eval __local754__ = output.img
TRACE  | | Eval sprintf("Specify a tag in the 'FROM' statement for image '%s'", [__local754__], __local509__)
TRACE  | | Eval msg = __local509__
TRACE  | | Eval __local755__ = output.cmd
TRACE  | | Eval data.lib.docker.result(msg, __local755__, __local510__)
TRACE  | | Index data.lib.docker.result (matched 1 rule)
TRACE  | | Enter data.lib.docker.result
TRACE  | | | Eval object.get(cmd, "EndLine", 0, __local470__)
TRACE  | | | Eval object.get(cmd, "Path", "", __local471__)
TRACE  | | | Eval object.get(cmd, "StartLine", 0, __local472__)
TRACE  | | | Eval result = {"endline": __local470__, "filepath": __local471__, "msg": msg, "startline": __local472__}
TRACE  | | | Exit data.lib.docker.result
TRACE  | | Eval res = __local510__
TRACE  | | Exit data.builtin.dockerfile.DS001.deny
TRACE  | Redo data.builtin.dockerfile.DS001.deny
TRACE  | | Redo res = __local510__
TRACE  | | Redo data.lib.docker.result(msg, __local755__, __local510__)
TRACE  | | Redo data.lib.docker.result
TRACE  | | | Redo result = {"endline": __local470__, "filepath": __local471__, "msg": msg, "startline": __local472__}
TRACE  | | | Redo object.get(cmd, "StartLine", 0, __local472__)
TRACE  | | | Redo object.get(cmd, "Path", "", __local471__)
TRACE  | | | Redo object.get(cmd, "EndLine", 0, __local470__)
TRACE  | | Redo __local755__ = output.cmd
TRACE  | | Redo msg = __local509__
TRACE  | | Redo sprintf("Specify a tag in the 'FROM' statement for image '%s'", [__local754__], __local509__)
TRACE  | | Redo __local754__ = output.img
TRACE  | | Redo output = data.builtin.dockerfile.DS001.fail_latest[_]
TRACE  | Exit data.builtin.dockerfile.DS001.deny = _
TRACE  Redo data.builtin.dockerfile.DS001.deny = _
TRACE  | Redo data.builtin.dockerfile.DS001.deny = _
TRACE


ID: DS002
File: Dockerfile
Namespace: builtin.dockerfile.DS002
Query: data.builtin.dockerfile.DS002.deny
Message: Last USER command in Dockerfile should not be 'root'
TRACE  Enter data.builtin.dockerfile.DS002.deny = _
TRACE  | Eval data.builtin.dockerfile.DS002.deny = _
TRACE  | Index data.builtin.dockerfile.DS002.deny (matched 2 rules)
TRACE  | Enter data.builtin.dockerfile.DS002.deny
TRACE  | | Eval data.builtin.dockerfile.DS002.fail_user_count
TRACE  | | Index data.builtin.dockerfile.DS002.fail_user_count (matched 1 rule, early exit)
TRACE  | | Enter data.builtin.dockerfile.DS002.fail_user_count
TRACE  | | | Eval __local771__ = data.builtin.dockerfile.DS002.get_user
TRACE  | | | Index data.builtin.dockerfile.DS002.get_user (matched 1 rule)
TRACE  | | | Enter data.builtin.dockerfile.DS002.get_user
TRACE  | | | | Eval user = data.lib.docker.user[_]
TRACE  | | | | Index data.lib.docker.user (matched 1 rule)
TRACE  | | | | Enter data.lib.docker.user
TRACE  | | | | | Eval instruction = input.stages[_][_]
TRACE  | | | | | Eval instruction.Cmd = "user"
TRACE  | | | | | Fail instruction.Cmd = "user"
TRACE  | | | | | Redo instruction = input.stages[_][_]
TRACE  | | | | | Eval instruction.Cmd = "user"
TRACE  | | | | | Exit data.lib.docker.user
TRACE  | | | | Redo data.lib.docker.user
TRACE  | | | | | Redo instruction.Cmd = "user"
TRACE  | | | | | Redo instruction = input.stages[_][_]
TRACE  | | | | | Eval instruction.Cmd = "user"
TRACE  | | | | | Fail instruction.Cmd = "user"
TRACE  | | | | | Redo instruction = input.stages[_][_]
TRACE  | | | | Eval username = user.Value[_]
TRACE  | | | | Exit data.builtin.dockerfile.DS002.get_user
TRACE  | | | Redo data.builtin.dockerfile.DS002.get_user
TRACE  | | | | Redo username = user.Value[_]
TRACE  | | | | Redo user = data.lib.docker.user[_]
TRACE  | | | Eval count(__local771__, __local536__)
TRACE  | | | Eval lt(__local536__, 1)
TRACE  | | | Fail lt(__local536__, 1)
TRACE  | | | Redo count(__local771__, __local536__)
TRACE  | | | Redo __local771__ = data.builtin.dockerfile.DS002.get_user
TRACE  | | Fail data.builtin.dockerfile.DS002.fail_user_count
TRACE  | Enter data.builtin.dockerfile.DS002.deny
TRACE  | | Eval cmd = data.builtin.dockerfile.DS002.fail_last_user_root[_]
TRACE  | | Index data.builtin.dockerfile.DS002.fail_last_user_root (matched 1 rule)
TRACE  | | Enter data.builtin.dockerfile.DS002.fail_last_user_root
TRACE  | | | Eval stage_users = data.lib.docker.stage_user[_]
TRACE  | | | Index data.lib.docker.stage_user (matched 1 rule)
TRACE  | | | Enter data.lib.docker.stage_user
TRACE  | | | | Eval stage = input.stages[stage_name]
TRACE  | | | | Eval users = [cmd | cmd = stage[_]; cmd.Cmd = "user"]
TRACE  | | | | Enter cmd = stage[_]; cmd.Cmd = "user"
TRACE  | | | | | Eval cmd = stage[_]
TRACE  | | | | | Eval cmd.Cmd = "user"
TRACE  | | | | | Fail cmd.Cmd = "user"
TRACE  | | | | | Redo cmd = stage[_]
TRACE  | | | | | Eval cmd.Cmd = "user"
TRACE  | | | | | Exit cmd = stage[_]; cmd.Cmd = "user"
TRACE  | | | | Redo cmd = stage[_]; cmd.Cmd = "user"
TRACE  | | | | | Redo cmd.Cmd = "user"
TRACE  | | | | | Redo cmd = stage[_]
TRACE  | | | | | Eval cmd.Cmd = "user"
TRACE  | | | | | Fail cmd.Cmd = "user"
TRACE  | | | | | Redo cmd = stage[_]
TRACE  | | | | Exit data.lib.docker.stage_user
TRACE  | | | Redo data.lib.docker.stage_user
TRACE  | | | | Redo users = [cmd | cmd = stage[_]; cmd.Cmd = "user"]
TRACE  | | | | Redo stage = input.stages[stage_name]
TRACE  | | | Eval count(stage_users, __local537__)
TRACE  | | | Eval len = __local537__
TRACE  | | | Eval minus(len, 1, __local538__)
TRACE  | | | Eval last = stage_users[__local538__]
TRACE  | | | Eval user = last.Value[0]
TRACE  | | | Eval user = "root"
TRACE  | | | Exit data.builtin.dockerfile.DS002.fail_last_user_root
TRACE  | | Redo data.builtin.dockerfile.DS002.fail_last_user_root
TRACE  | | | Redo user = "root"
TRACE  | | | Redo user = last.Value[0]
TRACE  | | | Redo last = stage_users[__local538__]
TRACE  | | | Redo minus(len, 1, __local538__)
TRACE  | | | Redo len = __local537__
TRACE  | | | Redo count(stage_users, __local537__)
TRACE  | | | Redo stage_users = data.lib.docker.stage_user[_]
TRACE  | | Eval msg = "Last USER command in Dockerfile should not be 'root'"
TRACE  | | Eval data.lib.docker.result(msg, cmd, __local540__)
TRACE  | | Index data.lib.docker.result (matched 1 rule)
TRACE  | | Enter data.lib.docker.result
TRACE  | | | Eval object.get(cmd, "EndLine", 0, __local470__)
TRACE  | | | Eval object.get(cmd, "Path", "", __local471__)
TRACE  | | | Eval object.get(cmd, "StartLine", 0, __local472__)
TRACE  | | | Eval result = {"endline": __local470__, "filepath": __local471__, "msg": msg, "startline": __local472__}
TRACE  | | | Exit data.lib.docker.result
TRACE  | | Eval res = __local540__
TRACE  | | Exit data.builtin.dockerfile.DS002.deny
TRACE  | Redo data.builtin.dockerfile.DS002.deny
TRACE  | | Redo res = __local540__
TRACE  | | Redo data.lib.docker.result(msg, cmd, __local540__)
TRACE  | | Redo data.lib.docker.result
TRACE  | | | Redo result = {"endline": __local470__, "filepath": __local471__, "msg": msg, "startline": __local472__}
TRACE  | | | Redo object.get(cmd, "StartLine", 0, __local472__)
TRACE  | | | Redo object.get(cmd, "Path", "", __local471__)
TRACE  | | | Redo object.get(cmd, "EndLine", 0, __local470__)
TRACE  | | Redo msg = "Last USER command in Dockerfile should not be 'root'"
TRACE  | | Redo cmd = data.builtin.dockerfile.DS002.fail_last_user_root[_]
TRACE  | Exit data.builtin.dockerfile.DS002.deny = _
TRACE  Redo data.builtin.dockerfile.DS002.deny = _
TRACE  | Redo data.builtin.dockerfile.DS002.deny = _
TRACE

```
参考：

 - [trivy 官方](https://aquasecurity.github.io/trivy/dev/docs/misconfiguration/custom/)

