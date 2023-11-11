#  Shell 查看帮助



---
Shell 脚本添加查看帮助，让他人更容易看懂理解。

##  1

```bash
help()
{
	header
	echo
	echo 'Usage: cckiller [OPTIONS] [N]'
	echo 'N : number of tcp/udp	connections (default 100)'
	echo
	echo 'OPTIONS:'
	echo "-h | --help: Show	this help screen"
	echo "-k | --kill: Block the offending ip making more than N connections"
	echo '-s | --show: Show The TOP "N" Connections of System Current'
	echo "-b | --banip: Ban The IP or IP subnet like cckiller -b 192.168.1.1"
	echo "-u | --unban: Unban The IP or IP subnet which is in the BlackList of iptables"
	echo
}

if [ $# -lt 2 ]; then
	help
fi
```

##  2

```bash
# Notes:
#  - Please install "jq" package before using this driver.
usage() {
	err "Invalid usage. Usage: "
	err "\t$0 init"
	err "\t$0 mount <mount dir> <json params>"
	err "\t$0 unmount <mount dir>"
	exit 1
}

err() {
	echo -ne $* 1>&2
}

...

if [ $# -lt 2 ]; then
	usage
fi
....

```

