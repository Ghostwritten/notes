# python 模块 logging日志处理



----
## 1.介绍
在 Python 中有一个标准的 logging 模块，我们可以使用它来进行标注的日志记录，利用它我们可以更方便地进行日志记录，同时还可以做更方便的级别区分以及一些额外日志信息的记录，如时间、运行模块信息等。
### 1.1 架构
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200517115746736.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)


整个日志记录的框架可以分为这么几个部分：

 - Logger 记录器，暴露了应用程序代码能直接使用的接口。
 - Handler 处理器，将（记录器产生的）日志记录发送至合适的目的地。
 - Filter 过滤器，提供了更好的粒度控制，它可以决定输出哪些日志记录。
 - Formatter 格式化器，指明了最终输出中日志记录的布局。

#### 1.1.1 Logger 记录器
Logger是一个树形层级结构，在使用接口`debug，info，warn，error，critical`之前必须创建Logger实例，即创建一个记录器，如果没有显式的进行创建，则默认创建一个root logger，并应用默认的日志级别(WARN)，处理器Handler(StreamHandler，即将日志信息打印输出在标准输出上)，和格式化器Formatter(默认的格式即为第一个简单使用程序中输出的格式)。

创建方法: 

```python
logger = logging.getLogger(logger_name)
```

创建Logger实例后，可以使用以下方法进行日志级别设置，增加处理器Handler。

```python
logger.setLevel(logging.ERROR)    #设置日志级别为ERROR，即只有日志级别大于等于ERROR的日志才会输出
logger.addHandler(handler_name) # 为Logger实例增加一个处理器
logger.removeHandler(handler_name) # 为Logger实例删除一个处理器
```
#### 1.1.2 Handler 处理器
Handler处理器类型有很多种，比较常用的有三个，`StreamHandler`，`FileHandler`，`NullHandler`。
创建StreamHandler之后，可以通过使用以下方法设置日志级别，设置格式化器Formatter，增加或删除过滤器Filter。

```python
ch.setLevel(logging.WARN)     #指定日志级别，低于WARN级别的日志将被忽略
ch.setFormatter(formatter_name)   #设置一个格式化器formatter
ch.addFilter(filter_name)    #增加一个过滤器，可以增加多个
ch.removeFilter(filter_name)   #删除一个过滤器
```

StreamHandler
创建方法: 

```python
sh = logging.StreamHandler(stream=None)
```

FileHandler
创建方法: 

```python
fh = logging.FileHandler(filename, mode='a', encoding=None, delay=False)
```

NullHandler
NullHandler类位于核心logging包，不做任何的格式化或者输出。
本质上它是个“什么都不做”的handler，由库开发者使用。
logging 模块提供的 全部Handler 有：

```bash
StreamHandler：logging.StreamHandler；  日志输出到流，可以是 sys.stderr，sys.stdout 或者文件。
FileHandler：logging.FileHandler；    日志输出到文件。
BaseRotatingHandler：logging.handlers.BaseRotatingHandler； 基本的日志回滚方式。
RotatingHandler：logging.handlers.RotatingHandler；日志回滚方式，支持日志文件最大数量和日志文件回滚。
TimeRotatingHandler：logging.handlers.TimeRotatingHandler；日志回滚方式，在一定时间区域内回滚日志文件。
SocketHandler：logging.handlers.SocketHandler；远程输出日志到TCP/IP sockets。
DatagramHandler：logging.handlers.DatagramHandler；远程输出日志到UDP sockets。
SMTPHandler：logging.handlers.SMTPHandler；远程输出日志到邮件地址。
SysLogHandler：logging.handlers.SysLogHandler；日志输出到syslog。
NTEventLogHandler：logging.handlers.NTEventLogHandler；远程输出日志到Windows NT/2000/XP的事件日志。
MemoryHandler：logging.handlers.MemoryHandler；日志输出到内存中的指定buffer。
HTTPHandler：logging.handlers.HTTPHandler；通过”GET”或者”POST”远程输出到HTTP服务器。
```

#### 1.1.3 Formatter 格式化器
使用Formatter对象设置日志信息最后的规则、结构和内容，默认的时间格式为`%Y-%m-%d %H:%M:%S`。

创建方法: 

```python
formatter = logging.Formatter(fmt=None, datefmt=None)
```

其中，fmt是消息的格式化字符串，datefmt是日期字符串。如果不指明fmt，将使用'%(message)s'。如果不指明datefmt，将使用ISO8601日期格式。

#### 1.1.4 Filter 过滤器
Handlers和Loggers可以使用Filters来完成比级别更复杂的过滤。Filter基类只允许特定Logger层次以下的事件。例如用‘A.B’初始化的Filter允许Logger ‘A.B’, ‘A.B.C’, ‘A.B.C.D’, ‘A.B.D’等记录的事件，logger‘A.BB’, ‘B.A.B’ 等就不行。 如果用空字符串来初始化，所有的事件都接受。

创建方法:

```python
 filter = logging.Filter(name='')
```

熟悉了这些概念之后，有另外一个比较重要的事情必须清楚，即Logger是一个树形层级结构;
Logger可以包含一个或多个Handler和Filter，即Logger与Handler或Fitler是一对多的关系;
一个Logger实例可以新增多个Handler，一个Handler可以新增多个格式化器或多个过滤器，而且日志级别将会继承。

![](https://img-blog.csdnimg.cn/20200517123231979.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

## 2. 优点
总的来说 logging 模块相比 print 有这么几个优点：

 - 可以在 logging 模块中设置日志等级，在不同的版本（如开发环境、生产环境）上通过设置不同的输出等级来记录对应的日志，非常灵活。
 - print 的输出信息都会输出到标准输出流中，而 logging 模块就更加灵活，可以设置输出到任意位置，如写入文件、写入远程服务器等。
 - logging 模块具有灵活的配置和格式化功能，如配置输出当前模块信息、运行时间等，相比 print 的字符串格式化更加方便易用。

## 3. 用法
### 3.1 默认输出
高于WARNING的日志信息才会输出
```python
$ cat log1.py 
import logging

logging.basicConfig()
logging.debug('This is a debug message')
logging.info('This is an info message')
logging.warning('This is a warning message')
logging.error('This is an error message')
logging.critical('This is a critical message')
```
输出：
```bash
$ python log1.py 
WARNING:root:This is a warning message
ERROR:root:This is an error message
CRITICAL:root:This is a critical message
```
### 3.2 日志配置格式输出
配置方式

 - 显式创建记录器Logger、处理器Handler和格式化器Formatter，并进行相关设置；
 - 通过简单方式进行配置，使用basicConfig()函数直接进行配置；
 - 通过配置文件进行配置，使用fileConfig()函数读取配置文件；
 - 通过配置字典进行配置，使用dictConfig()函数读取配置信息；
 - 通过网络进行配置，使用listen()函数进行网络配置。

basicConfig关键字参数

```bash
关键字	描述
filename	创建一个FileHandler，使用指定的文件名，而不是使用StreamHandler。
filemode	如果指明了文件名，指明打开文件的模式（如果没有指明filemode，默认为'a'）。
format	handler使用指明的格式化字符串。
datefmt	使用指明的日期／时间格式。
level	指明根logger的级别。
stream	使用指明的流来初始化StreamHandler。该参数与'filename'不兼容，如果两个都有，'stream'被忽略。
```
format格式

```bash
格式	描述
%(levelno)s	打印日志级别的数值
%(levelname)s	打印日志级别名称
%(pathname)s	打印当前执行程序的路径
%(filename)s	打印当前执行程序名称
%(funcName)s	打印日志的当前函数
%(lineno)d	打印日志的当前行号
%(asctime)s	打印日志的时间
%(thread)d	打印线程id
%(threadName)s	打印线程名称
%(process)d	打印进程ID
%(message)s	打印日志信息
```



#### 3.2.1 配置输出级别并指定到文件

```python
$ cat log2.py 
#!/usr/local/bin/python
# -*- coding:utf-8 -*-
import logging

# 通过下面的方式进行简单配置输出方式与日志级别
logging.basicConfig(filename='logger.log', level=logging.INFO)

logging.debug('debug message')
logging.info('info message')
logging.warn('warn message')
logging.error('error message')
logging.critical('critical message')
```
输出：
```bash
$ python log2.py
$ cat logger.log
INFO:root:info message  #info级别也能输出
WARNING:root:warn message
ERROR:root:error message
CRITICAL:root:critical message
```
#### 3.2.2 声明了一个 Logger 对象，它就是日志输出的主类
```python
$ cat log3.py 
#!/usr/local/bin/python
# -*- coding:utf-8 -*-
import logging

# 通过下面的方式进行简单配置输出方式与日志级别
logging.basicConfig(filename='logger.log', level=logging.INFO)
logger = logging.getLogger(__name__)
logger.debug('debug message')
logger.info('info message')
logger.warn('warn message')
logger.error('error message')
logger.critical('critical message')
```
输出：
```bash
$ python log3.py
$ cat logger.log 
INFO:__main__:info message
WARNING:__main__:warn message
ERROR:__main__:error message
CRITICAL:__main__:critical message
```
#### 3.2.3 格式化输出

```python
$ cat log4.py 
import logging

logging.basicConfig(filename="logger.log", filemode="w", format="%(asctime)s %(name)s:%(levelname)s:%(message)s", datefmt="%d-%M-%Y %H:%M:%S", level=logging.DEBUG)
logger = logging.getLogger(__name__)
logger.debug('debug message')
logger.info('info message')
logger.warn('warn message')
logger.error('error message')
logger.critical('critical message')
```
输出：

```python
$ python log4.py
$ cat logger.log 
17-49-2020 13:49:16 __main__:DEBUG:debug message
17-49-2020 13:49:16 __main__:INFO:info message
17-49-2020 13:49:16 __main__:WARNING:warn message
17-49-2020 13:49:16 __main__:ERROR:error message
17-49-2020 13:49:16 __main__:CRITICAL:critical message
```

### 3.3 Handler配置
#### 3.3.1 handler配置格式输出

```python
$ cat log5.py 
import logging
 
logger = logging.getLogger(__name__)
logger.setLevel(level=logging.INFO)
handler = logging.FileHandler('output.log')
formatter = logging.Formatter('%(asctime)s - %(name)s - %(levelname)s - %(message)s')
handler.setFormatter(formatter)
logger.addHandler(handler)
 
logger.info('This is a log info')
logger.debug('Debugging')
logger.warning('Warning exists')
logger.info('Finish')
```
没有再使用 basicConfig 全局配置，而是先声明了一个 Logger 对象，然后指定了其对应的 Handler 为 FileHandler 对象，然后 Handler 对象还单独指定了 Formatter 对象单独配置输出格式，最后给 Logger 对象添加对应的 Handler 即可，最后可以发现日志就会被输出到 output.log 中.

```python
$ python log5.py 
$ cat output.log 
2020-05-17 13:58:41,039 - __main__ - INFO - This is a log info
2020-05-17 13:58:41,039 - __main__ - WARNING - Warning exists
2020-05-17 13:58:41,039 - __main__ - INFO - Finish
2020-05-17 14:01:15,909 - __main__ - INFO - This is a log info
2020-05-17 14:01:15,909 - __main__ - WARNING - Warning exists
2020-05-17 14:01:15,909 - __main__ - INFO - Finish
```
#### 3.3.2 多种handle输出

```python
$ cat log6.py 
import logging
from logging.handlers import HTTPHandler
import sys
 
logger = logging.getLogger(__name__)
logger.setLevel(level=logging.DEBUG)
 
# StreamHandler
stream_handler = logging.StreamHandler(sys.stdout)
stream_handler.setLevel(level=logging.DEBUG)
logger.addHandler(stream_handler)
 
# FileHandler
file_handler = logging.FileHandler('output.log')
file_handler.setLevel(level=logging.INFO)
formatter = logging.Formatter('%(asctime)s - %(name)s - %(levelname)s - %(message)s')
file_handler.setFormatter(formatter)
logger.addHandler(file_handler)
 
# HTTPHandler
http_handler = HTTPHandler(host='localhost:80', url='log', method='POST')
logger.addHandler(http_handler)
 
# Log
logger.info('This is a log info')
logger.debug('Debugging')
logger.warning('Warning exists')
logger.info('Finish')
```
输出：

```python
$ python log6.py
This is a log info
Debugging
Warning exists
Finish
$ cat output.log
2020-05-17 14:05:48,809 - __main__ - INFO - This is a log info
2020-05-17 14:05:48,849 - __main__ - WARNING - Warning exists
2020-05-17 14:05:48,851 - __main__ - INFO - Finish
```
### 3.4 捕获 Traceback

```python
$ cat log7.py
import logging
 
logger = logging.getLogger(__name__)
logger.setLevel(level=logging.DEBUG)
 
# Formatter
formatter = logging.Formatter('%(asctime)s - %(name)s - %(levelname)s - %(message)s')
 
# FileHandler
file_handler = logging.FileHandler('result.log')
file_handler.setFormatter(formatter)
logger.addHandler(file_handler)
 
# StreamHandler
stream_handler = logging.StreamHandler()
stream_handler.setFormatter(formatter)
logger.addHandler(stream_handler)
 
# Log
logger.info('Start')
logger.warning('Something maybe fail.')
try:
    result = 10 / 0
except Exception:
    logger.error('Faild to get result', exc_info=True)
    #logger.error('Faild to get result')
logger.info('Finished')
```
在 error() 方法中添加了一个参数，将 exc_info 设置为了 True，这样我们就可以输出执行过程中的信息了，即完整的 Traceback 信息。

```bash
$ python log7.py
2020-05-17 14:13:02,217 - __main__ - INFO - Start
2020-05-17 14:13:02,217 - __main__ - WARNING - Something maybe fail.
2020-05-17 14:13:02,217 - __main__ - ERROR - Faild to get result
Traceback (most recent call last):
  File "log7.py", line 23, in <module>
    result = 10 / 0
ZeroDivisionError: integer division or modulo by zero
2020-05-17 14:13:02,217 - __main__ - INFO - Finished
```
### 3.5 配置共享
在写项目的时候，我们肯定会将许多配置放置在许多模块下面，这时如果我们每个文件都来配置 logging 配置那就太繁琐了，logging 模块提供了父子模块共享配置的机制，会根据 Logger 的名称来自动加载父模块的配置。

```python
$ vim main.py
import logging
import core
 
logger = logging.getLogger('main')
logger.setLevel(level=logging.DEBUG)
 
# Handler
handler = logging.FileHandler('result.log')
handler.setLevel(logging.INFO)
formatter = logging.Formatter('%(asctime)s - %(name)s - %(levelname)s - %(message)s')
handler.setFormatter(formatter)
logger.addHandler(handler)
 
logger.info('Main Info')
logger.debug('Main Debug')
logger.error('Main Error')
core.run()
```
这里我们配置了日志的输出格式和文件路径，同时定义了 Logger 的名称为 main，然后引入了另外一个模块 core，最后调用了 core 的 run() 方法。

```python
$ vim core.py
import logging
 
logger = logging.getLogger('main.core')
 
def run():
    logger.info('Core Info')
    logger.debug('Core Debug')
    logger.error('Core Error')
```
这里我们定义了 Logger 的名称为 main.core，注意这里开头是 main，即刚才我们在 main.py 里面的 Logger 的名称，这样 core.py 里面的 Logger 就会复用 main.py 里面的 Logger 配置，而不用再去配置一次了.

```python
$ python main.py
$ cat result.log 
2020-05-17 14:30:48,559 - main - INFO - Main Info
2020-05-17 14:30:48,559 - main - ERROR - Main Error
2020-05-17 14:30:48,559 - main.core - INFO - Core Info
2020-05-17 14:30:48,559 - main.core - ERROR - Core Error
```

### 3.6 文件配置

常见的配置文件有 ini 格式、yaml 格式、JSON 格式，这里以yaml为例。将配置写入到配置文件，然后运行时读取配置文件里面的配置，这样是更方便管理和维护.

#### 3.6.1 yaml配置文件
```python
$ vim config.yaml
version: 1
formatters:
  brief:
    format: "%(asctime)s - %(message)s"
  simple:
    format: "%(asctime)s - %(name)s - %(levelname)s - %(message)s"
handlers:
  console:
    class : logging.StreamHandler
    formatter: brief
    level   : INFO
    stream  : ext://sys.stdout
  file:
    class : logging.FileHandler
    formatter: simple
    level: DEBUG
    filename: debug.log
  error:
    class: logging.handlers.RotatingFileHandler
    level: ERROR
    formatter: simple
    filename: error.log
    maxBytes: 10485760
    backupCount: 20
    encoding: utf8
loggers:
  main.core:
    level: DEBUG
    handlers: [console, file, error]
root:
  level: DEBUG
  handlers: [console]
```
这里我们定义了 formatters、handlers、loggers、root 等模块，实际上对应的就是各个 Formatter、Handler、Logger 的配置，参数和它们的构造方法都是相同的。
定义一个主入口文件，main.py.

```python
$ vim main.py
import logging
import core
import yaml
import logging.config
import os
import io 
 
def setup_logging(default_path='config.yaml', default_level=logging.INFO):
    path = default_path
    if os.path.exists(path):
        with io.open(path, 'r', encoding='utf-8') as f:
            config = yaml.load(f)
            logging.config.dictConfig(config)
    else:
        logging.basicConfig(level=default_level)
 
 
def log():
    logging.debug('Start')
    logging.info('Exec')
    logging.info('Finished')
 
 
if __name__ == '__main__':
    yaml_path = 'config.yaml'
    setup_logging(yaml_path)
    log()
    core.run()

```
这里我们定义了一个 setup_logging() 方法，里面读取了 yaml 文件的配置，然后通过 dictConfig() 方法将配置项传给了 logging 模块进行全局初始化。

另外这个模块还引入了另外一个模块 core，所以我们定义 core.py 如下：

```python
import logging
 
logger = logging.getLogger('main.core')
 
def run():
    logger.info('Core Info')
    logger.debug('Core Debug')
    logger.error('Core Error')
```
输出：

```powershell
$ python main.py 
2020-05-17 14:47:09,905 - Exec
2020-05-17 14:47:09,905 - Finished
2020-05-17 14:47:09,905 - Core Info
2020-05-17 14:47:09,905 - Core Info
2020-05-17 14:47:09,905 - Core Error
2020-05-17 14:47:09,905 - Core Error

$ cat  error.log 
2020-05-17 14:47:09,905 - main.core - ERROR - Core Error

$  cat debug.log 
2020-05-17 14:47:09,905 - main.core - INFO - Core Info
2020-05-17 14:47:09,905 - main.core - DEBUG - Core Debug
2020-05-17 14:47:09,905 - main.core - ERROR - Core Error
```
#### 3.6.2 ini配置文件
下面只展示了 ini 格式和 yaml 格式的配置。

```python
test.ini 文件
[loggers]
keys=root,sampleLogger

[handlers]
keys=consoleHandler

[formatters]
keys=sampleFormatter

[logger_root]
level=DEBUG
handlers=consoleHandler

[logger_sampleLogger]
level=DEBUG
handlers=consoleHandler
qualname=sampleLogger
propagate=0

[handler_consoleHandler]
class=StreamHandler
level=DEBUG
formatter=sampleFormatter
args=(sys.stdout,)

[formatter_sampleFormatter]
format=%(asctime)s - %(name)s - %(levelname)s - %(message)s
```


testinit.py 文件

```python
import logging.config

logging.config.fileConfig(fname='test.ini', disable_existing_loggers=False)
logger = logging.getLogger("sampleLogger")
logger.info('Core Info')
logger.debug('Core Debug')
logger.error('Core Error')
```
输出：

```python
$ python test.py 
2020-05-17 15:07:47,328 - sampleLogger - INFO - Core Info
2020-05-17 15:07:47,328 - sampleLogger - DEBUG - Core Debug
2020-05-17 15:07:47,328 - sampleLogger - ERROR - Core Error
```



### 3.7 字符串拼接
 format() 构造与占位符用%
```python
$ log8.py
import logging
 
logging.basicConfig(level=logging.DEBUG, format='%(asctime)s - %(name)s - %(levelname)s - %(message)s')
 
# bad
logging.debug('Hello {0}, {1}!'.format('World', 'Congratulations'))
# good
logging.debug('Hello %s, %s!', 'World', 'Congratulations')
```
运行结果如下：

```python
$ python log8.py 
2020-05-17 14:55:29,709 - root - DEBUG - Hello World, Congratulations!
2020-05-17 14:55:29,709 - root - DEBUG - Hello World, Congratulations!
```

### 3.8 异常处理

```python
$ cat log9.py
import logging
 
logging.basicConfig(level=logging.DEBUG, format='%(asctime)s - %(name)s - %(levelname)s - %(message)s')
 
try:
    result = 5 / 0
except Exception as e:
    # bad
    logging.error('Error: %s', e)
    # good
    logging.error('Error', exc_info=True)
    # good
    logging.exception('Error')
```
结果输出：

```python
$ python log9.py
2020-05-17 14:56:51,027 - root - ERROR - Error: integer division or modulo by zero
2020-05-17 14:56:51,027 - root - ERROR - Error
Traceback (most recent call last):
  File "log9.py", line 6, in <module>
    result = 5 / 0
ZeroDivisionError: integer division or modulo by zero
2020-05-17 14:56:51,028 - root - ERROR - Error
Traceback (most recent call last):
  File "log9.py", line 6, in <module>
    result = 5 / 0
ZeroDivisionError: integer division or modulo by zero
```

### 3.9 日志文件按照时间划分或者按照大小划分
如果将日志保存在一个文件中，那么时间一长，或者日志一多，单个日志文件就会很大，既不利于备份，也不利于查看。我们会想到能不能按照时间或者大小对日志文件进行划分呢？答案肯定是可以的，并且还很简单，logging 考虑到了我们这个需求。logging.handlers 文件中提供了 TimedRotatingFileHandler 和 RotatingFileHandler 类分别可以实现按时间和大小划分。打开这个 handles 文件，可以看到还有其他功能的 Handler 类，它们都继承自基类 BaseRotatingHandler。

#### 3.9.1 TimedRotatingFileHandler构造函数
定义如下:

```python
TimedRotatingFileHandler(filename [,when [,interval [,backupCount]]])
```
示例：

```python
# 每隔 1小时 划分一个日志文件，interval 是时间间隔，备份文件为 10 个
handler2 = logging.handlers.TimedRotatingFileHandler("test.log", when="H", interval=1, backupCount=10)
```

 - filename： 是输出日志文件名的前缀，比如log/myapp.log
 - when ：是一个字符串的定义如下：
 “S”: Seconds
“M”: Minutes
“H”: Hours
“D”: Days
“W”: Week day (0=Monday)
“midnight”: Roll over at midnight
 - interval ：是指等待多少个单位when的时间后，Logger会自动重建文件
 - backupCount：是保留日志个数。默认的0是不会自动删除掉日志。若设3，则在文件的创建过程中库会判断是否有超过这个3，若超过，则会从最先创建的开始删除。


```python
$ cat log10.py 
#!/usr/bin/python
#---coding:utf-8

import logging
import logging.handlers
import datetime,time

#logging    初始化工作
logger = logging.getLogger("zjlogger")
logger.setLevel(logging.DEBUG)

# 添加TimedRotatingFileHandler
# 定义一个1秒换一次log文件的handler
# 保留3个旧log文件
rf_handler = logging.handlers.TimedRotatingFileHandler(filename="all.log",when='S',interval=1,\
                                                       backupCount=3)
rf_handler.setFormatter(logging.Formatter("%(asctime)s - %(levelname)s - %(filename)s[:%(lineno)d] - %(message)s"))

#在控制台打印日志
handler = logging.StreamHandler()
handler.setLevel(logging.DEBUG)
handler.setFormatter(logging.Formatter("%(asctime)s - %(levelname)s - %(message)s"))

logger.addHandler(rf_handler)
logger.addHandler(handler)

while True:
    logger.debug('debug message')
    logger.info('info message')
    logger.warning('warning message')
    logger.error('error message')
    logger.critical('critical message')
    time.sleep(1)
```

```python
$ ls
all.log  all.log.2020-05-17_15-21-55  all.log.2020-05-17_15-21-56  all.log.2020-05-17_15-21-57  log10.py
```

#### 3.9.2 RotatingFileHandler 构造函数
示例：
```python
# 每隔 1000 Byte 划分一个日志文件，备份文件为 3 个
file_handler = logging.handlers.RotatingFileHandler("test.log", mode="w", maxBytes=1000, backupCount=3, encoding="utf-8")
复制代码

```

```python
$ cat log11.py 
#!coding:utf-8
#!/usr/bin/env python
import time
import logging
import logging.handlers
# logging初始化工作
logging.basicConfig()
# myapp的初始化工作
myapp = logging.getLogger('myapp')
myapp.setLevel(logging.INFO)
# 写入文件，如果文件超过100个Bytes，仅保留5个文件。
handler = logging.handlers.RotatingFileHandler(
              'logs/myapp.log', maxBytes=100, backupCount=5)
myapp.addHandler(handler)
while True:
    time.sleep(0.01)
    myapp.info("file test")
```
输出：

```python
$ python log11.py
INFO:myapp:file test
INFO:myapp:file test
INFO:myapp:file test
INFO:myapp:file test
.....
$ ls logs
myapp.log  myapp.log.1  myapp.log.2  myapp.log.3  myapp.log.4  myapp.log.5
```
### 3.10 logging.disable暂时禁止日志输出
运行测试代码时，我不想登录到文件或控制台
在运行测试代码时将日志输出到文件或控制台通常会感到非常烦人。
如果将其输出到文件，则可以将其保留，但是如果将其记录到控制台，将很难理解测试执行的进度和结果。
而且，由于日志本身就是输入/输出，因此花费额外的测试执行时间没有错
 `logging.disable`会忽略低于指定日志级别的日志输出
Python的日志记录模块定义了一个名为disable的函数，该函数将日志级别作为参数传递。
例如

```bash
import logging
logging.disable(logging.FATAL)
```

如果这样做，将不会输出`FATAL`以下的日志。
换句话说，由于它低于最高级别FATAL，因此实际上没有日志输出。
反之亦然

```bash
import loggging
logging.disable(logging.NOTSET)
```

`NOTSET`是最低的日志级别，因此这与取消抑制日志相同。

我使用setUp和tearDown使用logging.disable
考虑到这一点，您可以执行以下操作来在编写测试代码时暂时停止日志输出。

```bash
import logging
import unittest

class FooTest(unittest.TestCase):
    def setUp(self):
        logging.disable(logging.FATAL)

    def tearDown(self):
        logging.disable(logging.NOTSET)

    def test(self):
```


参考:

 - [进击的Coder](https://cuiqingcai.com/6080.html)
 - [vigny的先生](https://www.jianshu.com/p/5a4e226444bd)
 - [好吃的野菜](https://www.jianshu.com/p/feb86c06c4f4)
 - [Python logging.disable() Examples](https://www.programcreek.com/python/example/5472/logging.disable)

