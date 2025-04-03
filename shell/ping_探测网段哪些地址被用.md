

![](https://i-blog.csdnimg.cn/blog_migrate/403237c46f713fd5280bd9974d5966d2.png)

```bash
#!/bin/bash

# 遍历192.168.3.1到192.168.3.254
for i in {1..254}
do
    ip="192.168.3.$i"
    # 对每个IP地址进行三次ping操作
    if ping -c 3 -W 1 $ip > /dev/null 2>&1
    then
        echo "$ip: yes"
    fi
done
```

```bash
$ sh test.sh
192.168.3.1: yes
192.168.3.95: yes
192.168.3.96: yes
192.168.3.98: yes
192.168.3.111: yes
192.168.3.122: yes
```

