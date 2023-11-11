## 下载地址
[http://linuxwireless.sipsolutions.net/en/users/Download/stable/__v210.html](http://linuxwireless.sipsolutions.net/en/users/Download/stable/__v210.html)
[https://github.com/search?q=compat-wireless](https://github.com/search?q=compat-wireless)

## 安装

```bash
wget https://github.com/hunghtbk/compat-wireless-2.6-2010-10/raw/master/compat-wireless-2010-10-10.tar.bz2
apt -y install vim bluetooth
tar -jxvf compat-wireless-2010-10-10.tar.bz2 
cd compat-wireless-2010-10-10
make load
```
报错：

```bash
/sbin/modprobe: invalid option -- 'l'
```
修复方法

```bash
cat > /sbin/modprobe.sh <<EOF
#!/bin/bash

if [[ \$1 == -l ]]
then

if [ -z \$2 ]    
then
find /lib/modules/\$(uname -r) -name '*.ko' | sed -e "s#\\/lib\/modules\/\$(uname -r)\/##g"
else
find /lib/modules/\$(uname -r) -name '*.ko' | sed -e "s#\/lib\/modules\/\$(uname -r)\/##g" | grep \$2
fi

else
/sbin/modprobe \$@
fi
EOF
```


```bash
chmod +x /sbin/modprobe.sh
alias modprobe=/sbin/modprobe.sh
echo "alias modprobe=/sbin/modprobe.sh" >> /etc/bash.bashrc
cp Makefile Makefile.save
sed -i -e 's/^MODPROBE.*/MODPROBE := \/sbin\/modprobe.sh/g' Makefile
```

```bash
$ make load
·····
Starting bluetooth service..
Starting bluetooth (via systemctl): bluetooth.service.
● bluetooth.service - Bluetooth service
     Loaded: loaded (/lib/systemd/system/bluetooth.service; disabled; vendor preset: disabled)
     Active: active (running) since Sun 2020-08-23 11:45:49 CST; 125ms ago
       Docs: man:bluetoothd(8)
   Main PID: 18127 (bluetoothd)
     Status: "Running"
      Tasks: 1 (limit: 810)
     Memory: 1.1M
     CGroup: /system.slice/bluetooth.service
             └─18127 /usr/lib/bluetooth/bluetoothd
········

$ airmon-ng
PHY	Interface	Driver		Chipset
phy14	wlan0		mac80211_hwsim	Software simulator of 802.11 radio(s) for mac80211
phy15	wlan1		mac80211_hwsim	Software simulator of 802.11 radio(s) for mac80211

$ ifconfig wlan0 up
$ ifconfig
ens33: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.211.141  netmask 255.255.255.0  broadcast 192.168.211.255
        inet6 fe80::20c:29ff:fe4e:88bc  prefixlen 64  scopeid 0x20<link>
        ether 00:0c:29:4e:88:bc  txqueuelen 1000  (Ethernet)
        RX packets 253628  bytes 353996977 (337.5 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 127758  bytes 8809364 (8.4 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

wlan0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        ether 02:00:00:00:00:00  txqueuelen 1000  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```
