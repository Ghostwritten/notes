#  Linux Command gpg 加密
```bash
$ cat test
ABC

$ gpg -c test

```
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/573cd6e63ba2f4ade280f0e90e550b9a.png)

```bash
$ ls 
test  test.gpg
$ rm -rf test
$ cat test.gpg 
�       d�#���S%��>����b����c�\�G
                                 ���ﮇR��L<"P`�iT�}��    ��:�D��?�߼l�d~�vw�r

$ gpg -d test.gpg > test
#输出密码

$ cat test
ABC
```

更多阅读：

 - [GPG入门教程](https://www.ruanyifeng.com/blog/2013/07/gpg.html)

----
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e90293c4165484b6736e03b6c17fef2c.gif#pic_center)

