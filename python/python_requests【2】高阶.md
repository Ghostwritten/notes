#  python 模块 requests (2) 处理url




---

##  1. 返回数据获取方式

```python
response.text
response.json()
response.content
```

## 2. request class 的格式

```python
import re
import requests
 
 
class HandleLaGou(object):
    def __init__(self):
        self.laGou_session = requests.session()
        self.header = {
            'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/73.0.3683.86 Safari/537.36'
        }
        self.city_list = ""
 
    #获取全国城市列表
    def handle_city(self):
        city_search = re.compile(r'zhaopin/">(.*?)</a>')
        city_url = "https://www.lagou.com/jobs/allCity.html"
        city_result = self.handle_request(method = "GET", url = city_url)
        self.city_list = city_search.findall(city_result)
 
 
    def handle_request(self, method, url, data = None, info = None):
        if method == "GET":
            response = self.laGou_session.get(url = url, headers = self.header)
        return response.text
 
 
if __name__ == '__main__':
    laGou = HandleLaGou()
    laGou.handle_city()
    print(laGou.city_list)
    pass
```

##  3. 爬取html分析标签

 - [正则表达式爬取网页信息及分析HTML标签总结](https://blog.csdn.net/Eastmount/article/details/51082253)

##  4. 模拟浏览器请求
 - [requests模拟浏览器发送请求](https://www.cnblogs.com/Summer-skr--blog/p/11396904.html) 
 

##   5. 返回列表包含的字典与元组
html内容
```kotlin
<tr >
    <td>60</td>
    <td><a>leeliao</a></td>
    <td><a>leeliao</a></td>
    <td>￥56.82 元</td>
    <td>￥56.82元</td>
    <td>2014-08-11 23:33:54</td>
    <td>手动</td>
    <td><div class='iconBidState0' /></td>
</tr>
 
 
<tr >
    <td>61</td>
    <td><a>luo321654</a></td>
    <td><a>luo321654</a></td>
    <td>￥4,000.00 元</td>
    <td>￥4,000.00元</td>
    <td>2014-08-11 23:34:32</td>
    <td>手动</td>
    <td><div class='iconBidState0' /></td>
</tr
```
代码

```python
r = re.compile(r'''<td>(?P<number>\d+)</td>.*?<td><a>(?P<name>\w+)</a></td''', re.S)
result = re.finditer(r, content)
 
print [m.groupdict() for m in result]
```
输出结果：

```python
[{'number': '60', 'name': 'leeliao'}, {'number': '61', 'name': 'luo321654'}]
```



✈<font color=	#FF4500 size=4 style="font-family:Courier New">推荐阅读：</font>

 - [python requests【1】入门](https://blog.csdn.net/xixihahalelehehe/article/details/108996025?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522164922239616781685346565%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fblog.%2522%257D&request_id=164922239616781685346565&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_ecpm_v1~rank_v31_ecpm-1-108996025.nonecase&utm_term=requests&spm=1018.2226.3001.4450)
 - [python re 正则表达式](https://ghostwritten.blog.csdn.net/article/details/106247378)
 - [python requests【2】高阶](https://ghostwritten.blog.csdn.net/article/details/124088523)
 - [w3schools Python Requests Module](https://www.w3schools.com/python/module_requests.asp)
 - [runoob Python requests 模块](https://www.runoob.com/python3/python-requests.html)
 - [realpython Python’s Requests Library (Guide)](https://realpython.com/python-requests/)
 - [different-behavior-between-re-finditer-and-re-findall](https://stackoverflow.com/questions/3765024/different-behavior-between-re-finditer-and-re-findall)
 - [re-findall-which-returns-a-dict-of-named-capturing-groups](https://stackoverflow.com/questions/11103856/re-findall-which-returns-a-dict-of-named-capturing-groups)
 
 
![在这里插入图片描述](https://img-blog.csdnimg.cn/dbb16cc85bae4ca6b53c1c2dd299a977.gif#pic_center)

