# Shell 命令返回值判断



----

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ce155c045737bd4e682acae94db9a4e8.gif#pic_center)


## 1.判断命令是否存在
### 1.1 优雅方法1
首先，检查命令是否有效的惯用方法直接在if语句中。
```powershell
if command; then
    echo notify user OK >&2
else
    echo notify user FAIL >&2
    return -1
fi
```
（良好做法：使用>&2将消息发送给stderr。）
### 1.2 优雅方法2
将通用逻辑转移到共享函数中。

```powershell
check() {
    local command=("$@")

    if "${command[@]}"; then
        echo notify user OK >&2
    else
        echo notify user FAIL >&2
        exit 1
    fi
}

check command1
check command2
check command3
```
### 1.3 优雅方法3

```powershell
installed () {
        command -v "$1" >/dev/null 2>&1
}
if installed <command1>
then
       <command1>  xx
else
        <command1>  xxx
 fi
```
## 2. 返回错误退出
### 2.1 || exit 退出

```powershell
command1 || exit
command2 || exit
command3 || exit
```
### 2.2 bash -e

```powershell
$  bash -e xx.sh
#!/bin/bash -e xx.sh
command1
command2
command3
```

### 2.3 set -e

```powershell
$ bash xx.sh 
#!/bin/bash
set -e 
command1
command2
command3
```
## 3. 返回错误提示
### 3.1 一般方法
方法1

```powershell
if do some command; then
    echo notify user OK
else
    echo notify user fail
    exit 255  # exit code must be unsigned short
fi
```
方法2

```powershell
do some command
if [ $? -eq 0 ]; then
    echo notify user OK
else
    echo notify user FAIL
    return -1
fi
```
### 3.2 优雅方法
方法1
```powershell
die() {
    local message=$1

    echo "$message" >&2
    exit 1
}

command1 || die 'command1 failed'
command2 || die 'command2 failed'
command3 || die 'command3 failed'
```
方法2（**推荐**）
```powershell
warn () {
  echo "$@" >&2
}

die () {
  status="$1"
  shift
  warn "$@"
  exit "$status"
}

do some command && echo notify user OK || die 255 Notify user fail
```

