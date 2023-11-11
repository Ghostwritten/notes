## 目录与文件

贯穿所有文档，你将会注意到很多地方都提到了 `$CATALINA_HOME`。这是 Tomcat 安装的根目录。假如文档中某处出现“该信息应该位于 `$CATALINA_HOME/README.txt` 文件中”，那它其实是指在 Tomcat 安装根目录下查看 `README.txt` 文件。另外，还可以配置多个 Tomcat 实例，只需为每一个实例都定义一个 `$CATALINA_BASE` 即可。当然，如果没有配置多个实例，那么 $CATALINA_BASE 其实就相当于 $CATALINA_HOME。

以下是 Tomcat 的一些关键目录：

`/bin` 存放用于启动及关闭的文件，以及其他一些脚本。其中，UNIX 系统专用的 *.sh 文件在功能上等同于 Windows 系统专用的 *.bat 文件。因为 Win32 的命令行缺乏某些功能，所以又额外地加入了一些文件。
`/conf` 配置文件及相关的 DTD。其中最重要的文件是 server.xml，这是容器的主配置文件。
`/log` 日志文件的默认目录。
`/webapps` 存放 Web 应用的相关文件。

## UNIX 守护进程

利用 `commons-daemon` 工程的 `jsvc` 工具，可以将 `Tomcat` 作为一个守护进程来运行。Tomcat 的二进制发行版中包含着 jsvc 的源代码包，它需要编译。构建 jsvc 需要一个 `C ANSI` 编译器（比如 GCC）、GNU Autoconf，以及一个 JDK。

在运行脚本之前，先将环境变量 `JAVA_HOME` 设置为 `JDK` 的基础路径。在调用 `./configure` 脚本时，需要使用 `--with-java` 参数来指定 JDK 路径，比如：`./configure --with-java=/usr/java`。

使用下列命令应该就能返回一个编译好的 jsvc 二进制可执行文件，位于 `$CATALINA_HOME/bin` 目录中——这需要的前提条件是：使用了 GNU TAR，并且将环境变量 CATALINA_HOME 指向 Tomcat 安装基本路径。

请注意，应该使用 GNU make（gmake）而不是 FreeBSD 系统下的原生 BSD make。

```bash
cd $CATALINA_HOME/bin
tar xvfz commons-daemon-native.tar.gz
cd commons-daemon-1.0.x-native-src/unix
./configure
make
cp jsvc ../..
cd ../..
```

使用下列命令，Tomcat 就可以作为一个守护进程来运行了。

```bash
CATALINA_BASE=$CATALINA_HOME
cd $CATALINA_HOME
./bin/jsvc \
    -classpath $CATALINA_HOME/bin/bootstrap.jar:$CATALINA_HOME/bin/tomcat-juli.jar \
    -outfile $CATALINA_BASE/logs/catalina.out \
    -errfile $CATALINA_BASE/logs/catalina.err \
    -Dcatalina.home=$CATALINA_HOME \
    -Dcatalina.base=$CATALINA_BASE \
    -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager \
    -Djava.util.logging.config.file=$CATALINA_BASE/conf/logging.properties \
    org.apache.catalina.startup.Bootstrap
```

如果 JVM 默认使用的是服务器 VM，而不是客户端 VM，则可能还需要指定 `-jvm server`。这一点已经在 OS X 系统下得到证实。

jsvc 还有其他一些有用的参数。比如：-user 就能让守护进程初始化完成后切换到另一个用户，从而能以非特权用户来运行 Tomcat，同时又能使用特权端口。不过要注意的是，如果使用这个选项来以根用户运行 Tomcat，需要禁用 `org.apache.catalina.security.SecurityListener` 检查，这个检查是用来防止以根用户来运行 Tomcat 的。

jsvc --help 参数会提供完整的 jsvc 用途信息。尤其是 -debug 参数，它对于调试 jsvc 运行中出现的问题是非常有用的一个工具。

`$CATALINA_HOME/bin/daemon.sh` 可以作为一个模板，利用 `jsvc /etc/init.d/` 在启动时自动开启 Tomcat。

注意，要想以上述方式运行 Tomcat，`Commons-Daemon JAR` 文件必须位于运行时的类路径上。Commons-Daemon JAR 文件在 `bootstrap.jar` 清单的类路径项中。如果某个 Commons-Daemon 类出现了 `ClassNotFoundException`（无法找到类） 或 `NoClassDefFoundError`（无法找到类定义） 这样的错误，那么在加载 jsvc 时将 Commons-Daemon JAR 添加到 -cp 参数中。

参考链接：
[https://www.w3cschool.cn/tomcat/mew21k93.html](https://www.w3cschool.cn/tomcat/mew21k93.html)
