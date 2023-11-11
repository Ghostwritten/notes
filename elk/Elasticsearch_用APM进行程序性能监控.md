


----
## 1. Elastic 全栈监控
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210319160309375.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
## 2. 核心应用指标
● 请求响应时间
● 未处理的错误及异常
● 可视化调用关系
● 发现性能瓶颈
● 代码下钻

##  3. apm
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210319160355505.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210319172854185.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210319172916404.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/2021031917293018.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210319173058138.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210319173153768.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/2021031917364448.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

```bash
[root@master apm]# tar -xvfz apm-server-7.3.1-linux-x86_64.tar.gz 
tar: z：无法 open: 没有那个文件或目录
tar: Error is not recoverable: exiting now
[root@master apm]# tar -zxvf apm-server-7.3.1-linux-x86_64.tar.gz 
apm-server-7.3.1-linux-x86_64/kibana/
apm-server-7.3.1-linux-x86_64/.build_hash.txt
apm-server-7.3.1-linux-x86_64/ingest/
apm-server-7.3.1-linux-x86_64/ingest/pipeline/
apm-server-7.3.1-linux-x86_64/ingest/pipeline/definition.json
apm-server-7.3.1-linux-x86_64/LICENSE.txt
apm-server-7.3.1-linux-x86_64/fields.yml
apm-server-7.3.1-linux-x86_64/apm-server
apm-server-7.3.1-linux-x86_64/NOTICE.txt
apm-server-7.3.1-linux-x86_64/README.md
apm-server-7.3.1-linux-x86_64/apm-server.yml
[root@master apm]# cd apm-server-7.3.1-linux-x86_64/
[root@master apm-server-7.3.1-linux-x86_64]# ls
apm-server  apm-server.yml  fields.yml  ingest  kibana  LICENSE.txt  NOTICE.txt  README.md
[root@master apm-server-7.3.1-linux-x86_64]# ./apm-server 
```
去kibana界面设置
![在这里插入图片描述](https://img-blog.csdnimg.cn/2021031917430968.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210319174348753.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)


## APM 如何整合到 Elastic Stack
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210319160428142.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
## 4. Demo
● 安装配置

 - ○ [https://www.elastic.co/cn/downloads/past-releases/apm-server-7-1-0](https://www.elastic.co/cn/downloads/past-releases/apm-server-7-1-0)

● 运行 Spring + MySQL 程序
● 运行性能测试脚本
● 查看结果 Dashboard

