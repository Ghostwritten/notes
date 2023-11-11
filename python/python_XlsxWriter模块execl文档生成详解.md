#  python 模块 XlsxWriter 生成execl文档



----


## 1. 介绍
Excel是当今最流行的电子表格处理软件，支持丰富的计算函数及图表，在系统运营方面广泛用于运营数据报表，比如业务质量、资源利用、安全扫描等报表，同时也是应用系统常见的文件导出格式，以便数据使用人员做进一步加工处理。本节主要讲述利用Python操作Excel的模块XlsxWriter（https://xlsxwriter.readthedocs.org），可以操作多个工作表的文字、数字、公式、图表等。[XlsxWriter](https://pypi.org/project/XlsxWriter/) 模块具有以下功能：

 - 100%兼容的Excel XLSX文件，支持Excel 2003、Excel 2007等版本；
 - 支持所有Excel单元格数据格式；
 - 单元格合并、批注、自动筛选、丰富多格式字符串等；
 - 支持工作表PNG、JPEG图像，自定义图表；
 - 内存优化模式支持写入大文件。

## 2. 安装
第一种：

```bash
 pip install XlsxWriter    #pip安装方法
```

 第二种：

```bash
 easy_install XlsxWriter    #easy_install安装方法 
```

第三种：#源码安装方法 

```bash
curl -O -L http：//github.com/jmcnamara/XlsxWriter/archive/master.tar.gz 
tar zxvf master.tar.gz 
cd XlsxWriter-master/ 
sudo python setup.py install
```
示例：

```python
#!/usr/bin/python
#---coding:-utf-8---

import xlsxwriter

workbook = xlsxwriter.Workbook('demo1.xlsx')    #创建一个Excel文件 worksheet =
worksheet = workbook.add_worksheet()    #创建一个工作表对象
worksheet.set_column('A:A', 20)    #设定第一列（A）宽度为20像素
bold= workbook.add_format({'bold': True})    #定义一个加粗的格式对象
worksheet.write('A1', 'Hello')    #A1单元格写入’Hello’
worksheet.write('A2', 'World', bold)  #A2单元格写入’World’并引用加粗格式对象bold
worksheet.write('B2', u'中文测试', bold) #B2单元格写入中文并引用加粗格式对象bold
worksheet.write(2, 0, 32)    #用行列表示法写入数字’32’与’35.5′
worksheet.write(3, 0, 35.5) #行列表示法的单元格下标以0作为起始值，’3，0’等价于’A3’
worksheet.write(4, 0, '=SUM（A3：A4')    #求A3：A4的和，并将结果写入’4，0’，即’A5’
worksheet.insert_image('B5', 'img/1.jpg')    #在B5单元格插入图片
workbook.close()    #关闭Excel文件
```
输出:
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200419164550935.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
## 3. 方法
### 3.1 Workbook类
Workbook类定义：
`Workbook（filename[，options]）`，该类实现创建一个XlsxWriter的Workbook对象。

 - Workbook类代表整个电子表格文件，并且存储在磁盘上。
 - 参数filename（String类型）为创建的Excel文件存储路径；
 - 参数options（Dict类型）为可选的Workbook参数，一般作为初始化工作表内容格式，例如值为{‘strings_to_numbers’：True}表示使用worksheet.write（）方法时激活字符串转换数字。

方法：

#### 3.1.1 `add_worksheet（[sheetname]）`
 添加一个新的工作表，
 参数`sheetname`（String类型）为可选的工作表名称，默认为`Sheet1`。
#### 3.1.2 `add_format（[properties]）`
在工作表中创建一个新的格式对象来格式化单元格。
参数`properties（dict类型）`为指定一个格式属性的字典，
例如：设置一个加粗的格式对象，`workbook.add_format（{‘bold’：True}）`。通过Format methods（格式化方法）也可以实现格式的设置，等价的设置加粗格式代码如下：

```python
#通过字典的方式直接设置格式
workfomat = workbook.add_format({
     'bold':  True,                 #字体加粗
     'border':1,                    #单元格边框宽度
     'align':    'center',          #对齐方式
     'valign':   'vcenter',         #字体对齐方式
     'fg_color': '#F4B084',         #单元格背景颜色
 })

#通过format对象的方式设置单元格格式
 workfomat = workbook.add_format()
 workfomat.set_bold(1)                #设置边框宽度
 workfomat.set_num_format('0.00')     #格式化数据格式为小数点后两位
 workfomat.set_align('center')        #设置对齐方式
 workfomat.set_fg_color('blue')       #设置单元格背景颜色
 workfomat.set_bg_color('red')        #设置单元格背景颜色 (经测试和上边的功能一样)


```

#### 3.1.3 `add_chart（options）`
在工作表中创建一个图表对象，内部是通过insert_chart（）方法来实现.
参数`options（dict类型）`为图表指定一个字典属性
例如：设置一个线条类型的图表对象，代码为`chart=workbook.add_chart({‘type’：’line’})`。
#### 3.1.4  `close（）`
作用是关闭工作表文件，如workbook.close（）

```python
#!/usr/bin/python
#---coding:-utf-8---

import xlsxwriter

workbook = xlsxwriter.Workbook('demo2.xlsx')    #创建一个Excel文件 worksheet =
worksheet1 = workbook.add_worksheet()    # Sheet1 
worksheet2 = workbook.add_worksheet('Foglio2')    # Foglio2 
worksheet3 = workbook.add_worksheet('Data')      # Data 
worksheet4 = workbook.add_worksheet()                #Sheet4
workbook.close()    #关闭Excel文件
```
输出：
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020041916534631.png)
### 3.2 Worksheet类

Worksheet类代表了一个Excel工作表，是XlsxWriter模块操作Excel内容最核心的一个类，例如将数据写入单元格或工作表格式布局等。Worksheet对象不能直接实例化，取而代之的是通过Workbook对象调用`add_worksheet（）`方法来创建。

```python
 workbook.add_worksheet()
```

Worksheet类提供了非常丰富的操作Excel内容的方法，其中几个常用的方法如下：

 #### 3.2.1  `write（row，col，*args）`
 写普通数据到工作表的单元格。
 参数row为行坐标，col为列坐标，坐标索引起始值为0；*args无名字参数为数据内容，可以为数字、公式、字符串或格式对象。为了简化不同数据类型的写入过程，write方法已经作为其他更加具体数据类型方法的别名，包括：

```python
·write_string（）写入字符串类型数据，如：

worksheet.write_string（0， 0， 'Your text here'）；
·write_number（）写入数字类型数据，如：

worksheet.write_number（'A2'， 2.3451）；
·write_blank（）写入空类型数据，如：

worksheet.write（'A2'， None）；
·write_formula（）写入公式类型数据，如：

worksheet.write_formula（2， 0， '=SUM（B1：B5）'）；
·write_datetime（）写入日期类型数据，如：

worksheet.write_datetime（7， 0，datetime.datetime.strptime（'2013-01-23'， '%Y-%m-%d'），workbook.add_format（{'num_format'： 'yyyy-mm-dd'}））；
·write_boolean（）写入逻辑类型数据，如：

worksheet.write_boolean（0， 0， True）；
·write_url（）写入超链接类型数据，如：

worksheet.write_url（'A1'， 'ftp：//www.python.org/'）。
```

下列通过具体的示例来观察别名write方法与数据类型方法的对应关系，代码如下：
实例1：
```python
#!/usr/bin/python
#---coding:-utf-8---

import xlsxwriter

workbook = xlsxwriter.Workbook('demo3.xlsx')    #创建一个Excel文件 worksheet =
worksheet = workbook.add_worksheet()    # Sheet1 
worksheet.write(0, 0, 'Hello')      # write_string（） 
worksheet.write(1, 0, 'World')          # write_string（） 
worksheet.write(2, 0, 2)                # write_number（）
worksheet.write(3,0, 3.00001)         # write_number（） 
worksheet.write(4, 0, '=SIN(PI()/4)')   # write_formula（） 
worksheet.write(5, 0, '')               # write_blank（） 
worksheet.write(6, 0, None)             # write_blank（）
workbook.close()    #关闭Excel文件
```
输出：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200419170703795.png)

实例2：循环写入

```python
import xlsxwriter

 # Create a workbook and add a worksheet.
 workbook = xlsxwriter.Workbook('Expenses02.xlsx')
 worksheet = workbook.add_worksheet()

 # 添加用于突出显示单元格的粗体格式。
 bold = workbook.add_format({'bold': True})

 # 为显式钱的单元格添加数字格式。
 money = workbook.add_format({'num_format': '$#,##0'})

 # Write some data headers.
 worksheet.write('A1', 'Item', bold)
 worksheet.write('B1', 'Cost', bold)

 # Some data we want to write to the worksheet.
 expenses = (
     ['Rent', 1000],
     ['Gas',   100],
     ['Food',  300],
     ['Gym',    50],
 )

 # Start from the first cell below the headers.
 row = 1
 col = 0

 # Iterate over the data and write it out row by row.
 for item, cost in (expenses):
     worksheet.write(row, col,     item)
     worksheet.write(row, col + 1, cost, money)
     row += 1

 # Write a total using a formula.
 worksheet.write(row, 0, 'Total',       bold)
 worksheet.write(row, 1, '=SUM(B2:B5)', money)

 workbook.close()
```
输出：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200419210731969.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)


 #### 3.2.2  `set_row（row，height，cell_format，options）`
 设置行单元格的属性。
  参数row（int类型）指定行位置，起始下标为0；
  参数height（float类型）设置行高，单位像素；
  参数cell_format（format类型）指定格式对象；
  参数options（dict类型）设置行hidden（隐藏）、level（组合分级）、collapsed（折叠）。操作示例如下：

```python
#!/usr/bin/python
#---coding:-utf-8---

import xlsxwriter

workbook = xlsxwriter.Workbook('demo4.xlsx')    #创建一个Excel文件 worksheet =
worksheet = workbook.add_worksheet()    # Sheet1 
worksheet.write('A1', 'Hello')     #在A1单元格写入'Hello'字符串 
cell_format = workbook.add_format({'bold': True})    #定义一个加粗的格式对象 
worksheet.set_row(0, 40, cell_format)    #设置第1行单元格高度为40像素，且引用加粗   格式对象 
worksheet.set_row(1, None, None, {'hidden': True})   #隐藏第2行单元格
workbook.close()    #关闭Excel文件
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200419171353874.png)

#### 3.2.3  `set_column（first_col，last_col，width，cell_format，options）`
作用为设置一列或多列单元格属性。
 参数first_col（int类型）指定开始列位置，起始下标为0；
 参数last_col（int类型）指定结束列位置，起始下标为0，可以设置成与first_col一样；
 参数width（float类型）设置列宽；
 参数cell_format（Format类型）指定格式对象；
 参数options（dict类型）设置行hidden（隐藏）、level（组合分级）、collapsed（折叠）。
 操作示例如下：

```python
#!/usr/bin/python
#---coding:-utf-8---

import xlsxwriter

workbook = xlsxwriter.Workbook('demo6.xlsx')    #创建一个Excel文件 worksheet =
worksheet = workbook.add_worksheet()    # Sheet1 
worksheet.write('A1', 'Hello')     #在A1单元格写入'Hello'字符串 
worksheet.write('B1', 'World')    #在B1单元格写入'World'字符串 
cell_format = workbook.add_format({'bold': True})    #定义一个加粗的格式对象设置0到1即（A到B） 列单元格宽度为10像素，且引用加粗格式对象 
worksheet.set_column(0,1, 10,cell_format)
worksheet.set_column('C:D', 20)    #设置C到D列单元格宽度为20像素 
worksheet.set_column('E:G', None, None, {'hidden': 1})    #隐藏E到G列单元格
workbook.close()    #关闭Excel文件
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200419172652434.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)




#### 3.2.4 `insert_image（row，col，image[，options]）`
插入图片到指定单元格，支持PNG、JPEG、BMP等图片格式。
 参数row为行坐标，col为列坐标，坐标索引起始值为0；
 参数image（string类型）为图片路径；
 参数options（dict类型）为可选参数，作用是指定图片的位置、比例、链接URL等信息。
 操作示例如下：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200419173107778.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
### 3.3 Chart类

Chart类实现在XlsxWriter模块中图表组件的基类，支持的图表类型包括面积、条形图、柱形图、折线图、饼图、散点图、股票和雷达等，一个图表对象是通过Workbook（工作簿）的add_chart方法创建，通过{type，’图表类型’}字典参数指定图表的类型，语句如下：
 

```python
(chart = workbook.add_chart({type, 'column'})   
```

 
创建一个column（柱形）图表 更多图表类型说明：

```python
area：创建一个面积样式的图表；

bar：创建一个条形样式的图表；

column：创建一个柱形样式的图表；

line：创建一个线条样式的图表；

pie：创建一个饼图样式的图表；

scatter：创建一个散点样式的图表；

stock：创建一个股票样式的图表；

radar：创建一个雷达样式的图表。
```

然后再通过Worksheet（工作表）的insert_chart（）方法插入到指定位置，语句如下：

`worksheet.insert_chart（'A7', chart）`    #在A7单元格插入图表

下面介绍chart类的几个常用方法：
#### 3.3.1 `chart.add_series（options）`
作用为添加一个数据系列到图表.
参数options（dict类型）设置图表系列选项的字典，操作示例如下：

```python
chart.add_series({
     'categories': '=Sheet1!$A$1:$A$5',
     'values':     '=Sheet1！$B$1:$B$5',
     'line':       {'color': 'red'},
 })
```

add_series方法最常用的三个选项为`categories、values、line`，其中categories作为是设置图表类别标签范围；values为设置图表数据范围；line为设置图表线条属性，包括颜色、宽度等。
#### 3.3.2 `set_x_axis（options）`
设置图表X轴选项，示例代码如下，效果图如图3-7所示。

```python
chart.set_x_axis({
     'name': 'Earnings per Quarter',    #设置X轴标题名称
     'name_font': {'size': 14, 'bold': True},  #设置X轴标题字体属性
     'num_font':  {'italic': True },    #设置X轴数字字体属性
})
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200419193447328.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
#### 3.3.3 `set_size（options）`
设置图表大小，如chart.set_size（{‘width’：720，’height’：576}），其中width为宽度，height为高度。

#### 3.3.4 `set_title（options）`
设置图表标题，如chart.set_title（{‘name’：’Year End Results’}）
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200419193634744.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
#### 3.3.5 `set_style(style_id)`
设置图表样式，style_id为不同数字则代表不同样式，如chart.set_style（37）
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200419193805764.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
#### 3.3.6 `set_table（options）`
设置X轴为数据表格形式，如`chart.set_table（）`
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200419193853674.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
## 4 实战
###  4.1 画线图

```python
import xlsxwriter                     #导入模块
 
workbook = xlsxwriter.Workbook('new_excel.xlsx')   #创建新的excel

worksheet = workbook.add_worksheet('sheet1')        #创建新的sheet

headings = ['Number','testA','testB']             #创建表头

data = [
    ['2017-9-1','2017-9-2','2017-9-3','2017-9-4','2017-9-5','2017-9-6'],
    [10,40,50,20,10,50],
    [30,60,70,50,40,30],
]                                                       #自己造的数据

worksheet.write_row('A1',headings)

worksheet.write_column('A2',data[0])
worksheet.write_column('B2',data[1])
worksheet.write_column('C2',data[2])                    #将数据插入到表格中

chart_col = workbook.add_chart({'type':'line'})        #新建图表格式 line为折线图
chart_col.add_series(                                   #给图表设置格式，填充内容
    {
        'name':'=sheet1!$B$1',
        'categories':'=sheet1!$A$2:$A$7',
        'values':   '=sheet1!$B$2:$B$7',
        'line': {'color': 'red'},
    }
)

chart_col.set_title({'name':'测试'})
chart_col.set_x_axis({'name':"x轴"})
chart_col.set_y_axis({'name':'y轴'})          #设置图表表头及坐标轴

chart_col.set_style(1)

worksheet.insert_chart('A10',chart_col,{'x_offset':25,'y_offset':10})   #放置图表位置

workbook.close()
```
输出：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200419213324897.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

### 4.2 定制自动化业务流量报表周报

```python
#!/usr/bin/python
#coding:utf-8 

import xlsxwriter 

workbook = xlsxwriter.Workbook('chart.xlsx')    #创建一个Excel文件 worksheet = 

worksheet = workbook.add_worksheet()    #创建一个工作表对象 

chart = workbook.add_chart({'type': 'column'})   #创建一个图表对象 

#定义数据表头列表 

title = [u'业务名称', u'星期一', u'星期二', u'星期三', u'星期四', u'星期五', u'星期六', u'星期日', u'平均流量'] 
buname= [u'业务官网', u'新闻中心', u'购物频道', u'体育频道', u'亲子频道']    #定义频道名称 

#定义5频道一周7天流量数据列表 

data = [ 

    [150, 152, 158, 149, 155, 145, 148],

     [89, 88, 95, 93, 98, 100, 99],

     [201, 200, 198, 175, 170, 198, 195],

     [75, 77, 78, 78, 74, 70, 79],

     [88, 85, 87, 90, 93, 88, 84],

 ] 

format=workbook.add_format()    #定义format格式对象 

format.set_border(1)    #定义format对象单元格边框加粗（1像素）的格式 

format_title=workbook.add_format()    #定义format_title格式对象 

format_title.set_border(1)   #定义format_title对象单元格边框加粗（1像素）的格式 

format_title.set_bg_color('#cccccc')   #定义format_title对象单元格背景颜色为                          #'#cccccc'的格式 

format_title.set_align('center')    #定义format_title对象单元格居中对齐的格式 

format_title.set_bold()    #定义format_title对象单元格内容加粗的格式 

format_ave=workbook.add_format()    #定义format_ave格式对象 

format_ave.set_border(1)    #定义format_ave对象单元格边框加粗（1像素）的格式 

format_ave.set_num_format('0.00')   #定义format_ave对象单元格数字类别显示格式

#下面分别以行或列写入方式将标题、业务名称、流量数据写入起初单元格，同时引用不同格式对象 

worksheet.write_row('A1', title, format_title)   

worksheet.write_column('A2', buname, format) 

worksheet.write_row('B2', data[0], format) 

worksheet.write_row('B3', data[1], format) 

worksheet.write_row('B4', data[2], format) 

worksheet.write_row('B5', data[3], format) 

worksheet.write_row('B6', data[4], format) 

#定义图表数据系列函数 

def chart_series(cur_row):

     worksheet.write_formula('I'+cur_row, \

      '=AVERAGE(B'+cur_row+':H'+cur_row+')', format_ave)    #计算（AVERAGE函数）频   道周平均流量     

     chart.add_series({

         'categories': '=Sheet1!$B$1:$H$1',    #将“星期一至星期日”作为图表数据标签（X轴） 

        'values': '=Sheet1!$B$'+cur_row+':$H$'+cur_row,    #频道一周所有数据作           为数据区域 

        'line': {'color': 'black'},    #线条颜色定义为black（黑色）

         'name': '=Sheet1!$A$'+cur_row,    #引用业务名称为图例项

     })

for row in range(2, 7):    #数据域以第2~6行进行图表数据系列函数调用     

    chart_series(str(row))

 #chart.set_table（）    #设置X轴表格格式，本示例不启用

 #chart.set_style（30）    #设置图表样式，本示例不启用

chart.set_size({'width': 577, 'height': 287})    #设置图表大小

chart.set_title ({'name': u'业务流量周报图表'})    #设置图表（上方）大标题 

chart.set_y_axis({'name': 'Mb/s'})    #设置y轴（左侧）小标题 

worksheet.insert_chart('A8', chart)    #在A8单元格插入图表 

workbook.close()    #关闭Excel文档

```
输出：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200419205321924.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

参考：

 - [Python | Create and write on excel file using xlsxwriter module](https://www.geeksforgeeks.org/python-create-and-write-on-excel-file-using-xlsxwriter-module/)
 - [Automate Excel with Python and XlsxWriter Part 1: Getting Started](https://www.youtube.com/watch?v=Cbt9JRbNBV0)

