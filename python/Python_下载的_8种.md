

----
## 1. 使用requests
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201119110050412.png#pic_center)
下载重定向的文件
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201119110427877.png#pic_center)

## 2. 使用wget
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201119110316136.png#pic_center)

## 3. 下载多个文件(并行/批量下载)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201119110611377.png#pic_center)
我们导入了os和time模块来检查下载文件需要多少时间。ThreadPool模块允许你使用池运行多个线程或进程。
让我们创建一个简单的函数，将响应分块发送到一个文件:
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201119110634215.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70#pic_center)
这个URL是一个二维数组，它指定了你要下载的页面的路径和URL。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201119110705509.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70#pic_center)
就像在前一节中所做的那样，我们将这个URL传递给requests.get。最后，我们打开文件(URL中指定的路径)并写入页面内容。
现在，我们可以分别为每个URL调用这个函数，我们也可以同时为所有URL调用这个函数。让我们在for循环中分别为每个URL调用这个函数，注意计时器:
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201119110727730.png#pic_center)
现在，使用以下代码行替换for循环：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201119110743798.png#pic_center)
## 4. 使用进度条进行下载
![**在这里插入图片描述**](https://img-blog.csdnimg.cn/20201119110826373.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70#pic_center)
## 5. 使用urllib下载网页
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201119111005506.png#pic_center)
## 6. 通过代理下载
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201119111022834.png#pic_center)
在这段代码中，我们创建了代理对象，并通过调用urllib的build_opener方法来打开该代理，并传入该代理对象。然后，我们创建请求来获取页面。
此外，你还可以按照官方文档的介绍来使用requests模块:
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201119111041330.png#pic_center)
## 7. 使用urllib3
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201119111143408.png#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201119111148432.png#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020111911120036.png#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020111911121342.png#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201119111219270.png#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201119111224633.png#pic_center)
## 8. 使用asyncio
asyncio模块主要用于处理系统事件。它围绕一个事件循环进行工作，该事件循环会等待事件发生，然后对该事件作出反应。这个反应可以是调用另一个函数。这个过程称为事件处理。asyncio模块使用协同程序进行事件处理。
要使用asyncio事件处理和协同功能，我们将导入asyncio模块:
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201119111437602.png#pic_center)
现在，像这样定义asyncio协同方法
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201119111449498.png#pic_center)
关键字async表示这是一个原生asyncio协同程序。在协同程序的内部，我们有一个await关键字，它会返回一个特定的值。我们也可以使用return关键字。
现在，让我们使用协同创建一段代码来从网站下载一个文件:
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201119111506893.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70#pic_center)
在这段代码中，我们创建了一个异步协同函数，它会下载我们的文件并返回一条消息。
然后，我们使用另一个异步协同程序调用main_func，它会等待URL并将所有URL组成一个队列。asyncio的wait函数会等待协同程序完成。
现在，为了启动协同程序，我们必须使用asyncio的get_event_loop()方法将协同程序放入事件循环中，最后，我们使用asyncio的run_until_complete()方法执行该事件循环。
