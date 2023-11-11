
## 1.打印日志输出
SecureCRT看不到前几分钟操作的内容，或者想把通过vi命令查看的日志输出到log文件（在懒得下载日志文件的情况下），所以接下来就这样操作：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200924104400565.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200924104402530.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70#pic_center)

文件保存路径 C:\secureCRT\logs\session_%Y_%M_%D_%H.log
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200924104509733.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70#pic_center)


最后记得勾选保存会话日志

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020092410445474.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70#pic_center)

这样每次操作完，就会自动记录操作产生屏幕内容的log日志了，生成的日志见下图，以 当前日期和IP地址记录了，注意：第二个图中选择的是Append to File 追加日志到文件，就算关闭了SecureCRT，再启动，登陆同一个服务器，日志在当天也是追加的。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200924104525510.png#pic_center)





题外话，修改默认卷屏行数当做一个操作，屏幕输出有上百行，当需要将屏幕回翻时，这个设置会有很大帮助，默认为500行，可以改为10000行，不用担心找不到了。 选项 => 全局选项 => Default Session => Edit Default Settings => Terminal => Emulation => Scrollback 修改为32000。


