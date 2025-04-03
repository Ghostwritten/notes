#  服务端
略
# 客户端

```bash
proxy_url="http://192.168.21.101:7890"
export no_proxy="192.168.21.2,10.0.0.0/8,192.168.0.0/16,localhost,127.0.0.0/8,.coding.net,.tencentyun.com,.myqcloud.com"
# proxy settings
enable_proxy() {
    export http_proxy="${proxy_url}"
    export https_proxy="${proxy_url}"
    git config --global http.proxy "${proxy_url}"
    git config --global http.proxy "${proxy_url}"
}

disable_proxy() {
    unset http_proxy
    unset https_proxy
    git config --global --unset http.proxy
    git config --global --unset https.proxy
}

disable_proxy
#enable_proxy
```


