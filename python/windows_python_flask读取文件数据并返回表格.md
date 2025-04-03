#  windows python flask读取文件数据并返回表格
tags: flask
<!--  catalog: ~ flask 文本数据转表格~ -->

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/af9b8c3098d08fbb93063d36213c04b3.png)




![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b9cd254d802d32d1b329552407e594f2.jpeg)


## data/puuv.txt

```python
日期  	PV	UV
2020/7/18	15000	150
2020/7/19	15001	151
2020/7/20	15002	152
2020/7/21	15003	153
2020/7/22	15004	154
2020/7/23	15005	155
2020/7/24	15006	156
2020/7/25	15007	157
2020/7/26	15008	158
2020/7/27	15009	159
2020/7/28	15010	160
2020/7/29	15011	161
2020/7/30	15012	162
2020/7/31	15013	163
2020/8/1	15014	164
2020/8/2	15015	165
2020/8/3	15016	166
2020/8/4	15017	167
2020/8/5	15018	168
2020/8/6	15019	169
2020/8/7	15020	170
```

## app.py

```python
from flask import Flask, render_template, request

app = Flask(__name__)


@app.route("/pvuv")
def pvuv():
    # read file
    data = []
    with open("./data/pvuv.txt") as fin:
        is_first_line =True
        for line in fin:
            if is_first_line:
                is_first_line = False
                continue
            line = line[:-1] #\n
            pdate, pv, uv = line.split("\t")
            data.append((pdate, pv, uv))
    return render_template("pvuv.html", data=data)
```
## templates/pvuv.html

```python
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>网站的pvuv数据</title>
</head>
<body>
<h1>网站的PVUV的数据表格<h1>
    <div>
        <table border = "1px">
            <tr style="background: #ee0000">
                <th>日期</th>
                <th>PV</th>
                <th>UV</th>
            </tr>
            {%  for row in data %}
            <tr>
                <th>{{ row[0] }}</th>
                <th>{{ row[1] }}</th>
                <th>{{ row[2] }}</th>
            </tr>
            {% endfor %}
        </table>
    </div>>

</body>
</html>
```
访问界面：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4087ee491c21ffc0e770cccd12dd8d70.png)
更多阅读：

 - [linux python web flask Hello World实战](https://blog.csdn.net/xixihahalelehehe/article/details/106111115)
 - [windows python web flask Hello World实战](https://ghostwritten.blog.csdn.net/article/details/106864137)
 - [windows python web flask 模板开放实战](https://ghostwritten.blog.csdn.net/article/details/106889489)
 - [windows python web flask获取请求参数数据](https://ghostwritten.blog.csdn.net/article/details/106888653)
 - [windows python flask返回json数据](https://ghostwritten.blog.csdn.net/article/details/107428589)
 - [windows python flask与mysql数据库写入查询显示等操作详解](https://ghostwritten.blog.csdn.net/article/details/107431748)

