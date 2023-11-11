#  Shell  大小写
tags: 艺术



---

##  1. Shell  大写转小写
```bash
$ cat test
ABC
```

 - `convertlowercase.sh`

```bash
#!/usr/bin/env bash

echo -n "Enter File Name: "
read -r file

if [ ! -f "$file" ]; then
        echo "Filename $file does not exists"
        exit 1
fi

tr '[:upper:]' '[:lower:]' < "$file" 
```
执行：

```bash
$ bash convertlowercase.sh
Enter File Name: test
abc
```

