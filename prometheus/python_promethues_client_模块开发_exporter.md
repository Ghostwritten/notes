#  python 模块 promethues_client 开发 exporter

![在这里插入图片描述](https://img-blog.csdnimg.cn/a74399206b294db89a8fbc0dc5478281.png)




## 1. 介绍
promethues_client模块为开放prometheus客户端，自定义监控指标。
## 2. 安装

```css
pip install prometheus_client
```
## 3. 如何启动一个prometheus client
### 3.1 start_http_server方法

```python
#---coding:utf-8

from prometheus_client import Gauge,start_http_server  
import random  
  
from prometheus_client import Gauge  
a = Gauge('a', 'Description of gauge')  
a.set(345)   #value自己定义，但是一定要为 整数或者浮点数  
  
g = Gauge('g', 'Description of gauge',['mylabelname'])  
  
#此时一定要注意,定义Gague标签的时候是一个列表,列表可以存多个lablename,类型是字符串  
#在给lable定义value的时候也要注意,mylablename 这里是一个方法,或者说是一个变量了,一定要注意.  
  
start_http_server(8000)  
while True:  
      g.labels(mylabelname='ghost').set(random.random()) 
```
运行`python client1.py`,访问:`http://127.0.0.1:8000`

```python
.....
python_info{implementation="CPython",major="2",minor="7",patchlevel="5",version="2.7.5"} 1.0
# HELP a Description of gauge
# TYPE a gauge
a 345.0
......
# TYPE g gauge
g{mylabelname="ghost"} 0.4350908628997736
```
### 3.2 Flask编写promtheus client

```python
import prometheus_client
from prometheus_client import Counter
from flask import Response, Flask
app = Flask(__name__)
requests_total = Counter("request_count_xxx", "Total request cout of the host")
@app.route("/metrics")
def requests_count():
  requests_total.inc()
  # requests_total.inc(2)
  return Response(prometheus_client.generate_latest(requests_total),
          mimetype="text/plain")
@app.route('/')
def index():
  requests_total.inc()
  return "Welcome to China"
if __name__ == "__main__":
  app.run(host="0.0.0.0")
```
访问`http://127.0.0.1:5000`，返回：

```python
Welcome to China
```

访问`http://127.0.0.1:5000metrics`，返回：

```python
# HELP request_count_total Total request cout of the host
# TYPE request_count_total counter
request_count_xxx_total 2.0
# HELP request_count_created Total request cout of the host
# TYPE request_count_created gauge
request_count_xxx_created 1.594264948484944e+09
```
## 4. LABELS配置
使用labels来区分metric的特征
```python
from prometheus_client import Counter
c = Counter('requests_total', 'HTTP requests total', ['method', 'clientip'])
c.labels('get', '127.0.0.1').inc()
c.labels('post', '192.168.0.1').inc(3)
c.labels(method="get", clientip="192.168.0.1").inc()
```
## 5. REGISTRY

```python
from prometheus_client import Counter, Gauge
from prometheus_client.core import CollectorRegistry
REGISTRY = CollectorRegistry(auto_describe=False)
requests_total = Counter("request_count", "Total request cout of the host", registry=REGISTRY)
random_value = Gauge("random_value", "Random value of the request", registry=REGISTRY)
```

## 6. Metrics自定义
Prometheus提供4种类型Metrics：`Counter`, `Gauge`, `Summary`和`Histogram`
定义4种metrics例子

```python
metrics名字, metrics说明, metrics支持的label
c = Counter('cc', 'A counter')
g = Gauge('gg', 'A gauge')
h = Histogram('hh', 'A histogram', buckets=(-5, 0, 5))
s = Summary('ss', 'A summary', ['label1', 'label2'])  
```

### 6.1 Counter
一直累加的计数器，不可以减少。
以上面代码为例，多次访问`http://127.0.0.1:5000metrics`，`request_count_xxx_total`持续增加。
定义它需要2个参数，第一个是metrics的名字，第二个是metrics的描述信息：

	
```
c = Counter('cc', 'A counter')
```

它的唯一方法就是inc，只允许增加不允许减少：

```
 def inc(self, amount=1):
        '''Increment counter by the given amount.'''
        if amount < 0:
            raise ValueError('Counters can only be incremented by non-negative amounts.')
        self._value.inc(amount)
```
counter适合用来记录访问总次数之类的，通过promql可以计算counter的增长速率，即可以得到类似的QPS诸多指标。
调用

```
# counter:  只增不减
c.inc()
```
输出metrics

```python
# HELP cc_total A counter
# TYPE cc_total counter
cc_total 46.0
# TYPE cc_created gauge
cc_created 1.546424546634121e+09
```
说明：

 - #HELP是cc的注释说明，我们刚才定义的时候指定的；
 - #TYPE说明cc是一个counter；
 - cc这个counter被输出为cc_total；
 - cc_created这个输出的TYPE是gauge类型，记录了cc这个metrics的创建时间。

### 6.2 gauge
gauge可增可减，可以任意设置，就代表了某个指标当前的值而已。
比如可以设置当前的CPU温度，内存使用量等等，它们都是上下浮动的，不是只增不减的。

定义：第一个是metrics的名字，第二个是描述。
	
```python
g = Gauge('gg', 'A gauge')
```
方法：

```python
def inc(self, amount=1):
      '''Increment gauge by the given amount.'''
      self._value.inc(amount)

def dec(self, amount=1):
      '''Decrement gauge by the given amount.'''
      self._value.inc(-amount)

 def set(self, value):
      '''Set gauge to the given value.'''
      self._value.set(float(value))
```
调用：
这里每次设置一个随机值，其值可以是任意浮点数：

```python
 # gauge: 任意值
 g.set(random.random())
```
输出metrics:

```python
# HELP gg A gauge
# TYPE gg gauge
gg 0.935768437404028
```
### 6.3 histogram
主要用来统计百分位。
比如你有100条访问请求的耗时时间，把它们从小到大排序，第90个时间是200ms，那么我们可以说90%的请求都小于200ms，这也叫做”90分位是200ms”，能够反映出服务的基本质量。当然，也许第91个时间是2000ms，这就没法说了。

实际情况是，我们每天访问量至少几个亿，不可能把所有访问数据都存起来，然后排序找到90分位的时间是多少。因此，类似这种问题都采用了一些估算的算法来处理，不需要把所有数据都存下来。

定义：

```python
h = Histogram('hh', 'A histogram', buckets=(-5, 0, 5))
```
第一个是metrics的名字，第二个是描述，第三个是分桶设置，重点说一下buckets。
这里(-5,0,5)实际划分成了几种桶：<=-5，<=0，<=5，<=无穷大。
方法:

```python
while True:
    # histogram: 任意值, 会给符合条件的bucket增加1次计数
    h.observe(random.randint(-5, 5))
```

输出metrics：

```python
# HELP hh A histogram
# TYPE hh histogram
hh_bucket{le="-5.0"} 12.0
hh_bucket{le="0.0"} 83.0
hh_bucket{le="5.0"} 153.0
hh_bucket{le="+Inf"} 153.0
hh_count 153.0
hh_sum -16.0
# TYPE hh_created gauge
hh_created 1.546499508889123e+09
```
描述：

 - hh_sum记录了observe的总和；
 - count记录了observe的次数；
 - bucket就是各种桶了；
 - le表示<=某值。
### 6.4 summary
略




参考：

 - [https://prometheus.io/docs/practices/histograms/](https://prometheus.io/docs/practices/histograms/)
 - [鱼儿的博客](https://yuerblog.cc/2019/01/03/prometheus-client-usage-and-principle/?unapproved=1309&moderation-hash=3d28f1afb1e72827de3063caf093903e#comment-1309)
 - [用Python编写Prometheus监控的方法](https://www.jb51.net/article/148897.htm)
 - [Python prometheus_client使用方式](https://blog.csdn.net/mywmy/article/details/103109561)
 - [Python Metric 示例](https://python.hotexamples.com/zh/examples/prometheus_client/Metric/-/python-metric-class-examples.html)
 - [prometheus_client.Counter](https://programtalk.com/python-examples/prometheus_client.Counter/)
