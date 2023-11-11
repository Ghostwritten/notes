#  python 模块 Selenium 自动化测试 Web 工具



---
## 1. Selenium简介
Selenium是一个用于测试网站的自动化测试工具，支持各种浏览器包括Chrome、Firefox、Safari等主流界面浏览器，同时也支持phantomJS无界面浏览器。可执行系统如Windows、Linux、IOS、Android等。

相关网站：
 - [github](https://github.com/SeleniumHQ/selenium/tree/trunk/py) 
 - [pypi](https://pypi.org/project/selenium/)    
 - [帮助文档](https://selenium-python.readthedocs.io/)  
 -  [官网](https://www.selenium.dev/)

![在这里插入图片描述](https://img-blog.csdnimg.cn/9f3a2f5b99514d2489d57af034f8b2a7.gif#pic_center)

##  2. 安装Selenium

```bash
pip install Selenium
```

## 3. 下载浏览器驱动

 - Firefox浏览器驱动：[geckodriver](https://github.com/mozilla/geckodriver/releases)
 - Chrome浏览器驱动：[chromedriver](https://sites.google.com/a/chromium.org/chromedriver/home) ,or [chromedriver 2](http://chromedriver.storage.googleapis.com/index.html)
 - IE浏览器驱动：[IEDriverServer](http://selenium-release.storage.googleapis.com/index.html)
 - Edge浏览器驱动：[MicrosoftWebDriver](https://developer.microsoft.com/en-us/microsoft-edge/tools/webdriver/)
 - Opera浏览器驱动：[operadriver](https://github.com/operasoftware/operachromiumdriver/releases)
 - PhantomJS浏览器驱动：[phantomjs](https://phantomjs.org/)

需要把浏览器驱动放入系统路径中，或者直接告知selenuim的驱动路径
##  4. 环境变量
设置浏览器的地址非常简单。 我们可以手动创建一个存放浏览器驱动的目录，如： F:\GeckoDriver , 将下载的浏览器驱动文件（例如：chromedriver、geckodriver）丢到该目录下。

我的电脑–>属性–>系统设置–>高级–>环境变量–>系统变量–>Path，将“F:\GeckoDriver”目录添加到Path的值中。比如：Path字段;F:\GeckoDriver


##  5. 参数

另外启动浏览器，可以设置一些参数，比如无界面之类的，[详细参考](https://zhuanlan.zhihu.com/p/60852696)

可以测试是否正常使用，以下代码：

```bash
from selenium import webdriver

driver = webdriver.Firefox()   # Firefox浏览器
# driver = webdriver.Firefox("驱动路径")

driver = webdriver.Chrome()    # Chrome浏览器

driver = webdriver.Ie()        # Internet Explorer浏览器

driver = webdriver.Edge()      # Edge浏览器

driver = webdriver.Opera()     # Opera浏览器

driver = webdriver.PhantomJS()   # PhantomJS

# 打开网页
driver.get(url) # 打开url网页 比如 driver.get("http://www.baidu.com")
```

##  6. 定位元素的8种方式



|定位一个元素	| 定位多个元素 |	含义 |
|--|--|--|
| find_element_by_id |	find_elements_by_id |	通过元素id定位 |
|find_element_by_name |	find_elements_by_name |	通过元素name定位 | 
find_element_by_xpath |	find_elements_by_xpath	| 通过xpath表达式定位 | 
|find_element_by_link_text	| find_elements_by_link_tex |	通过完整超链接定位 |
|find_element_by_partial_link_text |	find_elements_by_partial_link_text |	通过部分链接定位 | 
| find_element_by_tag_name |	find_elements_by_tag_name	| 通过标签定位 |
| find_element_by_class_name |	find_elements_by_class_name |	通过类名进行定位 |
| find_elements_by_css_selector |	find_elements_by_css_selector |	通过css选择器进行定位 |


demo：
假如我们有一个Web页面，通过前端工具（如，Firebug）查看到一个元素的属性是这样的。

```bash
<html>
  <head>
  <body link="#0000cc">
    <a id="result_logo" href="/" onmousedown="return c({'fm':'tab','tab':'logo'})">
    <form id="form" class="fm" name="f" action="/s">
      <span class="soutu-btn"></span>
        <input id="kw" class="s_ipt" name="wd" value="" maxlength="255" autocomplete="off">
```

通过id定位:

```bash
dr.find_element_by_id("kw")
```

通过name定位:

```bash
dr.find_element_by_name("wd")
```

通过class name定位:

```bash
dr.find_element_by_class_name("s_ipt")
```

通过tag name定位:

```bash
dr.find_element_by_tag_name("input")
```

通过`xpath`定位，`xpath`定位有N种写法，这里列几个常用写法:

```bash
dr.find_element_by_xpath("//*[@id='kw']")
dr.find_element_by_xpath("//*[@name='wd']")
dr.find_element_by_xpath("//input[@class='s_ipt']")
dr.find_element_by_xpath("/html/body/form/span/input")
dr.find_element_by_xpath("//span[@class='soutu-btn']/input")
dr.find_element_by_xpath("//form[@id='form']/span/input")
dr.find_element_by_xpath("//input[@id='kw' and @name='wd']")
```
通过css定位，css定位有N种写法，这里列几个常用写法:

```bash
dr.find_element_by_css_selector("#kw")
dr.find_element_by_css_selector("[name=wd]")
dr.find_element_by_css_selector(".s_ipt")
dr.find_element_by_css_selector("html > body > form > span > input")
dr.find_element_by_css_selector("span.soutu-btn> input#kw")
dr.find_element_by_css_selector("form#form > span > input")
```
接下来，我们的页面上有一组文本链接。

```bash
dr.find_element_by_link_text("新闻")
dr.find_element_by_link_text("hao123")
```
通过link text定位:

```bash
dr.find_element_by_link_text("新闻")
dr.find_element_by_link_text("hao123")
```
通过partial link text定位:

```bash
dr.find_element_by_partial_link_text("新")
dr.find_element_by_partial_link_text("hao")
dr.find_element_by_partial_link_text("123")
```


## 7.  控制浏览器操作方法
| 方法 |	说明 |
|--|--|
| set_window_size() |	设置浏览器的大小|
|back()	| 控制浏览器后退 | 
| forward() |	控制浏览器前进 |
| refresh() |	刷新当前页面 |
| clear() |	清除文本 |
| send_keys (value)	| 模拟按键输入 |
| click()	 | 单击元素 |
| submit() |	用于提交表单 |
| get_attribute(name)	| 获取元素属性值 |
| is_displayed() | 设置该元素是否用户可见 |
| size |	返回元素的尺寸 |
| text |	获取元素的文本 |

demo:

```bash
from selenium import webdriver

from time import sleep
#1.创建Chrome浏览器对象，这会在电脑上在打开一个浏览器窗口
browser = webdriver.Firefox(executable_path ="F:\GeckoDriver\geckodriver")

#2.通过浏览器向服务器发送URL请求
browser.get("https://www.baidu.com/")

sleep(3)

#3.刷新浏览器
browser.refresh()

#4.设置浏览器的大小
browser.set_window_size(1400,800)

#5.设置链接内容
element=browser.find_element_by_link_text("新闻")
element.click()

element=browser.find_element_by_link_text("“下团组”时间")
element.click()
```

##  8. 键盘事件
`Selenium`中的Key模块为我们提供了模拟键盘按键的方法，那就是`send_keys()`方法。它不仅可以模拟键盘输入，也可以模拟键盘的操作。

常用的键盘操作如下：
| 模拟键盘按键	| 说明 |
|--|--|
| send_keys(Keys.BACK_SPACE)	| 删除键（BackSpace）|
| send_keys(Keys.SPACE)	| 空格键(Space) |
| send_keys(Keys.TAB)	| 制表键(Tab) |
| send_keys(Keys.ESCAPE)	| 回退键（Esc）|
| send_keys(Keys.ENTER)	| 回车键（Enter）|

组合键的使用
| 模拟键盘按键	| 说明 |
|--|--|
| send_keys(Keys.CONTROL,‘a’)	| 全选（Ctrl+A） |
|send_keys(Keys.CONTROL,‘c’)	| 复制（Ctrl+C）| 
| send_keys(Keys.CONTROL,‘x’)	| 剪切（Ctrl+X）|
| send_keys(Keys.CONTROL,‘v’)	| 粘贴（Ctrl+V）
| send_keys(Keys.F1…Fn)	| 键盘 F1…Fn |

## 9. 获取断言信息
不管是在做功能测试还是自动化测试，最后一步需要拿实际结果与预期进行比较。这个比较的称之为断言。通过我们获取title 、URL和text等信息进行断言。

| 属性 |	说明 |
|--|--|
| title |	 用于获得当前页面的标题 |
| current_url |	用户获得当前页面的URL |
| text | 获取搜索条目的文本信息 |

demo：

```bash
from selenium import webdriver
from time import sleep

driver = webdriver.Firefox(executable_path ="F:\GeckoDriver\geckodriver")
driver.get("https://www.baidu.com")

print('Before search================')

# 打印当前页面title
title = driver.title
print(title)

# 打印当前页面URL
now_url = driver.current_url
print(now_url)

driver.find_element_by_id("kw").send_keys("selenium")
driver.find_element_by_id("su").click()
sleep(1)

print('After search================')

# 再次打印当前页面title
title = driver.title
print(title)

# 打印当前页面URL
now_url = driver.current_url
print(now_url)

# 获取结果数目
user = driver.find_element_by_class_name('nums').text
print(user)

#关闭所有窗口
driver.quit()
```
打印输出结果

```bash
Before search================
百度一下，你就知道
https://www.baidu.com/
After search================
selenium_百度搜索
https://www.baidu.com/s?ie=utf-8&f=8&rsv_bp=0&rsv_idx=1&tn=baidu&wd=selenium&rsv_pq=a1d51b980000e36e&rsv_t=a715IZaMpLd1w92I4LNUi7gKuOdlAz5McsHe%2FSLQeBZD44OUIPnjY%2B7pODM&rqlang=cn&rsv_enter=0&rsv_sug3=8&inputT=758&rsv_sug4=759
搜索工具
百度为您找到相关结果约7,170,000个
————————————————
```

##  10. 浏览器操作

```bash
browser.forward() # 向前
browser.back() # 返回
browser.fullscreen_window() # 全屏
browser.maximize_window() # 最大化
browser.minimize_window() # 最小化
```

##  11. 设置元素等待
###  11.1 显示等待
显式等待使WebdDriver等待某个条件成立时继续执行，否则在达到最大时长时抛出超时异常（TimeoutException）。

```bash
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC

driver = webdriver.Firefox()
driver.get("http://www.baidu.com")

element = WebDriverWait(driver, 5, 0.5).until(
                      EC.presence_of_element_located((By.ID, "kw"))
                      )
element.send_keys('selenium')
driver.quit()
```
`WebDriverWait`类是由WebDirver 提供的等待方法。在设置时间内，默认每隔一段时间检测一次当前页面元素是否存在，如果超过设置时间检测不到则抛出异常。具体格式如下：

```bash
WebDriverWait(driver, timeout, poll_frequency=0.5, ignored_exceptions=None)
```

 - `driver` ：浏览器驱动。
 - `timeout` ：最长超时时间，默认以秒为单位。
 - `poll_frequency` ：检测的间隔（步长）时间，默认为0.5S。
 - `ignored_exceptions` ：超时后的异常信息，默认情况下抛NoSuchElementException异常。
 - `WebDriverWait()`一般由until()或until_not()方法配合使用，下面是until()和until_not()方法的说明。
 - `until(method, message=‘’)` 调用该方法提供的驱动程序作为一个参数，直到返回值为True。
 - `until_not(method, message=‘’)` 调用该方法提供的驱动程序作为一个参数，直到返回值为False。

在本例中，通过as关键字将`expected_conditions` 重命名为EC，并调用presence_of_element_located()方法判断元素是否存在。

### 11.2 隐式等待
如果某些元素不是立即可用的，隐式等待是告诉WebDriver去等待一定的时间后去查找元素。 默认等待时间是0秒，一旦设置该值，隐式等待是设置该WebDriver的实例的生命周期。

```bash
from selenium import webdriver
driver = webdriver.Firefox()    
driver.implicitly_wait(10) # seconds    
driver.get("http://somedomain/url_that_delays_loading")    
myDynamicElement = driver.find_element_by_id("myDynamicElement") 
```

##  12. 多窗口切换
在页面操作过程中有时候点击某个链接会弹出新的窗口，这时就需要主机切换到新打开的窗口上进行操作。WebDriver提供了switch_to.window()方法，可以实现在不同的窗口之间切换。

|方法	|说明|
|--|--|
|current_window_handle	|获得当前窗口句柄|
|window_handles	| 返回所有窗口的句柄到当前会话
|switch_to.window()	 |用于切换到相应的窗口，与上一节的switch_to.frame()类似，前者用于不同窗口的切换，后者用于不同表单之间的切换。|


```bash
driver.switch_to_window("windowName")
driver.switch_to_frame("frameName")
```
以直接取表单的id 或name属性。如果iframe没有可用的id和name属性，则可以通过下面的方式进行定位。

```bash
#先通过xpth定位到iframe
xf = driver.find_element_by_xpath('//*[@id="x-URS-iframe"]')

#再将定位对象传给switch_to_frame()方法
driver.switch_to_frame(xf)
```
一旦我们完成了frame中的工作，我们可以这样返回父frame:

```bash
driver.switch_to_default_content()
```

demo：

```bash
from selenium import webdriver
import time
driver = webdriver.Chrome("F:\Chrome\ChromeDriver\chromedriver")
driver.implicitly_wait(10)
driver.get("http://www.baidu.com")

#1.获得百度搜索窗口句柄
sreach_windows = driver.current_window_handle

driver.find_element_by_link_text('登录').click()
driver.find_element_by_link_text("立即注册").click()

#1.获得当前所有打开的窗口的句柄
all_handles = driver.window_handles

#3.进入注册窗口
for handle in all_handles:
    if handle != sreach_windows:
        driver.switch_to.window(handle)
        print('跳转到注册窗口')
        driver.find_element_by_name("account").send_keys('123456789')
        driver.find_element_by_name('password').send_keys('123456789')
        time.sleep(2)
    
driver.quit()
```

## 13. 警告框处理

在WebDriver中处理JavaScript所生成的alert、confirm以及prompt十分简单，具体做法是使用 switch_to.alert 方法定位到 alert/confirm/prompt，然后使用text/accept/dismiss/ send_keys等方法进行操作。


|方法	|说明 |
|--|--|
|text	|返回 alert/confirm/prompt 中的文字信息|
|accept()	|接受现有警告框|
|dismiss()	| 解散现有警告框|
|send_keys(keysToSend)	|发送文本至警告框。keysToSend：将文本发送至警告框。|

demo：


```bash
from selenium import webdriver
from selenium.webdriver.common.action_chains import ActionChains
import time

driver = webdriver.Chrome("F:\Chrome\ChromeDriver\chromedriver")
driver.implicitly_wait(10)
driver.get('http://www.baidu.com')

# 鼠标悬停至“设置”链接
link = driver.find_element_by_link_text('设置')
ActionChains(driver).move_to_element(link).perform()

# 打开搜索设置
driver.find_element_by_link_text("搜索设置").click()

#在此处设置等待2s否则可能报错
time.sleep(2)
# 保存设置
driver.find_element_by_class_name("prefpanelgo").click()
time.sleep(2)

# 接受警告框
driver.switch_to.alert.accept()

driver.quit()
```



##  14. 定位一组元素
定位一组元素的方法与定位单个元素的方法类似，唯一的区别是在单词element后面多了一个s表示复数。

```bash
from selenium import webdriver
from time import sleep

driver =webdriver.Firefox(executable_path ="F:\GeckoDriver\geckodriver")
driver.get("https://www.baidu.com")

driver.find_element_by_id("kw").send_keys("selenium")
driver.find_element_by_id("su").click()
sleep(1)

#1.定位一组元素
elements = driver.find_elements_by_xpath('//div/h3/a')
print(type(elements))

#2.循环遍历出每一条搜索结果的标题
for t in elements:
    print(t.text)
    element=driver.find_element_by_link_text(t.text)
    element.click()
    sleep(3)

driver.quit()
```

##  15. 多表单切换
在Web应用中经常会遇到`frame/iframe`表单嵌套页面的应用，WebDriver只能在一个页面上对元素识别与定位，对于`frame/iframe`表单内嵌页面上的元素无法直接定位。这时就需要通过`switch_to.frame()`方法将当前定位的主体切换为`frame/iframe`表单的内嵌页面中。

|方法	| 说明 |
|--|--|
|switch_to.frame()	|将当前定位的主体切换为frame/iframe表单的内嵌页面中 |
|switch_to.default_content()	| 跳回最外层的页面 |

```bash
<html>
  <body>
    ...
    <iframe id="x-URS-iframe" ...>
      <html>
         <body>
           ...
           <input name="email" >
```
126邮箱登录框的结构大概是这样子的，想要操作登录框必须要先切换到iframe表单。

```bash
from selenium import webdriver

driver = webdriver.Chrome()
driver.get("http://www.126.com")

driver.switch_to.frame('x-URS-iframe')
driver.find_element_by_name("email").clear()
driver.find_element_by_name("email").send_keys("username")
driver.find_element_by_name("password").clear()
driver.find_element_by_name("password").send_keys("password")
driver.find_element_by_id("dologin").click()
driver.switch_to.default_content()

driver.quit()
```
`switch_to.frame()` 默认可以直接取表单的id 或name属性。如果iframe没有可用的id和name属性，则可以通过下面的方式进行定位。

```bash
……
#先通过xpth定位到iframe
xf = driver.find_element_by_xpath('//*[@id="x-URS-iframe"]')

#再将定位对象传给switch_to.frame()方法
driver.switch_to.frame(xf)
……
driver.switch_to.parent_frame()
```
##  16. 下拉框选择操作
导入选择下拉框Select类，使用该类处理下拉框操作。

```bash
from selenium.webdriver.support.select import Select
```
Select类的方法
|方法	|说明 |
|--|--|
|select_by_value(“选择值”)	| select标签的value属性的值 |
|select_by_index(“索引值”)	| 下拉框的索引 |
|select_by_visible_testx(“文本值”)	| 下拉框的文本值 |


有时我们会碰到下拉框，WebDriver提供了Select类来处理下拉框。

```bash
from selenium import webdriver
from selenium.webdriver.support.select import Select
from time import sleep

driver = webdriver.Chrome("F:\Chrome\ChromeDriver\chromedriver")
driver.implicitly_wait(10)
driver.get('http://www.baidu.com')

#1.鼠标悬停至“设置”链接
driver.find_element_by_link_text('设置').click()
sleep(1)
#2.打开搜索设置
driver.find_element_by_link_text("搜索设置").click()
sleep(2)

#3.搜索结果显示条数
sel = driver.find_element_by_xpath("//select[@id='nr']")
Select(sel).select_by_value('50')  # 显示50条

sleep(3)
driver.quit()
```

##  17. 文件上传
对于通过input标签实现的上传功能，可以将其看作是一个输入框，即通过`send_keys()`指定本地文件路径的方式实现文件上传。

通过`send_keys()`方法来实现文件上传:

```bash
from selenium import webdriver
import os

driver = webdriver.Firefox()
file_path = 'file:///' + os.path.abspath('upfile.html')
driver.get(file_path)

# 定位上传按钮，添加本地文件
driver.find_element_by_name("file").send_keys('D:\\upload_file.txt')

driver.quit()
```

##  18. cookie操作
有时候我们需要验证浏览器中cookie是否正确，因为基于真实cookie的测试是无法通过白盒和集成测试进行的。WebDriver提供了操作Cookie的相关方法，可以读取、添加和删除cookie信息。

`WebDriver`操作cookie的方法:

|方法	| 说明 |
|--|--|
|get_cookies() |	获得所有cookie信息 |
|get_cookie(name)	| 返回字典的key为“name”的cookie信息 |
|add_cookie(cookie_dict)	| 添加cookie。“cookie_dict”指字典对象，必须有name 和value 值 |
|delete_cookie(name,optionsString)	| 删除cookie信息。“name”是要删除的cookie的名称，“optionsString”是该cookie的选项，目前支持的选项包括“路径”，“域” |
|delete_all_cookies()	| 删除所有cookie信息|

demo:

```bash
from selenium import webdriver
import time
browser = webdriver.Chrome("F:\Chrome\ChromeDriver\chromedriver")
browser.get("http://www.youdao.com")

#1.打印cookie信息
print('=====================================')
print("打印cookie信息为：")
print(browser.get_cookies)

#2.添加cookie信息
dict={'name':"name",'value':'Kaina'}
browser.add_cookie(dict)

print('=====================================')
print('添加cookie信息为：')
#3.遍历打印cookie信息
for cookie in browser.get_cookies():
    print('%s----%s\n' %(cookie['name'],cookie['value']))
    
#4.删除一个cookie
browser.delete_cookie('name')
print('=====================================')
print('删除一个cookie')
for cookie in browser.get_cookies():
    print('%s----%s\n' %(cookie['name'],cookie['value']))

print('=====================================')
print('删除所有cookie后：')
#5.删除所有cookie,无需传递参数
browser.delete_all_cookies()
for cookie in browser.get_cookies():
    print('%s----%s\n' %(cookie['name'],cookie['value']))

time.sleep(3)
browser.close()
```

##  19. 调用JavaScript代码
虽然WebDriver提供了操作浏览器的前进和后退方法，但对于浏览器滚动条并没有提供相应的操作方法。在这种情况下，就可以借助JavaScript来控制浏览器的滚动条。WebDriver提供了`execute_script()`方法来执行`JavaScript`代码。

用于调整浏览器滚动条位置的JavaScript代码如下：

```bash
<!-- window.scrollTo(左边距,上边距); -->
window.scrollTo(0,450);
```
`window.scrollTo()`方法用于设置浏览器窗口滚动条的水平和垂直位置。方法的第一个参数表示水平的左间距，第二个参数表示垂直的上边距。其代码如下：

```bash
from selenium import webdriver
from time import sleep

#1.访问百度
driver=webdriver.Firefox(executable_path ="F:\GeckoDriver\geckodriver")
driver.get("http://www.baidu.com")

#2.搜索
driver.find_element_by_id("kw").send_keys("selenium")
driver.find_element_by_id("su").click()

#3.休眠2s目的是获得服务器的响应内容，如果不使用休眠可能报错
sleep(2)

#4.通过javascript设置浏览器窗口的滚动条位置
js="window.scrollTo(100,450);"
driver.execute_script(js)
sleep(3)

driver.close()
```
通过浏览器打开百度进行搜索，并且提前通过`set_window_size()`方法将浏览器窗口设置为固定宽高显示，目的是让窗口出现水平和垂直滚动条。然后通过`execute_script()`方法执行JavaScripts代码来移动滚动条的位置。
滚动条上下左右滚动代码演示

```bash
from selenium import webdriver
from time import sleep

driver=webdriver.Firefox(executable_path ="F:\GeckoDriver\geckodriver")
driver.set_window_size(400,400)
driver.get("https://www.baidu.com")

#2.搜索
# driver.find_element_by_id("kw").send_keys("selenium")
# driver.find_element_by_id("su").click()

#3.休眠2s目的是获得服务器的响应内容，如果不使用休眠可能报错
sleep(10)


#4 滚动左右滚动条---向右
js2 = "var q=document.documentElement.scrollLeft=10000"
driver.execute_script(js2)
sleep(15)

#5 滚动左右滚动条---向左
js3 = "var q=document.documentElement.scrollLeft=0"
driver.execute_script(js3)
sleep(15)

#6 拖动到滚动条底部---向下
js = "var q=document.documentElement.scrollTop=10000"
driver.execute_script(js)
sleep(15)

#7 拖动到滚动条底部---向上
js = "var q=document.documentElement.scrollTop=0"
driver.execute_script(js)
sleep(15)

driver.close()
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/734cb514bb5f46178b8ffd76330eec7c.gif#pic_center)

##  20. 窗口截图
自动化用例是由程序去执行的，因此有时候打印的错误信息并不十分明确。如果在脚本执行出错的时候能对当前窗口截图保存，那么通过图片就可以非常直观地看出出错的原因。WebDriver提供了截图函数`get_screenshot_as_file()`来截取当前窗口。

截屏方法：
|方法	| 说明 |
|--|--|
| get_screenshot_as_file(self, filename) | 用于截取当前窗口，并把图片保存到本地 |

```bash
from selenium import webdriver
from time import sleep

driver =webdriver.Firefox(executable_path ="F:\GeckoDriver\geckodriver")
driver.get('http://www.baidu.com')

driver.find_element_by_id('kw').send_keys('selenium')
driver.find_element_by_id('su').click()
sleep(2)

#1.截取当前窗口，并指定截图图片的保存位置
driver.get_screenshot_as_file("D:\\baidu_img.jpg")

driver.quit()
```

##  21. 关闭浏览器
在前面的例子中我们一直使用quit()方法，其含义为退出相关的驱动程序和关闭所有窗口。除此之外，WebDriver还提供了close()方法，用来关闭当前窗口。例多窗口的处理，在用例执行的过程中打开了多个窗口，我们想要关闭其中的某个窗口，这时就要用到close()方法进行关闭了。

|方法 |	说明|
|--|--|
|close() |	关闭单个窗口|
| quit()	| 关闭所有窗口|

##  22. 封装
将各种方式获取元素的进行封装，以类似 `get_by_browser('class:class_name')` 进行调用：

```bash
def get_by_browser(browser, by):
    by_kv = by.split(':')
    k, v = by_kv[0], by_kv[1]

    by_case = {
        "id": lambda x: browser.find_element_by_id(x),
        "name": lambda x: browser.find_element_by_name(x),
        "xpath": lambda x: browser.find_element_by_xpath(x),
        "link_text": lambda x: browser.find_element_by_link_text(x),
        "partial_link_text": lambda x: browser.find_element_by_partial_link_text(x),
        "tag_name": lambda x: browser.find_element_by_tag_name(x),
        "class_name": lambda x: browser.find_element_by_class_name(x),
        "css_selector": lambda x: browser.find_element_by_css_selector(x)
    }

    try:
        browser = by_case[k](v)
    except KeyError as e:
        browser = browser.find_element_by_tag_name('html')

    return browser
```
以上边的功能也可用以下方式实现：

```bash
from selenium.webdriver.common.by import By
# browser.find_element(by='id', value=None)
browser.find_element(By.ID, "content")
```


##  23. selenium 和 requests 结合
网页渲染后，单纯的 HTTP 请求可以使用 Requests 库来操作。还有种情况中，登录有复杂的验证码如拖动图像块、手机验证码等，需要 selenium 配合并人工完成，登录之后再用 Requests 访问数据。

```bash
import requests
s = requests.Session()
selenium_user_agent = browser.execute_script("return navigator.userAgent;")
s.headers.update({"user-agent": selenium_user_agent})
for cookie in browser.get_cookies():
    s.cookies.set(cookie['name'], cookie['value'], domain=cookie['domain'])

# 发起访问请求
r = s.get(target_url)
# browser.delete_all_cookies()
```
另外，上述登录验证码问题：

 - 滑动验证，可以 Selenium 模拟
 - 滑动距离，图像梯度算法可判断
 - 图文验证，可以 Python AI 库识别


##  24. 禁止加载图片和JS
禁止加载图片和JS，可使用以下方法：

```bash
chrome_options = webdriver.ChromeOptions()
prefs={
'profile.default_content_setting_values': {
'images':2,
'javascript':2
}
}

chrome_options.add_experimental_option("prefs", prefs)

driver.switch_to.frame('ptlogin_iframe') # 切换框架页
driver.implicitly_wait(10) # 自动检测，如果10秒未成功则报错
# 通过文字定位元素
b.find_element_by_xpath("//[text()='点我']")
b.find_element_by_xpath("//*[contains(text(), '我')]")
```

##  25. 解压 gzip
想解压通过 gzip 压缩的请求返回数据可通过以下方法：

```bash
# 方法二，推荐
import gzip
# r 为 gzip 压缩的 bytes
gzip.decompress(r)
# ...

# 方法二
import zlib
# r 为 gzip 压缩的 bytes
zlib.decompress(r, 16+zlib.MAX_WBITS)
# ...
```
##  26. 监听请求
如果要监听浏览器的请求，可使用对 Selenium 三方封装 `Selenium Wire`：

```bash
# pip install selenium-wire
from seleniumwire import webdriver  # Import from seleniumwire

# Create a new instance of the Chrome driver
driver = webdriver.Chrome()

# Go to the Google home page
driver.get('https://www.google.com')

# Access requests via the `requests` attribute
for request in driver.requests:
    if request.response:
        print(
            request.url,
            request.response.status_code,
            request.response.headers['Content-Type']
        )
'''
https://www.google.com/ 200 text/html; charset=...
https://www.google.com/images/branding/googlelo...
https://consent.google.com/status?continue=http...
https://www.google.com/images/branding/googlelo...
https://ssl.gstatic.com/gb/images/i2_2ec824b0.p...
https://www.google.com/gen_204?s=webaft&t=aft&a...
...
'''
```
----

✈<font color=	#FF4500 size=4 style="font-family:Courier New">参考阅读：</font>

 - [https://www.gairuo.com/p/python-selenium](https://www.gairuo.com/p/python-selenium)
 - [https://www.selenium.dev/](https://www.selenium.dev/)
 - [https://selenium-python.readthedocs.io/](https://selenium-python.readthedocs.io/)
 - [https://blog.csdn.net/weixin_36279318/article/details/79475388](https://blog.csdn.net/weixin_36279318/article/details/79475388)
 - [https://zhuanlan.zhihu.com/p/111859925](https://zhuanlan.zhihu.com/p/111859925)
 - [http://www.selenium.org.cn/1694.html](http://www.selenium.org.cn/1694.html)

![在这里插入图片描述](https://img-blog.csdnimg.cn/5bd131ce2e874e11868944b6bc87277c.gif#pic_center)


