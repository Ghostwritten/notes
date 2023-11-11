#  Linux Commnad wrk HTTP 性能测试工具
tags: 测试




------------
##  1. 简介
wrk是一个 HTTP 性能测试工具，内置了 LuaJIT，方便你根据实际需求，生成所需的请求负载，或者自定义响应的处理方法。

 - github:[https://github.com/wg/wrk](https://github.com/wg/wrk)

##  2. 安装

```bash
git clone https://github.com/wg/wrk
cd wrk
apt-get install build-essential -y
make
sudo cp wrk /usr/local/bin/

#常用方法
wrk -t12 -c400 -d30s http://127.0.0.1:8080/index.html
```

##  3. 参数

```bash
  Options:                                           
    -c, --connections <N>  跟服务器建立并保持的TCP连接数量 
    -d, --duration    <T>  压测时间          
    -t, --threads     <N>  使用多少个线程进行压测，压测时，是有一个主线程来控制我们设置的n个子线程间调度  
                                                    
    -s, --script      <S>  指定Lua脚本路径      
    -H, --header      <H>  为每一个HTTP请求添加HTTP头     
        --latency          在压测结束后，打印延迟统计信息  
        --timeout     <T>  超时时间    
    -v, --version          打印正在使用的wrk的详细版本信                                              

  <N>代表数字参数，支持国际单位 (1k, 1M, 1G)
  <T>代表时间参数，支持时间单位 (2s, 2m, 2h)
```

##  4. 用法
以上是使用8个线程200个连接，对bing首页进行了30秒的压测，并要求在压测结果中输出响应延迟信息。
```bash
root@test1:~/wrk# wrk -t8 -c200 -d30s --latency  http://www.bing.com
Running 30s test @ http://www.bing.com （压测时间30s）

  8 threads and 200 connections （共8个测试线程，200个连接）

  Thread Stats   Avg      Stdev     Max   +/- Stdev
              （平均值） （标准差）（最大值）（正负一个标准差所占比例）
    Latency    46.67ms  215.38ms   1.67s    95.59%
    （延迟）
    Req/Sec     7.91k     1.15k   10.26k    70.77%
    （处理中的请求数）

  Latency Distribution （延迟分布）
     50%    2.93ms
     75%    3.78ms
     90%    4.73ms
     99%    1.35s （99分位的延迟：%99的请求在1.35s以内）
  1790465 requests in 30.01s, 684.08MB read （30.01秒内共处理完成了1790465个请求，读取了684.08MB数据）
Requests/sec:  59658.29 （平均每秒处理完成59658.29个请求）
Transfer/sec:     22.79MB （平均每秒读取数据22.79MB）
```


```bash
# 测试8080端口性能
$ wrk --latency -c 100 -t 2 --timeout 2 http://192.168.0.30:8080/
Running 10s test @ http://192.168.0.30:8080/
  2 threads and 100 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency    43.60ms    6.41ms  56.58ms   97.06%
    Req/Sec     1.15k   120.29     1.92k    88.50%
  Latency Distribution
     50%   44.02ms
     75%   44.33ms
     90%   47.62ms
     99%   48.88ms
  22853 requests in 10.01s, 18.55MB read
Requests/sec:   2283.31
Transfer/sec:      1.85MB
```

参考链接

 - [wrk性能测试（详解）](https://www.cnblogs.com/l199616j/p/12156600.html)
 - [https://github.com/wg/wrk](https://github.com/wg/wrk)
 - [Intelligent benchmark with wrk](https://medium.com/@felipedutratine/intelligent-benchmark-with-wrk-163986c1587f)

