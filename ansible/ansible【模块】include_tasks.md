

----
## 1. 介绍
`include_tasks`包括一个文件，其中包含要在当前剧本中执行的任务列表

## 2. 参数

 - `apply`    
 接受任务的关键字的哈希值（例如tags，become），将被应用到内的任务包括。
 - `file`    
   导入文件的名称是直接指定的，没有任何其他选项。与import_tasks不同，大多数关键字（包括loop，with_items和conditional）都适用于此语句。include_tasks不支持do until循环。
 - `free-form`  
  通过- include_tasks: file.yml要包含的文件的自由格式提供文件名等效于指定file的参数。

## 3. 示例

```bash
- hosts: all
  tasks:
    - debug:
        msg: task1

    - name: Include task list in play
      include_tasks: stuff.yaml

    - debug:
        msg: task10

- hosts: all
  tasks:
    - debug:
        msg: task1

    - name: Include task list in play only if the condition is true
      include_tasks: "{{ hostvar }}.yaml"
      when: hostvar is defined

- name: Apply tags to tasks within included file
  include_tasks:
    file: install.yml
    apply:
      tags:
        - install
  tags:
    - always

- name: Apply tags to tasks within included file when using free-form
  include_tasks: install.yml
  args:
    apply:
      tags:
        - install
  tags:
    - always
```

