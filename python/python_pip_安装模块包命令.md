#  python pip 安装模块包命令


---


![在这里插入图片描述](https://img-blog.csdnimg.cn/fcb89b52e00b445d87196563f9306582.png#pic_center)
##  1. 简介
[pip](https://pypi.org/project/pip/) 是Python的包安装程序。您可以使用 pip 从Python 包索引和其他索引安装包。


## 2. 格式

```bash
pip install xx
```

## 3. 列出已安装包

```bash
$ pip freeze or pip list
```

## 4. 导出requirements.txt清单

```bash
pip freeze > <目录>/requirements.txt
```

## 5. 安装本地安装包

```bash
pip install <目录>/<文件名>
pip install --no-index -f=<目录>/ <包名>
```

## 6. 卸载包

```bash
pip uninstall <包名> 或 pip uninstall -r requirements.txt
```

## 7. 升级包

```bash
pip install -U <包名>
pip install <包名> --upgrade
pip install -U pip  #升级pip
```

## 8. 显示包所在的目录

```bash
pip show -f <包名>
```

## 9. 搜索包

```bash
pip search <搜索关键字>
```

## 10. 查询可升级的包

```bash
pip list -o
```

## 11. 下载包而不安装

```bash
pip install <包名> -d <目录>
pip install -d <目录> -r requirements.txt
```

## 12. 打包

```bash
pip wheel <包名>
```



## 13. 指定全局安装源

 - 在unix和macos，配置文件为：$HOME/.pip/pip.conf
 - 在windows上，配置文件为：%HOME%\pip\pip.ini

```bash
[global]
timeout = 6000
index-url = https://mirrors.aliyun.com/pypi/simple
```

## 14. 离线安装

```bash
pip install --no-index --find-link=.  <包>   #"."为当前目录
pip install   --no-index   --find-links=/<目录>/   pandas
pip install   --no-index   --find-links=/<目录>/   -r   requirements.txt 
```



