# linux Command sysbench 线程压力测试工具



---

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/84711ed17a78a04c2bcc37c4c24255e4.gif#pic_center)

## 1. 简介
 [sysbench](https://github.com/akopytov/sysbench) 是一个开源的、模块化的、跨平台的多线程性能测试工具，可以用来进行CPU、内存、磁盘I/O、线程、数据库的性能测试。目前支持的数据库有MySQL、Oracle和PostgreSQL。当前功能允许测试的系统参数有：

 - file I/O performance （文件I / O性能）
 - scheduler performance （调度性能）
 - memory allocation and transfer speed （内存分配和传输速度）
 - POSIX threads implementation performance （POSIX线程执行绩效）
 - database server performance (OLTP benchmark) （数据库服务器性能）


##  2. 安装
### 2.1 Ubuntu 
```bash
apt-get install sysbench
```
### 2.2 centos

```bash
yum -y install epel-release
yum -y update
yum install sysbench
```

###  2.3 编译安装
下载：  [sysbench-1.0.20](https://github.com/akopytov/sysbench/releases)
```bash
apt-get -y install automake libtool libtool-bin
sysbench-1.0.20.zip
unzip 
cd sysbench-1.0.20
mkdir /usr/sysbench/
./autogen.sh
```

> 要是出现：perl: warning: Falling back to the standard locale
> ("C")。则需要设置locale:

```bash
echo "export LC_ALL=C" >> /root/.bashrc
source /root/.bashrc
```
要是没有安装开发包，即`/usr/include/` 目录下面没有mysql文件夹。则需要执行安装(版本为12.04):

```bash
sudo apt-get -y install libmysqlclient-dev
sudo apt-get -y install libmysqld-dev
sudo apt-get -y install libmysqld-pic
```
执行configure操作：

```bash
./configure --prefix=/usr/sysbench/ --with-mysql-includes=/usr/include/mysql/ --with-mysql-libs=/usr/lib/mysql/ --with-mysql

```

说明：

 - --prefix=/usr/sysbench/                    ：指定sysbench的安装目录。
 - --with-mysql-includes=/usr/include/mysql/  ：指定安装mysql时候的includes目录。
 - --with-mysql-libs=/usr/lib/mysql/          ：指定装mysql时候的lib目录。
 - --with-mysql                               ：sysbench默认支持mysql，如果需要测试oracle或者pgsql则需要制定–with-oracle或者–with-pgsql。


```bash
cp /usr/bin/libtool /root/sysbench-1.0.20
make && make install 
```

##  3. 参数

```bash
$ sysbench --help
Usage:
  sysbench [options]... [testname] [command]

Commands implemented by most tests: prepare run cleanup help

General options:
  --threads=N                     number of threads to use [1]
  --events=N                      limit for total number of events [0]
  --time=N                        limit for total execution time in seconds [10]
  --forced-shutdown=STRING        number of seconds to wait after the --time limit before forcing shutdown, or 'off' to disable [off] #超过max-time强制中断。默认是off。
  --thread-stack-size=SIZE        size of stack per thread [64K]  #每个线程的堆栈大小。默认是32K。
  
  --rate=N                        average transactions rate. 0 for unlimited rate [0]
  --report-interval=N             periodically report intermediate statistics with a specified interval in seconds. 0 disables intermediate reports [0]
  --report-checkpoints=[LIST,...] dump full statistics and reset all counters at specified points in time. The argument is a list of comma-separated values representing the amount of time in seconds elapsed from start of test when report checkpoint(s) must be performed. Report checkpoints are off by default. []
  --debug[=on|off]                print more debugging info [off]
  --validate[=on|off]             perform validation checks where possible [off]
  --help[=on|off]                 print help and exit [off]
  --version[=on|off]              print version and exit [off]
  --config-file=FILENAME          File containing command line options
  --tx-rate=N                     deprecated alias for --rate [0]
  --max-requests=N                deprecated alias for --events [0]  #请求的最大数目。默认为10000，0代表不限制。
  --max-time=N                    deprecated alias for --time [0]  #最大执行时间，单位是s。默认是0,不限制
  --num-threads=N                 deprecated alias for --threads [1]  #创建测试线程的数目。默认为1.

Pseudo-Random Numbers Generator options:
  --rand-type=STRING random numbers distribution {uniform,gaussian,special,pareto} [special]
  --rand-spec-iter=N number of iterations used for numbers generation [12]
  --rand-spec-pct=N  percentage of values to be treated as 'special' (for special distribution) [1]
  --rand-spec-res=N  percentage of 'special' values to use (for special distribution) [75]
  --rand-seed=N      seed for random number generator. When 0, the current time is used as a RNG seed. [0]
  --rand-pareto-h=N  parameter h for pareto distribution [0.2]

Log options:
  --verbosity=N verbosity level {5 - debug, 0 - only critical messages} [3]

  --percentile=N       percentile to calculate in latency statistics (1-100). Use the special value of 0 to disable percentile calculations [95]
  --histogram[=on|off] print latency histogram in report [off]

General database options:

  --db-driver=STRING  specifies database driver to use ('help' to get list of available drivers)
  --db-ps-mode=STRING prepared statements usage mode {auto, disable} [auto]
  --db-debug[=on|off] print database-specific debug information [off]


Compiled-in database drivers:
  mysql - MySQL driver
  pgsql - PostgreSQL driver

mysql options:
  --mysql-host=[LIST,...]          MySQL server host [localhost]
  --mysql-port=[LIST,...]          MySQL server port [3306]
  --mysql-socket=[LIST,...]        MySQL socket
  --mysql-user=STRING              MySQL user [sbtest]
  --mysql-password=STRING          MySQL password []
  --mysql-db=STRING                MySQL database name [sbtest]
  --mysql-ssl[=on|off]             use SSL connections, if available in the client library [off]
  --mysql-ssl-cipher=STRING        use specific cipher for SSL connections []
  --mysql-compression[=on|off]     use compression, if available in the client library [off]
  --mysql-debug[=on|off]           trace all client library calls [off]
  --mysql-ignore-errors=[LIST,...] list of errors to ignore, or "all" [1213,1020,1205]
  --mysql-dry-run[=on|off]         Dry run, pretend that all MySQL client API calls are successful without executing them [off]

pgsql options:
  --pgsql-host=STRING     PostgreSQL server host [localhost]
  --pgsql-port=N          PostgreSQL server port [5432]
  --pgsql-user=STRING     PostgreSQL user [sbtest]
  --pgsql-password=STRING PostgreSQL password []
  --pgsql-db=STRING       PostgreSQL database name [sbtest]

Compiled-in tests:
  fileio - File I/O test
  cpu - CPU performance test
  memory - Memory functions speed test
  threads - Threads subsystem performance test
  mutex - Mutex performance test
```
###  3.1 sysbench  --test=fileio help

```bash
$ sysbench  --test=fileio help
sysbench 1.0.11 (using system LuaJIT 2.1.0-beta3)
fileio options:
  --file-num=N   创建测试文件的数量。默认是128
  --file-block-size=N  测试时文件块的大小。默认是16384(16K)
  --file-total-size=SIZE   测试文件的总大小。默认是2G
  --file-test-mode=STRING  文件测试模式{seqwr(顺序写), seqrewr(顺序读写), seqrd(顺序读), rndrd(随机读), rndwr(随机写), rndrw(随机读写)}
  --file-io-mode=STRING   文件操作模式{sync(同步),async(异步),fastmmap(快速map映射),slowmmap(慢map映射)}。默认是sync
  --file-extra-flags=STRING   使用额外的标志来打开文件{sync,dsync,direct} 。默认为空
  --file-fsync-freq=N   执行fsync()的频率。(0 – 不使用fsync())。默认是100
  --file-fsync-all=[on|off] 每执行完一次写操作就执行一次fsync。默认是off
  --file-fsync-end=[on|off] 在测试结束时才执行fsync。默认是on
  --file-fsync-mode=STRING  使用哪种方法进行同步{fsync, fdatasync}。默认是fsync
  --file-merged-requests=N   如果可以，合并最多的IO请求数(0 – 表示不合并)。默认是0
  --file-rw-ratio=N     测试时的读写比例。默认是1.5
```

###  3.2 sysbench  --test=cpu help

```bash
$ sysbench  --test=cpu help
sysbench 1.0.11 (using system LuaJIT 2.1.0-beta3)

cpu options:
  --cpu-max-prime=N upper limit for primes generator [10000] 最大质数发生器数量。默认是10000
```

###  3.3 sysbench  --test=memory help

```bash
$ sysbench  --test=memory help
sysbench 0.4.12:  multi-threaded system evaluation benchmark

memory options:
  --memory-block-size=SIZE  测试时内存块大小。默认是1K
  --memory-total-size=SIZE    传输数据的总大小。默认是100G
  --memory-scope=STRING    内存访问范围{global,local}。默认是global
  --memory-hugetlb=[on|off]  从HugeTLB池内存分配。默认是off
  --memory-oper=STRING     内存操作类型。{read, write, none} 默认是write
  --memory-access-mode=STRING存储器存取方式{seq,rnd} 默认是seq
```


###  3.4 sysbench  --test=threads help

```bash
threads options:
  --thread-yields=N   每个请求产生多少个线程。默认是1000
  --thread-locks=N    每个线程的锁的数量。默认是8
```
###  3.5 sysbench  --test=mutex help

```bash
mutex options:

  --mutex-num=N    数组互斥的总大小。默认是4096
  --mutex-locks=N    每个线程互斥锁的数量。默认是50000
  --mutex-loops=N    内部互斥锁的空循环数量。默认是10000
```
###  3.6 sysbench --test=oltp help

```bash
oltp options:
  --oltp-test-mode=STRING    执行模式{simple,complex(advanced transactional),nontrx(non-transactional),sp}。默认是complex
  --oltp-reconnect-mode=STRING 重新连接模式{session(不使用重新连接。每个线程断开只在测试结束),transaction(在每次事务结束后重新连接),query(在每个SQL语句执行完重新连接),random(对于每个事务随机选择以上重新连接模式)}。默认是session
  --oltp-sp-name=STRING   存储过程的名称。默认为空
  --oltp-read-only=[on|off]  只读模式。Update，delete，insert语句不可执行。默认是off
  --oltp-skip-trx=[on|off]   省略begin/commit语句。默认是off
  --oltp-range-size=N      查询范围。默认是100
  --oltp-point-selects=N          number of point selects [10]
  --oltp-simple-ranges=N          number of simple ranges [1]
  --oltp-sum-ranges=N             number of sum ranges [1]
  --oltp-order-ranges=N           number of ordered ranges [1]
  --oltp-distinct-ranges=N        number of distinct ranges [1]
  --oltp-index-updates=N          number of index update [1]
  --oltp-non-index-updates=N      number of non-index updates [1]
  --oltp-nontrx-mode=STRING   查询类型对于非事务执行模式{select, update_key, update_nokey, insert, delete} [select]
  --oltp-auto-inc=[on|off]      AUTO_INCREMENT是否开启。默认是on
  --oltp-connect-delay=N     在多少微秒后连接数据库。默认是10000
  --oltp-user-delay-min=N    每个请求最短等待时间。单位是ms。默认是0
  --oltp-user-delay-max=N    每个请求最长等待时间。单位是ms。默认是0
  --oltp-table-name=STRING  测试时使用到的表名。默认是sbtest
  --oltp-table-size=N         测试表的记录数。默认是10000
  --oltp-dist-type=STRING    分布的随机数{uniform(均匀分布),Gaussian(高斯分布),special(空间分布)}。默认是special
  --oltp-dist-iter=N    产生数的迭代次数。默认是12
  --oltp-dist-pct=N    值的百分比被视为'special' (for special distribution)。默认是1
  --oltp-dist-res=N    ‘special’的百分比值。默认是75

General database options:
  --db-driver=STRING  数据库类型，指定数据库驱动程序('help' to get list of available drivers)
  --db-ps-mode=STRING 数据库预处理模式{auto, disable} [auto]
Compiled-in database drivers:
    mysql - MySQL driver
mysql options:
  --mysql-host=[LIST,...]       MySQL server host [localhost]
  --mysql-port=N                MySQL server port [3306]
  --mysql-socket=STRING         MySQL socket
  --mysql-user=STRING           MySQL user [sbtest]
  --mysql-password=STRING       MySQL password []
  --mysql-db=STRING             MySQL database name [sbtest]
  --mysql-table-engine=STRING   storage engine to use for the test table {myisam,innodb,bdb,heap,ndbcluster,federated} [innodb]
  --mysql-engine-trx=STRING     whether storage engine used is transactional or not {yes,no,auto} [auto]
  --mysql-ssl=[on|off]          use SSL connections, if available in the client library [off]
  --myisam-max-rows=N           max-rows parameter for MyISAM tables [1000000]
  --mysql-create-options=STRING additional options passed to CREATE TABLE []
```

##  4. 测试
### 4.1 测试 CPU

```bash
$ sysbench --test=cpu --cpu-max-prime=2000 run
WARNING: the --test option is deprecated. You can pass a script name or path on the command line without any options.
sysbench 1.0.11 (using system LuaJIT 2.1.0-beta3)

Running the test with following options:
Number of threads: 1
Initializing random number generator from current time


Prime numbers limit: 2000

Initializing worker threads...

Threads started!

CPU speed:
    events per second:  7290.82

General statistics:
    total time:                          10.0017s
    total number of events:              72939

Latency (ms):
         min:                                  0.09
         avg:                                  0.14
         max:                                 20.14
         95th percentile:                      0.24
         sum:                               9907.92

Threads fairness:
    events (avg/stddev):           72939.0000/0.00
    execution time (avg/stddev):   9.9079/0.00
```

### 4.2  测试线程

```bash
$ sysbench  --test=threads --num-threads=500 --thread-yields=100 --thread-locks=4 run
WARNING: the --test option is deprecated. You can pass a script name or path on the command line without any options.
WARNING: --num-threads is deprecated, use --threads instead
sysbench 1.0.11 (using system LuaJIT 2.1.0-beta3)

Running the test with following options:
Number of threads: 500
Initializing random number generator from current time


Initializing worker threads...

Threads started!


General statistics:
    total time:                          10.1607s
    total number of events:              59317

Latency (ms):
         min:                                  0.05
         avg:                                 84.75
         max:                               2438.83
         95th percentile:                    337.94
         sum:                            5027365.00

Threads fairness:
    events (avg/stddev):           118.6340/83.13
    execution time (avg/stddev):   10.0547/0.05
```

### 4.3 测试 IO
`--num-threads` 开启的线程     `--file-total-size` 总的文件大小

 - prepare阶段，生成需要的测试文件，完成后会在当前目录下生成很多小文件。

```bash
sysbench --test=fileio --num-threads=16 --file-total-size=2G --file-test-mode=rndrw prepare    
```

 - run阶段

```bash
sysbench --test=fileio --num-threads=20 --file-total-size=2G --file-test-mode=rndrw run
```

 - 清理测试时生成的文件

```bash
sysbench --test=fileio --num-threads=20 --file-total-size=2G --file-test-mode=rndrw cleanup
```
###  4.4 测试内存

```bash
$ sysbench --test=memory --memory-block-size=8k --memory-total-size=1G run
WARNING: the --test option is deprecated. You can pass a script name or path on the command line without any options.
sysbench 1.0.11 (using system LuaJIT 2.1.0-beta3)

Running the test with following options:
Number of threads: 1
Initializing random number generator from current time


Running memory speed test with the following options:
  block size: 8KiB
  total size: 1024MiB
  operation: write
  scope: global

Initializing worker threads...

Threads started!

Total operations: 131072 (599444.59 per second)

1024.00 MiB transferred (4683.16 MiB/sec)


General statistics:
    total time:                          0.2148s
    total number of events:              131072

Latency (ms):
         min:                                  0.00
         avg:                                  0.00
         max:                                  4.26
         95th percentile:                      0.00
         sum:                                159.78

Threads fairness:
    events (avg/stddev):           131072.0000/0.00
    execution time (avg/stddev):   0.1598/0.00
```
###  4.5 测试 mutex
略

###  4.6 测试 OLTP

 - prepare阶段，生成需要的测试表

```bash
sysbench --test=oltp --mysql-table-engine=innodb --mysql-host=192.168.X.X --mysql-db=test --oltp-table-size=500000 --mysql-user=root --mysql-password=123456 prepare
```

 - run阶段

```bash
sysbench --num-threads=16 --test=oltp --mysql-table-engine=innodb --mysql-host=192.168.x.x --mysql-db=test --oltp-table-size=500000 --mysql-user=root --mysql-password=123456 run
```

 - 清理测试时生成的测试表

```bash
sysbench --num-threads=16 --test=oltp --mysql-table-engine=innodb --mysql-host=192.168.x.x --mysql-db=test --oltp-table-size=500000 --mysql-user=root --mysql-password=123456 cleanup
```
###  4.7 测试OLTP 结合lua脚本

```bash
$ dpkg -L sysbench
/.
/usr
/usr/bin
/usr/bin/sysbench
/usr/share
/usr/share/doc
/usr/share/doc/sysbench
/usr/share/doc/sysbench/README.md.gz
/usr/share/doc/sysbench/changelog.Debian.gz
/usr/share/doc/sysbench/copyright
/usr/share/doc/sysbench/manual.html
/usr/share/doc-base
/usr/share/doc-base/sysbench-manual
/usr/share/man
/usr/share/man/man1
/usr/share/man/man1/sysbench.1.gz
/usr/share/sysbench
/usr/share/sysbench/bulk_insert.lua
/usr/share/sysbench/oltp_common.lua
/usr/share/sysbench/oltp_delete.lua
/usr/share/sysbench/oltp_insert.lua
/usr/share/sysbench/oltp_point_select.lua
/usr/share/sysbench/oltp_read_only.lua
/usr/share/sysbench/oltp_read_write.lua
/usr/share/sysbench/oltp_update_index.lua
/usr/share/sysbench/oltp_update_non_index.lua
/usr/share/sysbench/oltp_write_only.lua
/usr/share/sysbench/select_random_points.lua
/usr/share/sysbench/select_random_ranges.lua
```
参数

 - `--test=tests/db/oltp.lua` 表示调用 tests/db/oltp.lua 脚本进行 oltp 模式测试
 - `--mysql-table-engine=innodb` 表示选择测试表的存储引擎
 - `--oltp_tables_count=10` 表示会生成 10 个测试表
 - `--oltp-table-size=100000` 表示每个测试表填充数据量为 100000
 - `--rand-init=on` 表示每个测试表都是用随机数据来填充的
 - `--num-threads=8` 表示发起 8个并发连接
 - `--oltp-read-only=off` 表示不要进行只读测试，也就是会采用读写混合模式测试
 - `--report-interval=10` 表示每10秒输出一次测试进度报告
 - `--rand-type=uniform` 表示随机类型为固定模式，其他几个可选随机模式：uniform(固定),gaussian(高斯),special(特定的),pareto(帕累托)
 - `--max-time=120` 表示最大执行时长为 120秒
 - `--max-requests=0` 表示总请求数为 0，因为上面已经定义了总执行时长，所以总请求数可以设定为 0；也可以只设定总请求数，不设定最大执行时长
 - `--percentile=99` 表示设定采样比例，默认是 95%，即丢弃1%的长请求，在剩余的99%里取最大值

如果在本机，也可以使用 –mysql-socket 指定 socket 文件来连接。加载测试数据时长视数据量而定，若过程比较久需要稍加耐心等待。

 - 初始化数据：`prepare`
在本地数据库的dba_test库中，初始化三张表（sbtest1、sbtest2、sbtest3），存储引擎是innodb，每张表50万数据。

```bash
sysbench --test=/usr/share/doc/sysbench/tests/db/oltp.lua --mysql-table-engine=innodb --mysql-host=127.0.0.1 --mysql-db=dba_test --oltp-table-size=500000 --oltp_tables_count=3 --rand-init=on --mysql-user=zjy --mysql-password=zjy  prepare
```

> 如果是本机测试，所以也可以使用--mysql-socket指定socket文件来连接。

 - 测试：run
 模拟对3个表并发OLTP测试，每个表50万行记录，持续压测时间为5分钟。

```bash
 sysbench --test=/usr/share/doc/sysbench/tests/db/oltp.lua --mysql-table-engine=innodb --mysql-host=127.0.0.1 --mysql-db=dba_test --num-threads=8 --oltp-table-size=500000 --oltp_tables_count=3 --oltp-read-only=off --report-interval=10 --rand-type=uniform --max-time=600 --max-requests=0 --percentile=99 --mysql-user=zjy --mysql-password=zjy run
```
结果：

```bash
#每10秒钟报告一次测试结果，tps、每秒读、每秒写、95%以上的响应时长统计
[  10s] threads: 8, tps: 66.60, reads: 943.67, writes: 269.62, response time: 431.72ms (95%), errors: 0.00, reconnects:  0.00
[  20s] threads: 8, tps: 34.30, reads: 480.20, writes: 137.20, response time: 598.28ms (95%), errors: 0.00, reconnects:  0.00
[  30s] threads: 8, tps: 36.60, reads: 512.40, writes: 146.40, response time: 494.87ms (95%), errors: 0.00, reconnects:  0.00

OLTP test statistics:
    queries performed:
        read:                            941248   #读总数
        write:                           268928   #写总数
        other:                           134464   #其他操作总数(SELECT、INSERT、UPDATE、DELETE之外的操作，例如COMMIT等)
        total:                           1344640  #全部总数
    transactions:                        67232  (112.04 per sec.)  #总事务数(每秒事务数)
    read/write requests:                 1210176 (2016.73 per sec.) #读写总数(每秒读写次数)
    other operations:                    134464 (224.08 per sec.) #其他操作总数(每秒其他操作次数)
    ignored errors:                      0      (0.00 per sec.)
    reconnects:                          0      (0.00 per sec.)

General statistics:
    total time:                          600.0698s   #总耗时
    total number of events:              67232  #共发生多少事务数
    total time taken by event execution: 4799.8569s # 所有事务耗时相加(不考虑并行因素)
    response time:
         min:                                  2.09ms   #最小耗时
         avg:                                 71.39ms  #平均耗时
         max:                                839.32ms #最大耗时
         approx.  95 percentile:             309.40ms 超过95%平均耗时

Threads fairness:
    events (avg/stddev):           8404.0000/17.56
    execution time (avg/stddev):   599.9821/0.02
```

 - 清理数据：cleanup

```bash
sysbench --test=/usr/share/doc/sysbench/tests/db/oltp.lua --mysql-table-engine=innodb --mysql-host=127.0.0.1 --mysql-db=dba_test --num-threads=8 --oltp-table-size=500000 --oltp_tables_count=3 --oltp-read-only=off --report-interval=10 --rand-type=uniform --max-time=600 --max-requests=0 --mysql-user=zjy --mysql-password=zjy  cleanup
```

参考：

 - [jyzhou 的sysbench 安装、使用和测试](https://www.cnblogs.com/zhoujinyi/archive/2013/04/19/3029134.html)
 - [https://github.com/akopytov/sysbench.git](https://github.com/akopytov/sysbench.git)
 - [How to Benchmark Your System (CPU, File IO, MySQL) with Sysbench](https://www.howtoforge.com/how-to-benchmark-your-system-cpu-file-io-mysql-with-sysbench)

