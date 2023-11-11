![在这里插入图片描述](https://img-blog.csdnimg.cn/c32c9de860274e64aa25741c7dfe7dec.png)

Nginx配置语法

 1. 每条指令以；分号结尾，指令与参数间以空格符号分隔
 2. 配置文件由指令与指令块构成
 3. 指令块以｛｝大括号将多条指令组织在一起
 4. 使用#符号添加注释，提高可读性
 5. include语句允许组合多个配置文件以提升可维护性
 6. 使用$符号使用变量
 7. 部分指令的参数支持正则表达式

指令块：

 - http
 - server
 - location
 - upstream

![在这里插入图片描述](https://img-blog.csdnimg.cn/e5259d34ac134dec93e68c6421068f78.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/1fd11325f483462ab5907002009cdcea.png)

