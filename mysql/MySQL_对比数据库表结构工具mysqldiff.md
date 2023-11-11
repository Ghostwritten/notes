```bash
mysqldiff --server1=user:pass@host:port:socket --server2=user:pass@host:port:socket db1.object1:db2.object1 db3:db4
```

这个语法有两个用法：

 - db1：db2:如果只是指定数据库，那么就将两个数据库中互相缺少的对象显示出来，而对象里面的差异不进行对比；这里的对象包括表、存储过程、函数、触发器等。
 - db1.object1:db2.object1：如果指定了具体表对象，那么就会详细对比两个表的差异，包括表名、字段名、备注、索引、大小写等都有的表相关的对象。

### 参数：

 - --server1：配置server1的连接
 - --server2：配置server2的连接
 - --character-set：配置连接时用的字符集，如果不显示配置默认使用“character_set_client”
 - --width：配置显示的宽度
 - --skip-table-options：这个选项的意思是保持表的选项不变，即对比的差异里面不包括表名、AUTO_INCREMENT,ENGINE, CHARSET等差异。
 - -d DIFFTYPE, --difftype：差异的信息显示的方式，有[unified|context|differ|sql](default: unified),如果使用sql那么就直接生成差异的SQL这样非常方便。
 - --changes-for=：例如--changes-for=server2,那么对比以sever1为主，生成的差异的修改也是针对server2的对象的修改。
 - --show-reverse：这个字面意思是显示相反的意思，其实是生成的差异修改里面同时会包含server2和server1的修改

## 实战
```bash
mysqldiff  --server1=root:root@localhost --server2=root:root@localhost --changes-for=server2   --show-reverse   --difftype=sql study.test1:study.test2
```
参考链接：
[pursuer.chen](https://www.cnblogs.com/chenmh/p/5447205.html)
