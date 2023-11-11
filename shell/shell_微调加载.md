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
![在这里插入图片描述](https://img-blog.csdnimg.cn/49bc0d86961449df81b62ec8e48a2be8.gif#pic_center)


