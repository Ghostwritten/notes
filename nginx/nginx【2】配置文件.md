![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/00c1bedde6563e8f6f518fa220ad6cd3.png)

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

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a0cb4dff044065ac76a11060cc9bc959.png)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/21a26ed3d881ef8be731ab8c5576c42f.png)

