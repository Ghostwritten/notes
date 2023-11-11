  [reading https://sum.golang.org/lookup/xxx.com](reading%20https://sum.golang.org/lookup/xxx.com)

  就是这里了，原来是这里在向8.8.8.8发起请求，然后8.8.8.8无法识别。

  问题原因：Go 1.13设置了默认的GOSUMDB=sum.golang.org，它当然是无法识别私有域名xxx.com .
   解决办法：
```bash
 go env -w GOPRIVATE=xxx.como
```
   或者：

```bash
 go env -w GOSUMDB=off
```

