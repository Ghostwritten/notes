#  shell 微调加载
tags: 艺术

 - `affect.sh`

```bash
#!/usr/bin/env bash

arr=('-' '\' '|' '/')
while true; do
	for c in "${arr[@]}"; do
		echo -en "\r $c "
		sleep .5
	done
done
```
执行：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/2e2828f61169c17e60fbc935a93449ca.gif#pic_center)


