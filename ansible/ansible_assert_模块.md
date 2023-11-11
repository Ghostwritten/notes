assert 模块可以很容易验证各种真理

```bash
tasks:

   - shell: /usr/bin/some-command --parameter value
     register: cmd_result

   - assert:
       that:
         - "'not ready' not in cmd_result.stderr"
         - "'gizmo enabled' in cmd_result.stdout"
```


如果你觉得需要测试通过 Ansible 设置的文件是否存在, ‘stat’ 模块是一个不错的选择:

```bash
tasks:

   - stat: path=/path/to/something
     register: p

   - assert:
       that:
         - p.stat.exists and p.stat.isdir
```

