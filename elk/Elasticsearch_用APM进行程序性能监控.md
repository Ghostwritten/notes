


----
## 1. Elastic 全栈监控
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/cbbfcf0e074e448e7586d9b79e47ff0a.png)
## 2. 核心应用指标
● 请求响应时间
● 未处理的错误及异常
● 可视化调用关系
● 发现性能瓶颈
● 代码下钻

##  3. apm
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/bd4bf49f53fc8f576c7d2796695c8fde.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b952932f6255d9b24039bbd78818fd62.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/163b089e809f437bd1861df7821c2bfd.png)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f7b3b40a534daf5eba3ad769f6c2d572.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f15c1dfe23329cef47f7f7a39e85a6ed.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/763864247af39bddbb52f72cc0b0a759.png)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/865bbccaf4d2d96eb5379a0b0d5591cb.png)

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
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/43402fcf3e98b36eb5dd90f5c44eda0a.png)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/5ab23061e5ef188c5b0049e4a519133c.png)


## APM 如何整合到 Elastic Stack
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/de9e43d69c7cc0b5ff4250e7ca95fabd.png)
## 4. Demo
● 安装配置

 - ○ [https://www.elastic.co/cn/downloads/past-releases/apm-server-7-1-0](https://www.elastic.co/cn/downloads/past-releases/apm-server-7-1-0)

● 运行 Spring + MySQL 程序
● 运行性能测试脚本
● 查看结果 Dashboard

