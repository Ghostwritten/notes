

---
## 1. 介绍
`Roles`是ansible自`1.2`版本引入的新特性，用于层次性，结构化地组织playbook，roles能够根据层次型结构自动自动装在变量文件、tasks以及handlers
等。要使用roles只需要在playbook中使用include指令即可。简单来讲，**roles就是通过分别将变量、文件、任务、模板及处理器放置于单独的目录中
并可以便捷地include他们的一种机制，角色一般用于主机构建服务的场景中，但也可以是用于构建守护进程等场景中。**


## 2. 创建roles步骤

 - 创建以roles命名的目录：
 - 在roles目录中分别创建以各角色名称命名的目录，如webservers等：
 - 在每个角色命名的目录中分别创建files、handlers、meta、tasks、templates和vars目录：用不到的目录可以创建为空目录，也可以不创建。
 - 在playbook文件中，调用各角色

## 3. roles各目录中文件

 1. `tasks`目录：至少创建一个名为main.yml的文件，tasks/main.yml -角色执行的主要任务列表：此文件可以使用 `include`包含其他的位于此目录中的tasks文件；

 2. `files`目录：存放由copy或者script等模块调用的文件；
 3. `templates`目录：templates模块会自动在此目录中寻找Jinjia2模板文件；
 4. `handlers`目录：handlers/main.yml -处理程序，可以在此角色内部或外部使用；
 5. `yml`文件：用于定义此角色用到的各handler：在handler中使用include包含的其他的
handler文件也应该位于此目录中；
 6. `vars`目录：应当包含一个main.yml文件，用于定义此角色用到的变量
 7. `meta`目录：应当包含一个main.yml文件，用于定义此角色的特殊设定及其依赖关系：`ansible 1.3`及其以后的版本才支持
 8. `default`目录：defaults/main.yml-角色的默认变量,这些变量在所有可用变量中具有最低的优先级，并且很容易被其他任何变量（包括库存变量）覆盖。
 9. library/my_module.py-模块，可以在该角色中使用（有关更多信息，请参见在模块中嵌入模块和插件）。

 roles：

    （1）目录名同角色名
    （2）目录结构有固定格式：
    （3） files：静态文件
    （4） templates：Jinjia2 模板文件
    （5） tasks：至少有一个main.yml文件，定义tasks：
    （6）  hangdlers：至少有一个main.yml文件，定义各handlers
    （7） vars：至少有一个main.yml文件，定义变量
    （8）  meta：定义依赖关系等信息


下面是一个role的目录结构

```bash
# playbooks
site.yml
webservers.yml
fooservers.yml
roles/
    common/
        tasks/
        handlers/
        library/
        files/
        templates/
        vars/
        defaults/
        meta/
    webservers/
        tasks/
        defaults/
        meta/
```

## 4. 存储和查找角色
默认情况下，Ansible在两个位置查找角色：

roles/相对于剧本文件位于名为的目录中

在 `/etc/ansible/roles`

如果您将角色存储在其他位置，请设置role_path配置选项，以便Ansible可以找到您的角色。将共享角色检入一个位置可以使它们更容易在多个剧本中使用。有关在ansible.cfg中管理设置的详细信息，请参见配置Ansible。

或者，您可以使用完全限定的路径来调用角色：

```bash
---
- hosts: webservers
  roles:
    - role: '/path/to/my/roles/common'
```

或者
```bash
---
- hosts: webservers
  roles:
    - { role: '/path/to/my/roles/common' }
```



## 5. 使用角色
您可以通过三种方式使用角色：

 1. 在播放级别具有以下roles选项：这是在播放中使用角色的经典方法。
 2. 在任务级别上使用`include_role`：您可以在tasks播放区域的任何地方使用include_role。
 3. 在任务级别上使用`import_role`：您可以在tasks剧本部分的任何位置静态重用角色import_role。

### 5.1 role
使用角色的经典（原始）方法是roles给定播放的选项：


```bash
---
- hosts: webservers
  roles:
    - foo
    - bar
    - foo
```

roles在播放级别使用该选项时，对于每个角色“ x”：

 - 如果存在`role / x / tasks / main.yml`，则Ansible将该文件中的任务添加到播放中。
 - 如果存在`role / x / handlers / main.yml`，则Ansible将该文件中的处理程序添加到播放中。

如果存在role / x / vars / main.yml，则Ansible将该文件中的变量添加到播放中。

 - 如果存在`role / x / defaults / main.yml`，则Ansible将该文件中的变量添加到播放中。
 - 如果存在role / x / meta / main.yml，则Ansible将该文件中的所有角色依赖项添加到角色列表中。
 - （在角色中）**任何副本，脚本，模板或包含任务都可以引用role / x / {文件，模板，任务} /（目录取决于任务）中的文件，而不必相对或绝对地进行路径设置**。

当您roles在播放级别使用该选项时，Ansible会将角色视为静态导入，并在剧本解析过程中对其进行处理。Ansible按以下顺序执行您的剧本：

 1. pre_tasks戏剧中定义的任何内容。
 2. 由pre_tasks触发的任何处理程序。
 3. 每个角色roles:，按列出的顺序列出。`meta/main.yml`首先运行角色中定义的任何角色依赖关系，但要遵循标签过滤和条件。有关更多详细信息，请参见使用角色依赖性。
 4. tasks戏剧中定义的任何内容。
 5. 由角色或任务触发的任何处理程序。
 6. post_tasks戏剧中定义的任何内容。
 7. 由post_tasks触发的任何处理程序。

> 如果对角色中的任务使用标记，请确保还标记您的`pre_tasks`，`post_tasks`和角色依赖性，并同时传递它们，尤其是如果`pre / post`任务和角色依赖性用于监视中断窗口控制或负载平衡。有关添加和使用标签的详细信息，请参见标签。

您可以将其他关键字传递给该roles选项：

您可以在每个角色定义中传递不同的参数，如下所示：

```bash
---
- hosts: webservers
  roles:
    - { role: foo, vars: { message: "first" } }
    - { role: foo, vars: { message: "second" } }
```

或者
```bash
---
- hosts: webservers
  roles:
    - common
    - role: foo_app_instance
      vars:
        dir: '/opt/a'
        app_port: 5000
      tags: typeA
    - role: foo_app_instance
      vars:
        dir: '/opt/b'
        app_port: 5001
      tags: typeB
```
将标签添加到role选项时，Ansible会将标签应用于角色中的所有任务。

#### 5.1.1 使用 allow_duplicates: true
添加到角色文件中`meta/main.yml`

```bash
# playbook.yml
---
- hosts: webservers
  roles:
    - foo
    - foo

# roles/foo/meta/main.yml
---
allow_duplicates: true
```

在此示例中，Ansible运行foo两次，因为我们已明确启用它。

#### 5.1.2 使用角色依赖
角色依赖性使您可以在使用角色时自动拉入其他角色。当您包含或导入角色时，Ansible不会执行角色依赖性。roles如果要让Ansible执行角色依赖关系，则必须使用关键字。

角色相关性存储在`meta/main.yml`角色目录内的文件中。此文件应包含角色和参数的列表，以在指定角色之前插入。例如：

```bash
# roles/myapp/meta/main.yml
---
dependencies:
  - role: common
    vars:
      some_parameter: 3
  - role: apache
    vars:
      apache_port: 80
  - role: postgres
    vars:
      dbname: blarg
      other_parameter: 12
```
Ansible始终在包含角色依赖项的角色之前执行角色依赖项。Ansible还执行递归角色依赖关系。如果一个角色依赖于第二个角色，而第二个角色依赖于第三个角色，则Ansible将执行第三个角色，然后执行第二个角色，然后执行第一个角色

#### 5.1.3 多次运行角色依赖性
Ansible对待重复的角色依赖项就像在下面列出的重复角色一样roles:：Ansible仅执行一次角色依赖项，即使定义了多次，除非角色上定义的参数，标签或when子句对于每个定义都不同。如果剧本中的两个角色都将第三个角色列为依赖关系，则除非您传递不同的参数，标签，when子句或在依赖（第三个）角色中使用，否则Ansible仅运行一次该角色依赖关系。
一个命名角色car取决于一个命名角色wheel，如下所示：

```bash
---
dependencies:
  - role: wheel
    vars:
      n: 1
  - role: wheel
    vars:
      n: 2
  - role: wheel
    vars:
      n: 3
  - role: wheel
    vars:
      n: 4
```

并且wheel角色取决于两个角色：tire和brake。然后，meta/main.ymlfor轮将包含以下内容：

```bash
---
dependencies:
  - role: tire
  - role: brake
```

并且`meta/main.yml`    `fortire`和`brake`将包含以下内容：

```bash
---
allow_duplicates: true
```

执行的结果顺序如下：

```bash
tire(n=1)
brake(n=1)
wheel(n=1)
tire(n=2)
brake(n=2)
wheel(n=2)
...
car
```

要与角色依赖项一起使用，必须为依赖角色而不是父角色指定它。另外，在上述的例子中，出现在的和作用。该角色不需要，因为定义的每个实例使用不同的参数值。



### 5.2 include_role
虽然在roles部分中添加的角色先于剧本中的任何其他任务运行，但包含的角色将按照定义的顺序运行。如果某个任务之前还有其他任务`include_role`，则其他任务将首先运行。

示例1

```bash
---
- hosts: webservers
  tasks:
    - name: Print a message
      debug:
        msg: "this task runs before the example role"

    - name: Include the example role
      include_role:
        name: example

    - name: Print a message
      debug:
        msg: "this task runs after the example role"
```

示例2：

```bash
---
- hosts: webservers
  tasks:
    - name: Include the foo_app_instance role
      include_role:
        name: foo_app_instance
      vars:
        dir: '/opt/a'
        app_port: 5000
      tags: typeA
  ...
```

示例3：

您可以有条件地包括一个角色：

```bash
---
- hosts: webservers
  tasks:
    - name: Include the some_role role
      include_role:
        name: some_role
      when: "ansible_facts['os_family'] == 'RedHat'"
```
### 5.3 import_role
您可以使用tasks来在剧本部分的任何位置静态重用角色import_role。行为与使用roles关键字相同。例如：

```bash
---
- hosts: webservers
  tasks:
    - name: Print a message
      debug:
        msg: "before we run our role"

    - name: Import the example role
      import_role:
        name: example

    - name: Print a message
      debug:
        msg: "after we ran our role"
```

导入角色时，可以传递其他关键字，包括变量和标签：

```bash
---
- hosts: webservers
  tasks:
    - name: Import the foo_app_instance role
      import_role:
        name: foo_app_instance
      vars:
        dir: '/opt/a'
        app_port: 5000
  ...
```



