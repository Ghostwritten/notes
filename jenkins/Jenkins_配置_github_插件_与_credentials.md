
![在这里插入图片描述](https://img-blog.csdnimg.cn/663db059a15048ceb0c72c405dfca7b1.png)





##   添加 github credentials
###  HTTPS github credentials
![在这里插入图片描述](https://img-blog.csdnimg.cn/93c2a9903f314b14a3f566cc21460cb3.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/88d13b894b714170bef5f0df5bf50155.png)

###  ssh github credentials
"`Credentials`" -> "`System`" -> "`Global credentials`" -> "`Add Credentials`",在“`kind`”字段下，单击下拉框，选择“`SSH Username with private key`”。提供你的用户名、私钥及密码码如下:

![在这里插入图片描述](https://img-blog.csdnimg.cn/12feb422a0e5485991b8dd3ed3e4695b.png)
`Passphrase` 通过 github `Settings` > `Developer settings` > `Tokens` 生成。

Private Key 生成。
```bash
ssh-keygen -t rsa -b 4096
```

```bash
$ cat /root/.ssh/id_rsa
-----BEGIN RSA PRIVATE KEY-----
MIIEwIBAAKCAQEAm/lBmQx4ySwI6POOLyMgpJOIhhmMuJoINZFuhKfAoFOAx51g
QTo3XllsVSQL5vyGez+LZZmRcD3r3nveXDMkTLHaqdNu9qIu3eG27U8KQNmEJnfh
h3++kjRWVeo5mjmQATpfEyzpkcNZcjA9gOIwyV6YSKraYpCOkikpIytVIvq+Bz+y
l10Zb0wIv5YPDv3ZLyGcRo08oBLSv5SnmvH13Udf0I2+9MtCoeAe/LlTMPF64/wE
UWRa0IhC5EycTr1drXDktcTWpWW9xl96NA1p9S+w0UzybV6fJax3OHQatCk6TlR0
2FNn8w/1lg7UepFIEvKXM2KfI7EPaWg3zn4MswIDAQABAoIBAGBYyTekZ5xFEj/r
6yGc/lYLOGM1tyJ1K6sDahSRl0CyNDOaOFIyhu9GJ9A0ExIdXWkWqKIbCace6Oum
8oVbpgmmN59+FuZM6sxu2FOg5oGGm8YzeWH8/BhOMiKiK/Djq9vGVeJ/dMxwsnkB
6YwKrP5G66S/vWTzTM5mlJ2/77hJ2l6dYqZ6LeyNzIpTYbrIE92wKP+X930ADM4L
V0lW1whbIUGgYi1qqyJBwb8PblhYZp7I06L717LL9GSzOspmR3QNMYq2Cu7xl7Ch
x4bvkbYc8We2LbcVVkI/zastP06uag8aQGBPwb0mdDM9K4b/ue/o8/lKS6+H6vCX
DjC4D+ECgYEAy0Xo/9P+jJuuCvSiTw1XNXNc4EOoEEL4RnsL49R7lL82xDpNeeBS
mY/7XkuOV0bEI5hjZ5ceX2LJOiJwgPYr3hNZE91Db2zy+uq43ZsUhG2vyf3jh/z8
68tRLj8LIrWF5Rk6v0s/2DmDHUUaLngrKL6V1vu3e1CC2q1uxEAzYrGa44w88AL6
5DPHi5n91pXdnFCVd4BGh3ahdy3MqS3GR298erU+P3jStsfCnDN/HQfTyqGay
SjSOkr/ZJ+297wHGow52gjL+O/ELcqQlL0nzj8n+sZh4xFJTNgIikz0+iLYwl41D
PwRlyV1+STSZcEzHd6S7jCOxEj9JRQuzwJmkYVECgYA12D7wBWfYNTTIhocdpwz
PGvNvB44sISzpsRvhUl3DW+kaw06y4EVzF35jGoYFFbAVw190znoanKDQ7wpbQwR
t25RJLtIOMPkU2jEyt+UdLmr9+TdkNYuX50RcwGwIAVnulcldPlPGxi40pxg69o+
/0LLYmHBfQCnSk5rZ5ojoQKBgGk2gQC+V1CH//WTOitC4nwqohxmx1+NkYTAdubG
aVdUgu3+ambqomvEkP2/UvwaFmtq7m5erkseJVOEvOoq+llWItXkOFak6qvSDfTB
WIsY96nqcC0/p0BTbm+NK1SQuYBDctRrJ5Xu1PijCtii12hXPPl4GJ0cdsNb3ED6
SpixAoGBAL8BBPWeC1NHohaB8HvtuXGulbtDg5oofa2qP+DuAWvx+sFi+1Jg7hUR
1vE1+LiV3zb4Xjb9Xzm5mYg6D/WsxBEsbJYG/cWDqFW4jjfm1r4FFhG4GkSyb5U3
JNFKDr4Qz1h9dZvyr2oHY2rGamX7Oadcn1HXa7o2gylL4hRPti5T
-----END RSA PRIVATE KEY-----
```



