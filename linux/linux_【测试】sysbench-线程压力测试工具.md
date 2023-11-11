[https://www.cnblogs.com/zhoujinyi/archive/2013/04/19/3029134.html](https://www.cnblogs.com/zhoujinyi/archive/2013/04/19/3029134.html)


```bash
 sysbench oltp_read_only \
    --threads=16 \
    --time=30 \
    --report-interval=1 \
    --rand-type=uniform \
    --db-driver=mysql \
    --mysql-user=root \
    --mysql-db=sbtest \
    --mysql-host=bench \
    --mysql-port=3390 \
    --tables=16 \
    --table-size=10000000 \
run > /result/`hostname`-res
```

