#  Go 代理加速配置

![在这里插入图片描述](https://img-blog.csdnimg.cn/b84b526a44c04d36892f09953a30da49.png)



##  1. Go 版本是 1.13 及以上 （推荐）

```bash
go env -w GO111MODULE=on
go env -w GOPROXY=https://goproxy.io,direct
```

#设置不走 proxy 的私有仓库，多个用逗号相隔（可选）

```bash
go env -w GOPRIVATE=*.corp.example.com
```

设置完上面几个环境变量后，您的 go 命令将从公共代理镜像中快速拉取您所需的依赖代码了。私有库的支持请看这里。

## 2. Go 版本是 1.12 及以下
### 2.1. Bash (Linux or macOS)


#### 2.1.1 第一种方式代理配置
```bash
启用 Go Modules 功能
export GO111MODULE=on
#配置 GOPROXY 环境变量
export GOPROXY=https://goproxy.io
```
#### 2.1.2 第二种方式代理配置
或者，根据文档可以把上面的命令写到.profile或.bash_profile文件中长期生效。

```bash
echo "export GO111MODULE=on" >> ~/.profile
echo "export GOPROXY=https://goproxy.io" >> ~/.profile
source ~/.profile
```
#### 2.1.3 第三种方式代理配置

```bash
# 启用 Go Modules 功能
go env -w GO111MODULE=on

# 配置 GOPROXY 环境变量，以下三选一
# 1. 官方
go env -w  GOPROXY=https://goproxy.io

# 2. 七牛 CDN
go env -w  GOPROXY=https://goproxy.cn

# 3. 阿里云
go env -w GOPROXY=https://mirrors.aliyun.com/goproxy/
```


### 2.2 PowerShell (Windows)

```bash
# 启用 Go Modules 功能
$env:GO111MODULE="on"
# 配置 GOPROXY 环境变量
$env:GOPROXY="https://goproxy.io"
```
1. 右键 我的电脑 -> 属性 -> 高级系统设置 -> 环境变量
2. 在 “[你的用户名]的用户变量” 中点击 ”新建“ 按钮
3. 在 “变量名” 输入框并新增 “GOPROXY”
4. 在对应的 “变量值” 输入框中新增 “https://goproxy.io”
5. 最后点击 “确定” 按钮保存设置

## 3. 测试

```bash
go get  github.com/gin-gonic/gin
go get github.com/astaxie/beego
```
参考：
- 官网文档：[https://goproxy.io/zh/docs/introduction.html](https://goproxy.io/zh/docs/introduction.html)
