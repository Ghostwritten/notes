
#  linux Command curl 
tags: 网络



---

##  1. 参数
不带有任何参数时，curl 就是发出 GET 请求。

```bash
$ curl https://www.example.com
```

上面命令向www.example.com发出 GET 请求，服务器返回的内容会在命令行输出。

### 1.1 -A

-A参数指定客户端的用户代理标头，即User-Agent。curl 的默认用户代理字符串是curl/[version]。

```bash
$ curl -A 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/76.0.3809.100 Safari/537.36' https://google.com
```

上面命令将User-Agent改成 Chrome 浏览器。

```bash
$ curl -A '' https://google.com
```

上面命令会移除User-Agent标头。

也可以通过-H参数直接指定标头，更改User-Agent。

```bash
$ curl -H 'User-Agent: php/1.0' https://google.com
```

### 1.2 -b

-b参数用来向服务器发送 Cookie。

```bash
$ curl -b 'foo=bar' https://google.com
```

上面命令会生成一个标头Cookie: foo=bar，向服务器发送一个名为foo、值为bar的 Cookie。

```bash
$ curl -b 'foo1=bar' -b 'foo2=baz' https://google.com
```

上面命令发送两个 Cookie。

```bash
$ curl -b cookies.txt https://www.google.com
```

上面命令读取本地文件cookies.txt，里面是服务器设置的 Cookie（参见-c参数），将其发送到服务器。

### 1.3 -c

-c参数将服务器设置的 Cookie 写入一个文件。

```bash
$ curl -c cookies.txt https://www.google.com
```

上面命令将服务器的 HTTP 回应所设置 Cookie 写入文本文件cookies.txt。

### 1.4 -d

-d参数用于发送 POST 请求的数据体。

```bash
$ curl -d'login=emma＆password=123'-X POST https://google.com/login
$ curl http://127.0.0.1:8080/check_your_status?user=Summer&passwd=12345678
```

#或者

```bash
$ curl -d 'login=emma' -d 'password=123' -X POST  https://google.com/login
```

使用-d参数以后，HTTP 请求会自动加上标头Content-Type : application/x-www-form-urlencoded。并且会自动将请求转为 POST 方法，因此可以省略-X POST。

-d参数可以读取本地文本文件的数据，向服务器发送。

```bash
$ curl -d '@data.txt' https://google.com/login
```

上面命令读取data.txt文件的内容，作为数据体向服务器发送。

--data-urlencode

--data-urlencode参数等同于-d，发送 POST 请求的数据体，区别在于会自动将发送的数据进行 URL 编码。

```bash
$ curl --data-urlencode 'comment=hello world' https://google.com/login
```

上面代码中，发送的数据hello world之间有一个空格，需要进行 URL 编码。


Curl命令提供了若干个设置HTTP POST数据的选项，这里比较如下：

```bash
-d,--data <key=value>
```

将HTTP POST请求中的数据发送给HTTP服务器，与用户提交HTML表单时浏览器的行为完全一样。
默认Content-type为application/x-www-form-urlencoded
@file_name,表示数据来自一个文件，文件中的回车符和换行符将被转换
-,表示数据来自stdin，即标准输入设备

```bash
--data-ascii <key=value>
```

等价于-d

```bash
--data-binary <key=value>
```

HTTP POST请求中的数据为纯二进制数据
如果是@file_name,则保留文件中的回车符和换行符，不做任何转换

```bash
--data-raw <key=value>
```

不处理@字符，即@not_as_a_file_name
其他等价于-d

```bash
--data-urlencode <key=value>
```

先对数据进行URL编码，再发送给HTTP服务器，即对表单中的字段值进行URL编码然后再发送。
为了兼容CGI，格式为name=content：
name=content,将content进行URL编码，然后发送给HTTP服务器
=content,同上
content,同上，但是content中不能包含=，@符号
name@filename,从文件中读取数据（包括换行符），将读取的数据进行URL编码，然后发送给HTTP服务器
@filename,同上
其他等价于-d



### 1.5 -e

-e参数用来设置 HTTP 的标头Referer，表示请求的来源。

```bash
curl -e 'https://google.com?q=example' https://www.example.com
```

上面命令将Referer标头设为https://google.com?q=example。

-H参数可以通过直接添加标头Referer，达到同样效果。

```bash
curl -H 'Referer: https://google.com?q=example' https://www.example.com
```

### 1.6 -F

-F参数用来向服务器上传二进制文件。

```bash
$ curl -F 'file=@photo.png' https://google.com/profile
```

上面命令会给 HTTP 请求加上标头Content-Type: multipart/form-data，然后将文件photo.png作为file字段上传。

-F参数可以指定 MIME 类型。

```bash
$ curl -F 'file=@photo.png;type=image/png' https://google.com/profile
```

上面命令指定 MIME 类型为image/png，否则 curl 会把 MIME 类型设为application/octet-stream。

-F参数也可以指定文件名。

```bash
$ curl -F 'file=@photo.png;filename=me.png' https://google.com/profile
```

上面命令中，原始文件名为photo.png，但是服务器接收到的文件名为me.png。

### 1.7 -G

-G参数用来构造 URL 的查询字符串。

```bash
$ curl -G -d 'q=kitties' -d 'count=20' https://google.com/search
```

上面命令会发出一个 GET 请求，实际请求的 URL 为https://google.com/search?q=kitties&count=20。如果省略--G，会发出一个 POST 请求。

如果数据需要 URL 编码，可以结合`--data--urlencode`参数。

```bash
$ curl -G --data-urlencode 'comment=hello world' https://www.example.com
```

### 1.8 -H

-H参数添加 HTTP 请求的标头。

```bash
$ curl -H 'Accept-Language: en-US' https://google.com
```

上面命令添加 HTTP 标头`Accept-Language: en-US`。

```bash
$ curl -H 'Accept-Language: en-US' -H 'Secret-Message: xyzzy' https://google.com
```

上面命令添加两个 HTTP 标头。

```bash
$ curl -d '{"login": "emma", "pass": "123"}' -H 'Content-Type: application/json' https://google.com/login

curl -H "Content-Type:application/json" -X POST --data '{"message": "sunshine"}' http://localhost:8000/
```

上面命令添加 HTTP 请求的标头是`Content-Type: application/json`，然后用-d参数发送 `JSON` 数据。

###  1.9 -i

-i参数打印出服务器回应的 HTTP 标头。

```bash
$ curl -i https://www.example.com
```

上面命令收到服务器回应后，先输出服务器回应的标头，然后空一行，再输出网页的源码。

### 1.10 -I
-I参数向服务器发出 HEAD 请求，然会将服务器返回的 HTTP 标头打印出来。

```bash
$ curl -I https://www.example.com
```

上面命令输出服务器对 HEAD 请求的回应。

--head参数等同于-I。

```bash
$ curl --head https://www.example.com
```

### 1.11 -k

-k参数指定跳过 SSL 检测。

```bash
$ curl -k https://www.example.com
```

上面命令不会检查服务器的 SSL 证书是否正确。

### 1.12 -L

-L参数会让 HTTP 请求跟随服务器的重定向。curl 默认不跟随重定向。

```bash
$ curl -L -d 'tweet=hi' https://api.twitter.com/tweet
```

### 1.13 --limit-rate

--limit-rate用来限制 HTTP 请求和回应的带宽，模拟慢网速的环境。

```bash
$ curl --limit-rate 200k https://google.com
```

上面命令将带宽限制在每秒 200K 字节。

### 1.14 -o

-o参数将服务器的回应保存成文件，等同于wget命令。

```bash
$ curl -o example.html https://www.example.com
```

上面命令将www.example.com保存成example.html。

### 1.15 -O

-O参数将服务器回应保存成文件，并将 URL 的最后部分当作文件名。

```bash
$ curl -O https://www.example.com/foo/bar.html
```

上面命令将服务器回应保存成文件，文件名为bar.html。

### 1.16 -s

-s参数将不输出错误和进度信息。

```bash
$ curl -s https://www.example.com
```

上面命令一旦发生错误，不会显示错误信息。不发生错误的话，会正常显示运行结果。

如果想让 curl 不产生任何输出，可以使用下面的命令。

```bash
$ curl -s -o /dev/null https://google.com
```

### 1.17 -S

-S参数指定只输出错误信息，通常与-o一起使用。

```bash
$ curl -s -o /dev/null https://google.com
```

上面命令没有任何输出，除非发生错误。

### 1.18 -u

-u参数用来设置服务器认证的用户名和密码。

```bash
$ curl -u 'bob:12345' https://google.com/login
```

上面命令设置用户名为bob，密码为12345，然后将其转为 HTTP 标头Authorization: Basic Ym9iOjEyMzQ1。

curl 能够识别 URL 里面的用户名和密码。

```bash
$ curl https://bob:12345@google.com/login
```

上面命令能够识别 URL 里面的用户名和密码，将其转为上个例子里面的 HTTP 标头。

```bash
$ curl -u 'bob' https://google.com/login
```

上面命令只设置了用户名，执行后，curl 会提示用户输入密码。

### 1.19 -v

-v参数输出通信的整个过程，用于调试。

```bash
$ curl -v https://www.example.com
```

--trace参数也可以用于调试，还会输出原始的二进制数据。

```bash
$ curl --trace - https://www.example.com
```

### 1.20 -x

-x参数指定 HTTP 请求的代理。

```bash
$ curl -x socks5://james:cats@myproxy.com:8080 https://www.example.com
```

上面命令指定 HTTP 请求通过myproxy.com:8080的 socks5 代理发出。

如果没有指定代理协议，默认为 HTTP。

```bash
$ curl -x james:cats@myproxy.com:8080 https://www.example.com
```

上面命令中，请求的代理使用 HTTP 协议。

### 1.21 -X

-X参数指定 HTTP 请求的方法。

```bash
$ curl -X POST https://www.example.com
```

上面命令对https://www.example.com发出 POST 请求。


## 2. 返回码
shell命令：curl -I -m 10 -o /dev/null -s -w %{http_code} http://test.com


```bash
curl_init — 初始化一个curl会话
curl_copy_handle — 拷贝一个curl连接资源的所有内容和参数
curl_errno — 返回一个包含当前会话错误信息的数字编号
curl_error — 返回一个包含当前会话错误信息的字符串
curl_exec — 执行一个curl会话
curl_getinfo — 获取一个curl连接资源句柄的信息
curl_multi_init — 初始化一个curl批处理句柄资源
curl_multi_add_handle — 向curl批处理会话中添加单独的curl句柄资源
curl_multi_close — 关闭一个批处理句柄资源
curl_multi_exec — 解析一个curl批处理句柄
curl_multi_getcontent — 返回获取的输出的文本流
curl_multi_info_read — 获取当前解析的curl的相关传输信息
curl_multi_remove_handle — 移除curl批处理句柄资源中的某个句柄资源
curl_multi_select — Get all the sockets associated with the cURL extension, which can then be "selected"
curl_setopt_array — 以数组的形式为一个curl设置会话参数
curl_setopt — 为一个curl设置会话参数
curl_version — 获取curl相关的版本信息
curl_close — 关闭一个curl会话
```


curl爬取过程中，会返回一个http_code，下面是他们的意义信息

```bash
$http_code["0"]="Unable to access";
$http_code["100"]="Continue";
$http_code["101"]="Switching Protocols";
$http_code["200"]=”OK”;
$http_code["201"]=”Created”;
$http_code["202"]=”Accepted”;
$http_code["203"]=”Non-Authoritative Information”;
$http_code["204"]=”No Content”;
$http_code["205"]=”Reset Content”;
$http_code["206"]=”Partial Content”;
$http_code["300"]=”Multiple Choices”;
$http_code["301"]=”Moved Permanently”;
$http_code["302"]=”Found”;
$http_code["303"]=”See Other”;
$http_code["304"]=”Not Modified”;
$http_code["305"]=”Use Proxy”;
$http_code["306"]=”(Unused)”;
$http_code["307"]=”Temporary Redirect”;
$http_code["400"]=”Bad Request”;
$http_code["401"]=”Unauthorized”;
$http_code["402"]=”Payment Required”;
$http_code["403"]=”Forbidden”;
$http_code["404"]=”Not Found”;
$http_code["405"]=”Method Not Allowed”;
$http_code["406"]=”Not Acceptable”;
$http_code["407"]=”Proxy Authentication Required”;
$http_code["408"]=”Request Timeout”;
$http_code["409"]=”Conflict”;
$http_code["410"]=”Gone”;
$http_code["411"]=”Length Required”;
$http_code["412"]=”Precondition Failed”;
$http_code["413"]=”Request Entity Too Large”;
$http_code["414"]=”Request-URI Too Long”;
$http_code["415"]=”Unsupported Media Type”;
$http_code["416"]=”Requested Range Not Satisfiable”;
$http_code["417"]=”Expectation Failed”;
$http_code["500"]=”Internal Server Error”;
$http_code["501"]=”Not Implemented”;
$http_code["502"]=”Bad Gateway”;
$http_code["503"]=”Service Unavailable”;
$http_code["504"]=”Gateway Timeout”;
$http_code["505"]=”HTTP Version Not Supported”;
```

```bash
# -w表示只输出HTTP状态码及总时间，-o表示将响应重定向到/dev/null
$ curl -s -w 'Http code: %{http_code}\nTotal time:%{time_total}s\n' -o /dev/null http://192.168.0.30/
...
Http code: 200
Total time:0.002s


# --connect-timeout表示连接超时时间
$ curl -w 'Http code: %{http_code}\nTotal time:%{time_total}s\n' -o /dev/null --connect-timeout 10 http://192.168.0.30
...
Http code: 000
Total time:10.001s
curl: (28) Connection timed out after 10000 milliseconds
```

参考：

 - [jiapeng](https://www.cnblogs.com/hujiapeng/p/8470099.html)
 - [curl command in Linux with Examples](https://www.geeksforgeeks.org/curl-command-in-linux-with-examples/)
 - [Linux curl命令详解](https://www.cnblogs.com/duhuo/p/5695256.html)

