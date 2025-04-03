![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0111b4bf6d81d1289ecb637db5c00238.png)






## 1. 插入标题

```bash
大标题  
====  
```

```bash
标题  
-------  
```

```c
# 一级标题  
## 二级标题  
### 三级标题  
#### 四级标题  
##### 五级标题  
###### 六级标题  
```

---

## 2. 显示文本

```c
Hello, World #普通文本
```

```bash
Thank `You` . Please `Call` Me `Coder`  //高亮文本
```

```bash
[唯美的图片网站](https://www.pexels.com/zh-cn/)   //超链接文本
```

[唯美的图片网站](https://www.pexels.com/zh-cn/)

## 3. 字体格式
1.文字加粗：

```bash
**这是加粗的文字** 
```

2.斜体文字：

```bash
*这是斜体文字* 
```

3.自定义字体大小：

```bash
<font size=4>我是变大的字</font>
```
## 4. 插入表格
1居中插入

```bash
 一个普通标题 | 一个普通标题 | 一个普通标题 |
| ------ | ------ | ------ |
| 短文本 | 中等文本 | 稍微长一点的文本 |
| 稍微长一点的文本 | 短文本 | 中等文本 |
```
 一个普通标题 | 一个普通标题 | 一个普通标题 |
| ------ | ------ | ------ |
| 短文本 | 中等文本 | 稍微长一点的文本 |
| 稍微长一点的文本 | 短文本 | 中等文本 |

2.居左或居右
```bash
| 左对齐标题 | 右对齐标题 | 居中对齐标题 |
| :------| ------: | :------: |
| 短文本 | 中等文本 | 稍微长一点的文本 |
| 稍微长一点的文本 | 短文本 | 中等文本 |
```
| 左对齐标题 | 右对齐标题 | 居中对齐标题 |
| :------| ------: | :------: |
| 短文本 | 中等文本 | 稍微长一点的文本 |
| 稍微长一点的文本 | 短文本 | 中等文本 |

---
## 5. 插入图片
语法：
```bash
![alet text](图片链接 "optional title")
![alet text](/home/picture/1.png)
```
###  设置图片大小

```bash
<img src="https://i-blog.csdnimg.cn/blog_migrate/ca99e804f90f4c92d1e7a04a8a96a2e9.png" width="50%" height="50%">
```
<img src="https://i-blog.csdnimg.cn/blog_migrate/ca99e804f90f4c92d1e7a04a8a96a2e9.png" width="50%" height="50%">

###  5.1 图片加超链接

```bash
[![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ca99e804f90f4c92d1e7a04a8a96a2e9.png#pic_center)](https://www.rottentomatoes.com/m/men_2022)
```
[![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ca99e804f90f4c92d1e7a04a8a96a2e9.png#pic_center)](https://www.rottentomatoes.com/m/men_2022)

## 6. 插入符号

```bash
* k8s      //插入*
* docker
* prometheus
```
* k8s      //插入*
* docker
* prometheus

## 7. 设置超链接
```
[Google](http://www.google.com/)
```
效果
[Google](http://www.google.com/)


## 8. 设置颜色

```bash
<font face="黑体">我是黑体字</font>
<font face="微软雅黑">我是微软雅黑</font>
<font face="STCAIYUN">我是华文彩云</font>
<font color=red>我是红色</font>
<font color=#008000>我是绿色</font>
<font color=Blue>我是蓝色</font>
<font size=5>我是尺寸</font>
<font face="黑体" color=green size=5>我是黑体，绿色，尺寸为5</font>

```
<font face="黑体">我是黑体字</font>
<font face="微软雅黑">我是微软雅黑</font>
<font face="STCAIYUN">我是华文彩云</font>
<font color=red>我是红色</font>
<font color=#008000>我是绿色</font>
<font color=Blue>我是蓝色</font>
<font size=5>我是尺寸</font>
<font face="黑体" color=green size=5>我是黑体，绿色，尺寸为5</font>


## 9. 设置背景

```bash
<table><tr><td bgcolor=orange>背景色yellow</td></tr></table>

```
<table><tr><td bgcolor=orange>背景色yellow</td></tr></table>


## 10. 插入换行

在一行的末尾添加两个或多个空格，然后按回车键,即可创建一个换行( <br> )。

## 11. 插入一个横线

```bash
---
```
效果：

---


## 12. 插入空行

```bash
<br/> 
```
或者

```bash
&nbsp; 
```


## 13. 合并单元格

### 13.1 第二张方法
```bash
|    姓名    |   语文   |   数学   |   英语   |
| ---------- | -------- | -------- | -------- |
| 学生甲     |   95     |   98     |   90     |
| 学生乙     |   88     |   92     |   95     |
| 学生丙     |   92     |   87     |   89     |
| <th colspan="3">总分</th> |   275    |   277    |   274    |
| <th colspan="3">平均分</th> | 91.67 | 92.33 | 91.33 |
```

|    姓名    |   语文   |   数学   |   英语   |
| ---------- | -------- | -------- | -------- |
| 学生甲     |   95     |   98     |   90     |
| 学生乙     |   88     |   92     |   95     |
| 学生丙     |   92     |   87     |   89     |
| <th colspan="3">总分</th> |   275    |   277    |   274    |
| <th colspan="3">平均分</th> | 91.67 | 92.33 | 91.33 |




### 13.2 第二种方法

```bash
<table>
	<tr>
	    <th colspan="3">计算机网络分层模型</th>
	</tr >
	<tr>
	    <td >OSI七层模型</td>
	    <td>TCP/IP四层模型</td>
	    <td>TCP/IP五层模型</td>  
	</tr >
	<tr >
	    <td>应用层</td>
	    <td rowspan="3">应用层</td>
	    <td rowspan="3">应用层</td>
	</tr>
	<tr>
	    <td>表示层</td>
	</tr>
	<tr>
	    <td>会话层</td>
	</tr>
	<tr>
	    <td>传输层</td>
	    <td>传输层</td>
       <td>传输层</td>
	</tr>
	<tr>
       <td>网络层</td>
	    <td>网络层</td>
       <td>网络层</td>
	</tr>
	<tr>
	    <td>数据链路层</td>
	    <td rowspan="2">网络接口层</td>
       <td>数据链路层</td>
	</tr>
	<tr>
	    <td>物理层</td>
	    <td>物理层</td>
	</tr>
</table>

```

<table>
	<tr>
	    <th colspan="3">计算机网络分层模型</th>
	</tr >
	<tr>
	    <td >OSI七层模型</td>
	    <td>TCP/IP四层模型</td>
	    <td>TCP/IP五层模型</td>  
	</tr >
	<tr >
	    <td>应用层</td>
	    <td rowspan="3">应用层</td>
	    <td rowspan="3">应用层</td>
	</tr>
	<tr>
	    <td>表示层</td>
	</tr>
	<tr>
	    <td>会话层</td>
	</tr>
	<tr>
	    <td>传输层</td>
	    <td>传输层</td>
       <td>传输层</td>
	</tr>
	<tr>
       <td>网络层</td>
	    <td>网络层</td>
       <td>网络层</td>
	</tr>
	<tr>
	    <td>数据链路层</td>
	    <td rowspan="2">网络接口层</td>
       <td>数据链路层</td>
	</tr>
	<tr>
	    <td>物理层</td>
	    <td>物理层</td>
	</tr>
</table>


## 14. 单元格内换行

```bash
<table>
	<tr>
	    <th>指标</th>
	    <th>数量</th>
	    <th>描述</th>
	    <th>时间</th>
	</tr >
	<tr>
	    <td rowspan="2">部署集群</td>
	    <td rowspan="7">7套</td>
	    <td>a-p-wms-k8s-134.43
	    <br>a-s-wms-118.63
	    <br>a-s-cs-k8s-118.123 
	    <br>a-p-k8sv2-134.83</td> 
	</tr >
	<tr >
	    <td >a-p-k8sv2-134.83</td>
	</tr>
</table>
```

<table>
	<tr>
	    <th>指标</th>
	    <th>数量</th>
	    <th>描述</th>
	    <th>时间</th>
	</tr >
	<tr>
	    <td rowspan="2">部署集群</td>
	    <td rowspan="7">7套</td>
	    <td>a-p-wms-k8s-134.43
	    <br>a-s-wms-118.63
	    <br>a-s-cs-k8s-118.123 
	    <br>a-p-k8sv2-134.83</td> 
	</tr >
	<tr >
	    <td >a-p-k8sv2-134.83</td>
	</tr>
</table>


参考：
 - [Markdown手册](https://xianbai.me/learn-md/article/about/readme.html)
 - [RGB颜色对照表](https://tool.oschina.net/commons?type=3)
