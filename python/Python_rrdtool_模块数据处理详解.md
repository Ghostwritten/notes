# Python  模块 rrdtool 数据处理



-----
## 1. 介绍
[rrdtool](https://pypi.org/project/rrdtool/)（round robin database）工具为`环状数据库的存储格式`，**round robin是一种处理定量数据以及当前元素指针的技术。**rrdtool主要用来跟踪对象的变化情况，生成这些变化的走势图，比如**业务的访问流量、系统性能、磁盘利用率**等趋势图，很多流行监控平台都使用到rrdtool，比较有名的为`Cacti、Ganglia、Monitorix`等。
rrdtool是一个复杂的工具，涉及较多参数概念，本节主要通过Python的rrdtool模块对rrdtool的几个常用方法进行封装，包括`create、fetch、graph、info、update`等方法。
## 2. 安装

```bash
easy_install python-rrdtool 
pip install python-rrdtool
yum install rrdtool-python
```
## 3. 方法
rrdtool模块常用的几个方法，包括create（创建rrd）、update（更新rrd）、graph（绘图）、fetch（查询rrd）等。
### 3.1 Create方法
`create filename[–start|-b start time][–step|-s step][DS：ds-name：DST：heartbeat：min：max][RRA：CF：xff：steps：rows]`方法，创建一个后缀为rrd的rrdtool数据库。
参数说明如下：
```
filename创建的rrdtool数据库文件名，默认后缀为.rrd；

–start指定rrdtool第一条记录的起始时间，必须是timestamp的格式；

–step指定rrdtool每隔多长时间就收到一个值，默认为5分钟；

DS用于定义数据源，用于存放脚本的结果的变量；

DST用于定义数据源类型，rrdtool支持COUNTER（递增类型）、DERIVE（可递增可递减类型）、ABSOLUTE（假定前一个时间间隔的值为0，再计算平均值）、GUAGE（收到值后直接存入RRA）、COMPUTE（定义一个表达式，引用DS并自动计算出某个值）5种，比如网卡流量属于计数器型，应该选择COUNTER；
 
RRA用于指定数据如何存放，我们可以把一个RRA看成一个表，保存不同间隔的统计结果数据，为CF做数据合并提供依据，定义格式为：[RRA：CF：xff：steps：rows]；

CF统计合并数据，支持AVERAGE（平均值）、MAX（最大值）、MIN（最小值）、LAST（最新值）4种方式。
```
 
### 3.2 update方法
`update filename[–template|-t ds-name[：ds-name]…]N|timestamp：value[：value…][timestamp：value[：value…]…]`方法，存储一个新值到rrdtool数据库，updatev和update类似，区别是每次插入后会返回一个状态码，以便了解是否成功（updatev用0表示成功，–1表示失败）。
参数说明如下：
```
 filename指定存储数据到的目标rrd文件名；
 
 -t ds-name[：ds-name]指定需要更新的DS名称；
 
 N|Timestamp表示数据采集的时间戳，N表示当前时间戳；
 
 value[：value…]更新的数据值，多个DS则多个值。
 ```
### 3.3 graph方法
`graph filename[-s|–start seconds][-e|–end seconds][-x|–x-grid x-axis grid and label][-y|–y-grid y-axis grid and label][–alt-y-grid][–alt-y-mrtg][–alt-autoscale][–alt-autoscale-max][–units-exponent]value[-v|–vertical-label text][-w|–width pixels][-h|–height pixels][-i|–interlaced][-f|–imginfo formatstring][-a|–imgformat GIF|PNG|GD][-B|–background value][-O|–overlay value][-U|–unit value][-z|–lazy][-o|–logarithmic][-u|–upper-limit value][-l|–lower-limit value][-g|–no-legend][-r|–rigid][–step value][-b|–base value][-c|–color COLORTAG#rrggbb][-t|–title title][DEF：vname=rrd：ds-name：CF][CDEF：vname=rpn-expression][PRINT：vname：CF：format][GPRINT：vname：CF：format][COMMENT：text][HRULE：value#rrggbb[：legend]][VRULE：time#rrggbb[：legend]][LINE{1|2|3}：vname[#rrggbb[：legend]]][AREA：vname[#rrggbb[：legend]]][STACK：vname[#rrggbb[：legend]]]`方法，根据指定的rrdtool数据库进行绘图。
参数说明如下：
```
     filename指定输出图像的文件名，默认是PNG格式；
  
     –start指定起始时间；·–end指定结束时间；
     
     –x-grid控制X轴网格线刻度、标签的位置；
     
     –y-grid控制Y轴网格线刻度、标签的位置；
     
     –vertical-label指定Y轴的说明文字；
     
     –width pixels指定图表宽度（像素）；
     
     –height pixels指定图表高度（像素）；
     
     –imgformat指定图像格式（GIF|PNG|GD）；
     
     –background指定图像背景颜色，支持#rrggbb表示法；
     
     –upper-limit指定Y轴数据值上限；
     
     –lower-limit指定Y轴数据值下限；
     
     –no-legend取消图表下方的图例；
     
     –rigid严格按照upper-limit与lower-limit来绘制；
     
     –title图表顶部的标题；
     
     DEF：vname=rrd：ds-name：CF指定绘图用到的数据源；
     
     CDEF：vname=rpn-expression合并多个值；
     
     GPRINT：vname：CF：format图表的下方输出最大值、最小值、平均值等；
     
     COMMENT：text指定图表中输出的一些字符串；
     
     HRULE：value#rrggbb用于在图表上面绘制水平线；
     
     VRULE：time#rrggbb用于在图表上面绘制垂直线；
     
     LINE{1|2|3}：vname使用线条来绘制数据图表，{1|2|3}表示线条的粗细；
     
     AREA：vname使用面积图来绘制数据图表。
```
### 3.3 fetch方法
fetch filename CF[–resolution|-r resolution][–start|-s start][–end|-e end]方法，根据指定的rrdtool数据库进行查询
关键参数说明如下：
```
filename指定要查询的rrd文件名；

CF包括AVERAGE、MAX、MIN、LAST，要求必须是建库时RRA中定义的类型，否则会报错；

–start–end指定查询记录的开始与结束时间，默认可省略。
```

## 4. 实践
### 4.1 实现网卡流量图表绘制
在日常运营工作当中，观察数据的变化趋势有利于了解我们的服务质量，比如在系统监控方面，网络流量趋势图直接展现了当前网络的吞吐。CPU、内存、磁盘空间利用率趋势则反映了服务器运行健康状态。通过这些数据图表管理员可以提前做好应急预案，对可能存在的风险点做好防范。本次实践通过rrdtool模块实现服务器网卡流量趋势图的绘制，即先通过create方法创建一个rrd数据库，再通过update方法实现数据的写入，最后可以通过graph方法实现图表的绘制，以及提供last、first、info、fetch方法的查询。图3-12为rrd创建到输出图表的过程。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f093ae99b085d046584f241e522ccc07.png)
#### 4.1.1 第一步： `create`方法创建rrd数据库
采用`create`方法创建rrd数据库，参数指定了一个rrd文件、更新频率step、起始时间–start、数据源DS、数据源类型DST、数据周期定义RRA等。
【/python/rrdtool/create.py】

```python
# -*- coding: utf-8 -*-  
#!/usr/bin/python  
import rrdtool  
import time  
 
cur_time=str(int(time.time()))    #获取当前Linux时间戳作为rrd起始时间  
#数据写频率--step为300秒(即5分钟一个数据点)  
rrd=rrdtool.create('Flow.rrd','--step','300','--start',cur_time,  
#定义数据源eth1_in(入流量)、eth1_out(出流量)；类型都为COUNTER(递增)；600秒为心跳值，  
#其含义是600秒没有收到值，则会用UNKNOWN代替；0为最小值；最大值用U代替，表示不确定  
  'DS:eth1_in:COUNTER:600:0:U',  
  'DS:eth1_out:COUNTER:600:0:U',  
 
  #RRA定义格式为[RRA:CF:xff:steps:rows]，CF定义了AVERAGE、MAX、MIN三种数据合并方式  
  #xff定义为0.5，表示一个CDP中的PDP值如超过一半值为UNKNOWN，则该CDP的值就被标为UNKNOWN  
  #下列前4个RRA的定义说明如下，其他定义与AVERAGE方式相似，区别是存最大值与最小值  
  # 每隔5分钟(1*300秒)存一次数据的平均值,存600笔，即2.08天  
  # 每隔30分钟(6*300秒)存一次数据的平均值,存700笔，即14.58天（2周）  
  # 每隔2小时(24*300秒)存一次数据的平均值,存775笔，即64.58天（2个月）  
  # 每隔24小时(288*300秒)存一次数据的平均值,存797笔，即797天(2年)  
  'RRA:AVERAGE:0.5:1:600',  
  'RRA:AVERAGE:0.5:6:700',  
  'RRA:AVERAGE:0.5:24:775',  
  'RRA:AVERAGE:0.5:288:797',  
  'RRA:MAX:0.5:1:600',  
  'RRA:MAX:0.5:6:700',  
  'RRA:MAX:0.5:24:775',  
  'RRA:MAX:0.5:444:797',  
  'RRA:MIN:0.5:1:600',  
  'RRA:MIN:0.5:6:700',  
  'RRA:MIN:0.5:24:775',  
  'RRA:MIN:0.5:444:797')  
if rrd:  
    print rrdtool.error()
```

####  4.1.2 第二步：采用updatev方法更新rrd数据库
　采用updatev方法更新rrd数据库，参数指定了当前的Linux时间戳，以及指定eth0_in、eth0_out值（当前网卡的出入流量），网卡流量我们通过psutil模块来获取，如psutil.net_io_counters（）[1]为入流量。详细源码如下：
　【/python/rrdtool/update.py】

```python
# -*- coding: utf-8 -*-  
#!/usr/bin/python  
import rrdtool  
import time,psutil  
 
total_input_traffic = psutil.network_io_counters()[1]    #获取网卡入流量  
total_output_traffic = psutil.network_io_counters()[0]    #获取网卡出流量  
starttime=int(time.time())    #获取当前Linux时间戳  
#将获取到的三个数据作为updatev的参数，返回{'return_value': 0L}则说明更新成功，反之失败  
update=rrdtool.updatev('/root/python/rddtool/Flow.rrd','%s:%s:%s' % (str(starttime),str(total_input_traffic),str(total_output_traffic)))  
print update
```
将代码加入crontab，并配置5分钟作为采集频率，crontab配置如下：


```bash
   */5 * * * * /usr/bin/python /root/python/rddtool/update.py > /dev/null 2>&1 
```

#### 4.1.3 第三步　采用graph方法绘制图表
此示例中关键参数使用了–x-grid定义X轴网格刻度；DEF指定数据源；使用CDEF合并数据；HRULE绘制水平线（告警线）；GPRINT输出最大值、最小值、平均值等。详细源码如下：

【/python/rrdtool/graph.py】

```python
# -*- coding: utf-8 -*-  
#!/usr/bin/python  
import rrdtool  
import time  
#定义图表上方大标题  
title="Server network  traffic flow ("+time.strftime('%Y-%m-%d',time.localtime(time.time()))+")"  
#重点解释"--x-grid","MINUTE:12:HOUR:1:HOUR:1:0:%H"参数的作用（从左往右进行分解）  
"MINUTE:12" #表示控制每隔12分钟放置一根次要格线  
"HOUR:1"  #表示控制每隔1小时放置一根主要格线  
"HOUR:1" # 表示控制1个小时输出一个label标签  
"0:%H"  #0表示数字对齐格线，%H表示标签以小时显示  
rrdtool.graph( "Flow.png", "--start", "-1d","--vertical-label=Bytes/s",
"--x-grid","MINUTE:12:HOUR:1:HOUR:1:0:%H",
 "--width","650","--height","230","--title",title,  
 "DEF:inoctets=Flow.rrd:eth1_in:AVERAGE",    #指定网卡入流量数据源DS及CF  
 "DEF:outoctets=Flow.rrd:eth1_out:AVERAGE",    #指定网卡出流量数据源DS及CF  
 "CDEF:total=inoctets,outoctets,+",    #通过CDEF合并网卡出入流量，得出总流量total  
 
"LINE1:total#FF8833:Total traffic",    #以线条方式绘制总流量  
 "AREA:inoctets#00FF00:In traffic",    #以面积方式绘制入流量  
 "LINE1:outoctets#0000FF:Out traffic",    #以线条方式绘制出流量  
 "HRULE:6144#FF0000:Alarm value\\r",    #绘制水平线，作为告警线，阈值为6.1k  
 "CDEF:inbits=inoctets,8,*",    #将入流量换算成bit，即*8，计算结果给inbits  
 "CDEF:outbits=outoctets,8,*",    #将出流量换算成bit，即*8，计算结果给outbits  
"COMMENT:\\r",                    #在网格下方输出一个换行符  
 "COMMENT:\\r",  
 "GPRINT:inbits:AVERAGE:Avg In traffic\: %6.2lf %Sbps",    #绘制入流量平均值  
 "COMMENT:   ",  
 "GPRINT:inbits:MAX:Max In traffic\: %6.2lf %Sbps",    #绘制入流量最大值  
 "COMMENT:  ",  
 "GPRINT:inbits:MIN:MIN In traffic\: %6.2lf %Sbps\\r",    #绘制入流量最小值  
 "COMMENT: ",  
 "GPRINT:outbits:AVERAGE:Avg Out traffic\: %6.2lf %Sbps",    #绘制出流量平均值  
 "COMMENT: ",  
 "GPRINT:outbits:MAX:Max Out traffic\: %6.2lf %Sbps",    #绘制出流量最大值  
 "COMMENT: ",  
 "GPRINT:outbits:MIN:MIN Out traffic\: %6.2lf %Sbps\\r")    #绘制出流量最小值
```
以上代码将生成一个Flow.png文件，如图3-13所示。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/bb3bd87acb87c896f2a4ce0a6bcb08b3.png)


查看rrd文件内容有利于观察数据的结构、更新等情况.
rrdtool提供几个常用命令：
```
info查看rrd文件的结构信息，如rrdtool info Flow.rrd；

first查看rrd文件第一个数据的更新时间，如rrdtool first Flow.rrd；

last查看rrd文件最近一次更新的时间，如rrdtool last Flow.rrd；

fetch根据指定时间、CF查询rrd文件，如rrdtool fetch Flow.rrd AVERAGE （必须为大写）
```

参考：

 - [rrdpython](https://oss.oetiker.ch/rrdtool/prog/rrdpython.en.html)
 - [对python-rrdtool模块的浅研究。](https://www.cnblogs.com/jin-xin/p/6774828.html) 
 - [Storing Data with RRDTool](https://www.pythonstudio.us/system-administration/storing-data-with-rrdtool.html)

