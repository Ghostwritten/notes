#  shell 颜色输出
tags: 艺术

 - color.sh

```bash
#!/usr/bin/env bash

for c in 90 31 91 32 33 34 35 95 36 97; do
	echo -en "\r \e[${c}m LOVE \e[0m "
	sleep 1
done
```
执行：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/bd31535556e190706285c3f6319c0902.gif#pic_center)

