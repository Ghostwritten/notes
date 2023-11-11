![在这里插入图片描述](https://img-blog.csdnimg.cn/2020120917511545.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70#pic_center)


## 1. 安装
### 1.1 tar包安装
下载：
https://github.com/s3tools/s3cmd/releases

```bash
wget https://github.com/s3tools/s3cmd/releases/download/v2.0.2/s3cmd-2.0.2.tar.gz
tar -xvf s3cmd-2.0.2.tar.gz
cd  s3cmd-2.0.2
python setup.py install
```

**注意：**
常见依赖工具：

```bash
pip install python-dateutil
pip install setuptools
```
### 1.2 pip安装

```bash
pip3 install s3cmd
```

## 2. 配置与文件处理
```bash
1.配置Access Key ID 和 Secret Access Key
 $ s3cmd --configure

2.列举所有的Buckets
$ s3cmd ls

3.创建 bucket，且 bucket 名称是唯一的，不能重复。
$ s3cmd mb s3://my-bucket-name

4.删除空 bucket
$ s3cmd rb s3://my-bucket-name

5.列举 Bucket 中的内容
$ s3cmd ls s3://my-bucket-name

6.上传 file.txt 到某个 bucket
$ s3cmd put file.txt s3://my-bucket-name/file.txt

7.上传并将权限设置为所有人可读
$ s3cmd put --acl-public file.txt s3://my-bucket-name/file.txt

8.批量上传文件
$ s3cmd put ./* s3://my-bucket-name/

9.下载文件
$ s3cmd get s3://my-bucket-name/file.txt file.txt

10.批量下载
$ s3cmd get s3://my-bucket-name/* ./

11.删除文件
$ s3cmd del s3://my-bucket-name/file.txt

12.来获得对应的bucket所占用的空间大小
$ s3cmd du -H s3://my-bucket-name
```

## 3. 文件夹处理规则
带"/"斜杠的 dir1，相当于上传yh目录下的所有文件，即类似 "cp ./* "

```bash
$ s3cmd put -r yh s3://yaohong-bucket
　　upload: 'yh/1' -> 's3://yaohong-bucket/yh/1' [1 of 4]
　　0 of 0 0% in 0s 0.00 B/s done
　　upload: 'yh/2' -> 's3://yaohong-bucket/yh/2' [2 of 4]
　　0 of 0 0% in 0s 0.00 B/s done
　　upload: 'yh/3.py' -> 's3://yaohong-bucket/yh/3.py' [3 of 4]
　　0 of 0 0% in 0s 0.00 B/s done
　　upload: 'yh/3.sh' -> 's3://yaohong-bucket/yh/3.sh' [4 of 4]
　　0 of 0 0% in 0s 0.00 B/s done
```

## 4. sync同步方法
1.同步当前目录下所有文件

```bash
$ s3cmd sync  ./  s3://yaohong-bucket/
```

2.加 "--dry-run"参数后，仅列出需要同步的项目，不实际进行同步。

```bash
$ s3cmd sync  --dry-run ./  s3://my-bucket-name/
```

3.加 " --delete-removed"参数后，会删除本地不存在的文件。

```bash
$ s3cmd sync  --delete-removed ./  s3://my-bucket-name/
```

4.加 " --skip-existing"参数后，不进行MD5校验，直接跳过本地已存在的文件。

```bash
$ s3cmd sync  --skip-existing ./  s3://my-bucket-name/
```

## 5. 高级同步
**排除、包含规则（--exclude 、--include）
file1-1.txt被排除，file2-2.txt同样是txt格式却能被包含**

```bash
$ s3cmd sync --dry-run --exclude '*.txt' --include 'dir2/*' ./  s3://my-bucket-name/
exclude: dir1/file1-1.txt
upload: ./dir2/file2-2.txt -> s3://my-bucket-name/dir2/file2-2.txt
```

从文件中载入排除或包含规则。(--exclude-from、--include-from)

```bash
$ s3cmd sync  --exclude-from pictures.exclude ./  s3://my-bucket-name/
```
排除或包含规则支持正则表达式
**--rexclude 、--rinclude、--rexclude-from、--rinclude-from**

## 6. 报错

```bash
# s3cmd mb s3://czsss-xxxx
ERROR: [Errno -2] Name or service not known
ERROR: Connection Error: Error resolving a server hostname.
Please check the servers address specified in 'host_base', 'host_bucket', 'cloudfront_host', 'website_endpoint'
```
缺失配置
s3cmd --configure 生成的配置文件如下


```bash
$ cat /root/.s3cfg
```

```bash
access_key = QFBD6HTA7KVCQ4FF0XGT
secret_key = 5yfezCjCiZxK8icwiG3MUWYD54jkU36f9cmEfaRO
host_base = s3-beta5.51wyq.cn:7480
host_bucket = %(bucket)s.s3-beta5.51wyq.cn:7480
simpledb_host = s3-beta5.51wyq.cn:7480
cloudfront_host = s3-beta5.51wyq.cn:7480
website_endpoint = http://%(bucket)s.s3-beta5.51wyq.cn:7480/
```


