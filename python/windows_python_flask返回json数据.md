#  windows python flask返回json数据
tags: flask
<!--  catalog: ~flask 返回 json~ -->

![在这里插入图片描述](https://img-blog.csdnimg.cn/501d01259317413291275262667dd1e4.png)




![在这里插入图片描述](https://img-blog.csdnimg.cn/20200718144252758.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

## 实战
### data/pvuv.txt

```bash
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
### app.py

```bash
from flask import Flask, render_template, request
import json
app = Flask(__name__)
def read_pvuv_data():
    """ read pv uv data
    return: list , ele:(pdate, pv, uv)"""
    
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
    return data
@app.route("/getjson")
def getjson():
    # read json
    data = read_pvuv_data()
    # return html
    return json.dumps(data)

if __name__ == '__main__':
    app.run(host='192.168.1.4',debug=True)
```
执行，界面访问
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200718143904522.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

### test_get_json.py
利用api接口处理json文件
```bash
import requests
import json

url = "http://192.168.1.4:5000/getjson"

r = requests.get(url)

print(r.status_code)
for row in json.loads(r.text):
    print(row)
```
执行`python test_get_json.py`

```bash
200
['2020/7/18', '15000', '150']
['2020/7/19', '15001', '151']
['2020/7/20', '15002', '152']
['2020/7/21', '15003', '153']
['2020/7/22', '15004', '154']
['2020/7/23', '15005', '155']
['2020/7/24', '15006', '156']
['2020/7/25', '15007', '157']
['2020/7/26', '15008', '158']
['2020/7/27', '15009', '159']
['2020/7/28', '15010', '160']
['2020/7/29', '15011', '161']
['2020/7/30', '15012', '162']
['2020/7/31', '15013', '163']
['2020/8/1', '15014', '164']
['2020/8/2', '15015', '165']
['2020/8/3', '15016', '166']
['2020/8/4', '15017', '167']
['2020/8/5', '15018', '168']
['2020/8/6', '15019', '169']
['2020/8/7', '15020', '170']
```

更多阅读：

 - [linux python web flask Hello World实战](https://blog.csdn.net/xixihahalelehehe/article/details/106111115?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522164017965816780265478768%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fblog.%2522%257D&request_id=164017965816780265478768&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_ecpm_v1~rank_v31_ecpm-9-106111115.nonecase&utm_term=flask&spm=1018.2226.3001.4450)
 - [windows python web flask Hello World实战](https://ghostwritten.blog.csdn.net/article/details/106864137)
 - [windows python web flask 模板开放实战](https://ghostwritten.blog.csdn.net/article/details/106889489)
 - [windows python web flask获取请求参数数据](https://ghostwritten.blog.csdn.net/article/details/106888653)

