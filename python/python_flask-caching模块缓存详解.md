#   python 模块 flask-caching 缓存





-----
## 1. 介绍
为了尽量减少缓存穿透，同时减少web的响应时间，我们可以针对那些需要一定时间才能获取结果的函数和那些不需要频繁更新的视图函数提供缓存服务，可以在一定的时间内直接返回结果而不是每次都需要计算或者从数据库中查找。flask_caching插件就是提供这种功能的神器。
## 2. 安装

```python
$ easy_install Flask-Caching

$ pip install Flask-Caching
```


## 3. 配置参数

```python
CACHE_TYPE:设置缓存的类型

# 下面五个参数是所有的类型共有的
CACHE_NO_NULL_WARNING = "warning" # null类型时的警告消息
CACHE_ARGS = []    # 在缓存类实例化过程中解包和传递的可选列表，用来配置相关后端的额外的参数
CACHE_OPTIONS = {}    # 可选字典,在缓存类实例化期间传递，也是用来配置相关后端的额外的键值对参数
CACHE_DEFAULT_TIMEOUT # 默认过期/超时时间，单位为秒
CACHE_THRESHOLD    # 缓存的最大条目数

CACHE_TYPE = null # 默认的缓存类型，无缓存
CACHE_TYPE = 'simple' # 使用本地python字典进行存储，线程非安全

CACHE_TYPE = 'filesystem' # 使用文件系统来存储缓存的值
CACHE_DIR = "" # 文件目录

CACHE_TYPE = 'memcached' # 使用memcached服务器缓存
CACHE_KEY_PREFIX # 设置cache_key的前缀
CAHCE_MEMCACHED_SERVERS    # 服务器地址的列表或元组
CACHE_MEMCACHED_USERNAME # 用户名
CACHE_MEMCACHED_PASSWORD # 密码

CACHE_TYPE = 'uwsgi' # 使用uwsgi服务器作为缓存
CACHE_UWSGI_NAME # 要连接的uwsgi缓存实例的名称

CACHE_TYPE = 'redis' # 使用redis作为缓存
CACHE_KEY_PREFIX # 设置cache_key的前缀
CACHE_REDIS_HOST  # redis地址
CACHE_REDIS_PORT  # redis端口
CACHE_REDIS_PASSWORD # redis密码
CACHE_REDIS_DB # 使用哪个数据库
# 也可以一键配置
CACHE_REDIS_URL    连接到Redis服务器的URL。示例redis://user:password@localhost:6379/2
```

## 4. cache方法

```python
cache.cached：装饰器，装饰无参数函数，使得该函数结果可以缓存
参数:
timeout:超时时间
key_prefix：设置该函数的标志
unless：设置是否启用缓存，如果为True，不启用缓存
forced_update：设置缓存是否实时更新，如果为True，无论是否过期都将更新缓存
query_string：为True时，缓存键是先将参数排序然后哈希的结果

cache.memoize：装饰器，装饰有参数函数，使得该函数结果可以缓存
make_name：设置函数的标志，如果没有就使用装饰的函数
# 其他参数同cached

cache.delete_memoized：删除缓存
参数：
fname：缓存函数的名字或引用
*args：函数参数

cache.clear() # 清除缓存所有的缓存，这个操作需要慎重
cache.cache # 获取缓存对象

#获取某个网页是否存在缓存，key值如'view//gbook.html'
cache.cache.has('view/{}'.format(request.path))
#打印该缓存
print(request.path,cache.get('view/{}'.format(request.path)))
#删除该缓存
cache.delete('view//gbook.html')
```


##   5. 显示缓存存储
利用`cache.get`与`cache.set`方法
```python
from flask import Flask, request , render_template_string
from flask_caching import Cache  


app = Flask(__name__)
cache = Cache(app, config={'CACHE_TYPE': 'simple'})

@app.route("/html")  
@app.route("/html/<foo>")  
def html(foo=None):  
    if foo is not None:  
       cache.set("foo", foo)  
       bar = cache.get("foo")  
    return render_template_string(  
    "<html><body>foo cache: {{bar}}</body></html>", bar=bar  
    )

if __name__ == '__main__':
   app.run(host='0.0.0.0')  
```
执行`python cache1.py`， 访问：`http://127.0.0.1:5000/html/hello`

```python
foo cache: hello
```
##  6. 缓存20秒效果
使用装饰器 cached() 能够缓存等待
```python
#---coding:utf-8
from flask import Flask, request  
from flask_caching import Cache  
  
app = Flask(__name__)  
# simple使用字典存储  
cache = Cache(app, config={'CACHE_TYPE': 'simple'})  
  
@app.route('/index')  
@cache.cached(timeout=20)  
def index():  
    print(request.path)  
    s = 'test cache'  
    cache.set('b',123)  
    print('test cache')  
  
    return s  
  
@app.route('/test')  
def test():  
    print(cache.get('b'))  
    return 'ok'  
  
if __name__ == '__main__':  
   app.run(host='0.0.0.0')  
```
执行`python cache2.py` ,访问：`http://127.0.0.1:5000/index`
界面输出，多次刷新

```python
test cache
```
日志输出：
```python
test cache
192.168.211.1 - - [09/Jul/2020 18:59:39] "GET /index HTTP/1.1" 200 -
192.168.211.1 - - [09/Jul/2020 18:59:42] "GET /index HTTP/1.1" 200 -
192.168.211.1 - - [09/Jul/2020 18:59:43] "GET /index HTTP/1.1" 200 -
192.168.211.1 - - [09/Jul/2020 18:59:43] "GET /index HTTP/1.1" 200 -
192.168.211.1 - - [09/Jul/2020 18:59:46] "GET /index HTTP/1.1" 200 -
192.168.211.1 - - [09/Jul/2020 18:59:47] "GET /index HTTP/1.1" 200 -
192.168.211.1 - - [09/Jul/2020 18:59:49] "GET /index HTTP/1.1" 200 -
192.168.211.1 - - [09/Jul/2020 18:59:49] "GET /index HTTP/1.1" 200 -
192.168.211.1 - - [09/Jul/2020 18:59:51] "GET /index HTTP/1.1" 200 -
192.168.211.1 - - [09/Jul/2020 18:59:52] "GET /index HTTP/1.1" 200 -
192.168.211.1 - - [09/Jul/2020 18:59:53] "GET /index HTTP/1.1" 200 -
192.168.211.1 - - [09/Jul/2020 18:59:54] "GET /index HTTP/1.1" 200 -
192.168.211.1 - - [09/Jul/2020 18:59:55] "GET /index HTTP/1.1" 200 -
192.168.211.1 - - [09/Jul/2020 18:59:56] "GET /index HTTP/1.1" 200 -
192.168.211.1 - - [09/Jul/2020 18:59:56] "GET /index HTTP/1.1" 200 -
192.168.211.1 - - [09/Jul/2020 18:59:57] "GET /index HTTP/1.1" 200 -
192.168.211.1 - - [09/Jul/2020 18:59:58] "GET /index HTTP/1.1" 200 -
/index
test cache
192.168.211.1 - - [09/Jul/2020 18:59:59] "GET /index HTTP/1.1" 200 -
```
访问`http://127.0.0.1:5000/test`，多次刷新
界面输出：ok

日志输出：

```python
123
192.168.211.1 - - [09/Jul/2020 19:01:27] "GET /test HTTP/1.1" 200 -
123
192.168.211.1 - - [09/Jul/2020 19:01:27] "GET /test HTTP/1.1" 200 -
123
192.168.211.1 - - [09/Jul/2020 19:01:27] "GET /test HTTP/1.1" 200 -
123
192.168.211.1 - - [09/Jul/2020 19:01:28] "GET /test HTTP/1.1" 200 -
```



参考：

 - [FLASK插件之使用FLASK_CACHING缓存](https://www.heanny.cn/post-485.html)
 - [linux python web flask Hello World实战](https://blog.csdn.net/xixihahalelehehe/article/details/106111115?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522164017965816780265478768%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fblog.%2522%257D&request_id=164017965816780265478768&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_ecpm_v1~rank_v31_ecpm-9-106111115.nonecase&utm_term=flask&spm=1018.2226.3001.4450)
 - [windows python web flask Hello World实战](https://ghostwritten.blog.csdn.net/article/details/106864137)
 - [windows python web flask 模板开放实战](https://ghostwritten.blog.csdn.net/article/details/106889489)
 - [windows python web flask获取请求参数数据](https://ghostwritten.blog.csdn.net/article/details/106888653)
 - [windows python flask返回json数据](https://ghostwritten.blog.csdn.net/article/details/107428589)
 - [windows python flask与mysql数据库写入查询显示等操作详解](https://ghostwritten.blog.csdn.net/article/details/107431748)

