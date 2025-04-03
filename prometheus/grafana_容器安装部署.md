

-----
## 1. 部署
创建存储目录
```bash
mkdir /var/lib/grafana

chmod 777 /var/lib/grafana
```


qq企业邮箱的配置

$ cat /root/grafana/grafana.ini  

```bash
; #################################### SMTP / Emailing ##########################
[smtp]
enabled = true
host = smtp.exmail.qq.com:465  #qq企业发件服务器
user =XX将此邮箱作为发件邮箱XX   
; # If the password contains # or ; you have to wrap it with trippel quotes. Ex """#password;"""
password =XX自己的密码XX
; ;cert_file =
; ;key_file =
skip_verify = true
from_address = XX发件邮箱（同user）XX
; ;from_name = Grafana
; # EHLO identity in SMTP dialog (defaults to instance_name)
; ;ehlo_identity = dashboard.example.com
;
; [emails]
; ;welcome_email_on_sign_up = false
;
; #################################### Logging ##########################
```


容器创建
```bash
$ docker run -d   -p 3000:3000 -v -e "GF_SECURITY_ADMIN_PASSWORD=123456" --name grafana docker.io/grafana/grafana:5.0.4 # 简单的测试部署

docker run -d -p 3000:3000 -v /var/lib/grafana:/var/lib/grafana  -v /root/grafana/grafana.ini:/etc/grafana/grafana.ini  -e "GF_SERVER_ROOT_URL=http://grafana.server.com" -e "GF_SECURITY_ADMIN_PASSWORD=123456" --name grafana docker.io/grafana/grafana:5.0.4 #生产持久方便修改配置的部署
```

 - 来自默认配置 $WORKING_DIR/conf/defaults.ini
 - 自定义配置 $WORKING_DIR/conf/custom.ini
 - 自定义配置文件路径可以使用--config参数覆盖

访问grafana:`http://grafana.server.co`m;如果你在虚拟机做的部署，记得添加域名解析

​​​​

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6924f258e93473c1346065451129b13f.png)
## 2. 配置文件说明
app_mode：应用名称，默认是production
 

```bash
[path]
data：一个grafana用来存储sqlite3、临时文件、回话的地址路径
logs：grafana存储logs的路径
 
[server]
http_addr：监听的ip地址，，默认是0.0.0.0
http_port：监听的端口，默认是3000
protocol：http或者https，，默认是http
domain：这个设置是root_url的一部分，当你通过浏览器访问grafana时的公开的domian名称，默认是localhost
enforce_domain：如果主机的header不匹配domian，则跳转到一个正确的domain上，默认是false
root_url：这是一个web上访问grafana的全路径url，默认是%(protocol)s://%(domain)s:%(http_port)s/
router_logging：是否记录web请求日志，默认是false
cert_file：如果使用https则需要设置
cert_key：如果使用https则需要设置
 
[database]
grafana默认需要使用数据库存储用户和dashboard信息，默认使用sqlite3来存储，你也可以换成其他数据库
type：可以是mysql、postgres、sqlite3，默认是sqlite3
path：只是sqlite3需要，定义sqlite3的存储路径
host：只是mysql、postgres需要，默认是127.0.0.1:3306
name：grafana的数据库名称，默认是grafana
user：连接数据库的用户
password：数据库用户的密码
ssl_mode：只是postgres使用
```

## 3. 安装插件
命令行安装插件：

```bash
grafana-cli plugins install grafana-polystat-panelanel grafana-polystat-panel
```
## 4. 修改密码

```bash
grafana-cli admin reset-admin-password --homepath "/usr/share/grafana" newpass
```

如果你没有丢失admin密码，最好是在grafana的UI中设置。如果你想在脚本中设置密码，那么可以使用grafana的API，这里有个使用curl执行的例子：

```bash
curl -X PUT -H "Content-Type: application/json" -d '{
  "oldPassword": "admin",
  "newPassword": "newpass",
  "confirmNew": "newpass"
}' http://admin:admin@<your_grafana_host>:3000/api/user/password
```

