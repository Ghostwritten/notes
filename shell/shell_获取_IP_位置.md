#  Shell 获取 IP 位置
tags: 监控

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ffbcff2b1783036dca54cda1a6e1e169.gif#pic_center)

 - `whereIP.sh`

```bash
#!/usr/bin/env bash

#
# Author: Abhishek Shingane (abhisheks@iitbhilai.ac.in)
# Date: 11 Sep 2020
#

if ! [ -x "$(command -v jq)" ]; then
  echo 'Error: jq is not installed. Install via https://stedolan.github.io/jq/download/'
  exit 1
fi

if [[ $# -ne 1 ]]; then
	echo 'Provide I.P as command line parameter. Usage:  ' $0 ' 15.45.0.1 '
	exit 1
fi
link=$(echo "http://ip-api.com/json/"$1)
data=$(curl $link -s) # -s for slient output

status=$(echo $data | jq '.status' -r)

if [[ $status == "success" ]]; then
	
	city=$(echo $data | jq '.city' -r)
	regionName=$(echo $data | jq '.regionName' -r)
	country=$( echo $data | jq '.country' -r)
	echo $city, $regionName in $country. 
fi 

```
执行：

```bash
$ bash  whereIP.sh 8.8.8.8
Ashburn, Virginia in United States.

$ bash  whereIP.sh 114.114.114.114
Weifang, Shandong in China.
```


