![在这里插入图片描述](https://img-blog.csdnimg.cn/f87d5a42acbb4820a0e8c1ae9a28ae83.png)
热升级能保证在不停止服务的情况下更换它的binary文件;这个功能非常有用,但在我们执行nginx的binary文件升级过程中,还是可能会遇到很多问题,比如:老的worker进程一直退不掉,新的worker进程出现了问题,我们要考虑使用回滚,或者说我们升级了新的nginx文件以后,会发现很多我们预期的功能或者指向的配置文件出现了错误,下面我们来看下热升级的流程是怎样升级的?

　　第一步:就是把旧的nginx的binary文件替换成新的nginx的binary文件(注意备份旧的nginx配置文件);这里我们只说替换binary文件,因为大部分场景下我们新编译的nginx文件所指定的相应的配置文件的选项,比如说配置文件的目录在哪里,log所在的目录在哪里;必须保证和老的nginx的是一致的;否则的话我们没有办法去复用nginx的conf配置文件;在替换的时候新版本的Linux中会要求在覆盖一个正在使用的文件的时候要加上 -f(cp -f);

　　第二步:向老的nginx启动的master进程发送USR2信号;这时候我们注意到我们没有办法通过nginx的命令行即直接用nginx -s 信号 来处理,这是因为nginx到目前还没有实施这样的信号;

　　第三步:发送完USR2信号以后尼.现有的这个master进程会修改pid文件名,加后缀.oldbin,以便给新的master进程让路;因为我们说过虽然master进程和worker进程它们都可以接收信号,但是为了管理方便我们通常不对worker进程直接发送信号;所以我们依赖于master进程,必须把它的pid保存下来;为了给新的master让它使用pid.bin这样的一个文件名,所以我们先把老的pid文件改名;

　　第四步:master进程用新的nginx二进制文件启动新的master进程;所以到此为止,会出现两个master进程;然后新的master进程会启动新的worker进程,

　　第五步:向老的master进程发送QUIT信号;关闭老的master进程;怎么找到老的master进程尼?我们需要根据ps 看到master进程的进程号;或者通过.oldbin里面找到老的master进程的进程号;老的master进程会优雅的关闭；老的worker进程;

　　第六步:这样的情况下我们的热升级已经结束了,但是老的master进程一直是保存下来的;这是为了回滚;也就是发现新的nginx有问题了;这个时候尼,因为老的master进程还在,所以通过向老的master发送HUP信号;向新的master进程发送QUIT信号;

　　接下来我们来看下具体的流程图:
![在这里插入图片描述](https://img-blog.csdnimg.cn/20c8667b6d9c417fae7cff8145c3a02b.png)

