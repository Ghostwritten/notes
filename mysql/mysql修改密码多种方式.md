## 第一种
格式：

```bash
mysql> set password for 用户名@localhost = password('新密码');  
```
示例
```bash
mysql> set password for root@localhost = password('123');  
```
## 第二种
格式：

```bash
mysqladmin -u用户名 -p旧密码 password 新密码  
```

 
例子：

```bash
mysqladmin -uroot -p123456 password 123  
```
## 第三种

```bash
mysql> use mysql;   
mysql> update user set password=password('123') where user='root' and host='localhost';  
mysql> flush privileges;  
```


