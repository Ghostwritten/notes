

## 1. 备份

```bash
$ cat consul_backup.sh
```

```bash
#!/bin/bash

ts=$(date +"%Y_%m_%d_%H_%M")

processName="consul_backup.sh"
processNum=`ps -ef | grep $processName | grep -v grep | wc -l`
if [ $processName gt 3 ];then
echo " $processName already existed!"
exit 1
fi



status_dir=/data/consul/status_dir
kv_dir=/data1/consul/kv_dir

[ -d $status_dir ] || mkdir $status_dir
[ -d $kv_dir ] || mkdir $kv_dir


status_backup() {

consul snapshot save -token=72f41f43-849d-b45f-ee73-3133a3060365 -http-addr=192.168.211.40:8500 $status_dir/consul_state_$i{ts}.snap

}

kv_backup() {

 consul kv export  -token=72f41f43-849d-b45f-ee73-3133a3060365 -http-addr=192.168.211.40:8500 >  $kv_dir/consul_kv_${ts}.json
 if [ $? == 0 ];then
    tar -czPf  $kv_dir/consul_kv_${ts}.tar.gz $kv_dir/consul_kv_${ts}.json
    rm -rf $kv_dir/consul_kv_${ts}.json
 fi

}

kv_restore() {
 date=$1
 tar xPf $kv_dir/consul_kv_${date}.tar.gz -C /
 ls $kv_dir/consul_kv_${date}.json > /dev/null
 if [ $? == 0 ];then

    consul kv import  -token=72f41f43-849d-b45f-ee73-3133a3060365 -http-addr=192.168.211.40:8500 @$kv_dir/consul_kv_${date}.json

 fi

}



status_backup
kv_backup
```

执行:

```bash
[root@control1 consul]# bash consul_backup.sh
[root@control1 consul]# ls kv_dir/
consul_kv_2021_05_18_16_36.tar.gz
[root@control1 consul]# du -sh kv_dir/
21M kv_dir/
[root@control1 consul]# ls kv_dir/
consul_kv_2021_05_18_16_36.json consul_kv_2021_05_18_16_36.tar.gz data1
[root@control1 consul]# du -sh kv_dir/consul_kv_2021_05_18_16_36.json
110M kv_dir/consul_kv_2021_05_18_16_36.json
```

##  2. 还原
状态还原

```bash
# 还原配置文件
tar -xzpf consul_config_20180521145032.tar.gz -C /

# 还原consul服务器状态
consul snapshot restore --http-addr=http://192.168.211.40:8500 -token=b3a9bca3-6e8e-9678-ea35-ccb8fb272d42 consul_state_20180521145032.snap
```
kv存储还原

```bash
consul kv import --http-addr=http://192.168.211.40:8500 -token=b3a9bca3-6e8e-9678-ea35-ccb8fb272d42 @consul_kv_20180521150322.json
```

