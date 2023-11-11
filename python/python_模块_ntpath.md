#  python 模块 ntpath




---
## 1. 简介
不要直接导入这个模块，而是导入 os 并将这个
模块称为 os.path。

## 2. 示例
 - File: ntpath-example-1.py

```bash
import ntpath

file = "/my/little/pony"

print "isabs", "=>", ntpath.isabs(file)
print "dirname", "=>", ntpath.dirname(file)
print "basename", "=>", ntpath.basename(file)
print "normpath", "=>", ntpath.normpath(file)
print "split", "=>", ntpath.split(file)
print "join", "=>", ntpath.join(file, "zorba")
```
输出：

```bash
isabs => 1
dirname => /my/little
basename => pony

normpath => \my\little\pony
split => ('/my/little', 'pony')
join => /my/little/pony\zorba
```

更多阅读：

 - [Python ntpath Module](https://www.programcreek.com/python/index/586/ntpath)

