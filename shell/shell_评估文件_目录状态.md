#  Shell 判断文件/目录状态
tags: 监控

 - `test-file.sh`

```bash
#!/bin/bash
# test-file: Evaluate the status of a file
echo "Hey what's the File/Directory name (using the absolute path)?"
read FILE

if [ -e "$FILE" ]; then
	if [ -f "$FILE" ]; then
		echo "$FILE is a regular file."
	fi
	if [ -d "$FILE" ]; then
		echo "$FILE is a directory."
	fi
	if [ -r "$FILE" ]; then
		echo "$FILE is readable."
	fi
	if [ -w "$FILE" ]; then
		echo "$FILE is writable."
	fi
	if [ -x "$FILE" ]; then
		echo "$FILE is executable/searchable."
	fi
else
	echo "$FILE does not exist"
	exit 1
fi
exit
```
执行：

```bash
$ ls 
test    dirname

$ bash test-file.sh
Hey what's the File/Directory name (using the absolute path)?
test
test is a regular file.
test is readable.
test is writable.

$ bash test-file.sh
Hey what's the File/Directory name (using the absolute path)?
dirname
dirname is a directory.
dirname is readable.
dirname is writable.
dirname is executable/searchable.
```

