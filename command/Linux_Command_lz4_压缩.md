#  Linux Command lz4 压缩
![在这里插入图片描述](https://img-blog.csdnimg.cn/d57974856682420a90bdb3accb51c1c5.gif#pic_center)



## 1. lz4
特点：lz4压缩率略低，解压速度惊人
###  1.1 安装
```bash
yum install -y lz4 lz4-devel 
```

###  1.2 方法

```bash
压缩 (默认解压之后的名称filename.lz4)
# lz4 filename      
 
解压缩
# lz4 -d filename.lz4
 
centos7下默认有lz4_decompress 命令，可以直接解压, 并可以定义解压后的文件名
# lz4_decompress filename.lz4 filename
# lz4_decompress filename.lz4 filename.txt
```
###  1.3 参数

```bash
参数
-1:  快速压缩（默认）
-9:  高压缩
-d:  解压缩（默认为.lz4扩展名）
-z:  强制压缩
-f:  覆盖输出而不提示
-k:  保留源文件（默认）
--rm:  成功地解除/压缩后删除源文件
-h/-h:  显示帮助/长帮助和退出

高级参数
-v:  显示版本号并退出
-v:  详细模式
-q:  取消警告；指定两次也可以取消错误
-c:  强制写入标准输出，即使它是控制台
-t:  测试压缩文件完整性
-m:  多个输入文件（表示自动输出文件名）
-r:  在目录上递归操作（也设置为-m）
-l:  使用旧格式压缩（Linux内核压缩）
```
### 1.4 举例


```bash
1) 对test文件进行压缩
$ lz4 test
Compressed filename will be : test.lz4
Compressed 8 bytes into 27 bytes ==> 337.50%      
$ ls
test  test.lz4
  
快速压缩（-1参数），默认的就是快速压缩，如上面那条命令
$ rm -f test.lz4
$ lz4 -1 test
Compressed filename will be : test.lz4
Compressed 8 bytes into 27 bytes ==> 337.50%
$ ls
test  test.lz4
  
高压缩（-9参数）
$ rm -f test.lz4
$ lz4 -9 test
Compressed filename will be : test.lz4
Compressed 8 bytes into 27 bytes ==> 337.50%  
$ ls
test  test.lz4
  
当出现同名压缩文件时，直接压缩默认会有是否覆盖的提示信息
$ lz4 -9 test
Compressed filename will be : test.lz4
test.lz4 already exists; do you wish to overwrite (y/N) ? y
Compressed 8 bytes into 27 bytes ==> 337.50%    
  
已存在同名压缩文件时，直接压缩而不输出是否覆盖的提示信息
$ lz4 -9 -f test
Compressed filename will be : test.lz4
Compressed 8 bytes into 27 bytes ==> 337.50%                                 
$ ls
test  test.lz4
  
压缩文件时，保留源文件 （-f 参数），默认压缩后就是保留源文件，所以-f参数加不加都可以
$ rm -f test.lz4
$ lz4 test
Compressed filename will be : test.lz4
Compressed 8 bytes into 27 bytes ==> 337.50%                                 
$ ls
test  test.lz4
  
$ rm -f test.lz4
$ lz4 -f test
Compressed filename will be : test.lz4
Compressed 8 bytes into 27 bytes ==> 337.50%                                 
$ ls
test  test.lz4
  
压缩成功后，将源文件删除 （--rm参数）
$ rm -f test.lz4
$ ls
test
$ lz4 --rm test
Compressed filename will be : test.lz4
Compressed 8 bytes into 27 bytes ==> 337.50% 
$ ls
test.lz4
  
2）对压缩文件进行解压缩
  
默认通过-d参数进行解压缩
$ ls
test.lz4
$ lz4 -d test.lz4
Decoding file test
test.lz4             : decoded 8 bytes   
$ ls
test  test.lz4
$ cat test
haha,hello world!!
  
也可以使用lz4_decompress命令进行解压缩，并且可以自定义解压缩之后的文件名
$ rm -f test
$ ls
test.lz4
$ lz4_decompress test.lz4 kevin        #将test.lz4解压缩，解压缩之后为kevin文件
$ ls
kevin  test.lz4
$ cat kevin
haha,hello world!!
  
3） 压缩时，取消告警提示信息 （-q参数）
$ rm -f test.lz4
$ ls
kevin
$ lz4 -q kevin
$ ls
kevin  kevin.lz4
$ lz4 -q -f kevin  
$ lz4 -q -f --rm kevin
$ ls
kevin.lz4
  
4）对多个文件进行匹配压缩
$ ls
bobo  kevin
$ lz4 -m bobo kevin
$ ls
bobo  bobo.lz4  kevin  kevin.lz4
  
$ rm -rf bobo kevin
$ ls
bobo.lz4  kevin.lz4
  
$ lz4 -d bobo.lz4 -q
$ lz4 -d kevin.lz4 -q  
$ ls
bobo  bobo.lz4  kevin  kevin.lz4
```
![](https://img-blog.csdnimg.cn/fbc5767b60d44cb783138f4a98c31585.png)



更多阅读：
 1. [Linux Command lz4 压缩](https://blog.csdn.net/xixihahalelehehe/article/details/125201619)
 2. [Linux Command tar 压缩](https://ghostwritten.blog.csdn.net/article/details/106056841)
 3. [Linux Command gzip 压缩](https://blog.csdn.net/xixihahalelehehe/article/details/104724596)
 4. [Linux Command zip 压缩](https://blog.csdn.net/xixihahalelehehe/article/details/125203106)




