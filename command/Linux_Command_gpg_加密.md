#  Linux Command gpg 加密
```bash
$ cat test
ABC

$ gpg -c test

```
![在这里插入图片描述](https://img-blog.csdnimg.cn/3cf048c7a3de44a8b77f238136fbce1e.png)

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
![在这里插入图片描述](https://img-blog.csdnimg.cn/38bf1860b14041b99fbf0aec108bedda.gif#pic_center)

