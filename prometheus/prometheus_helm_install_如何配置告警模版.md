

## 对接企业微信

### 获取企业id

注册完成之后，通过企业微信官网登录后台管理，在【我的企业】的企业信息里面，获取到Alertmanager服务配置需用到的第一个配置：企业ID
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/52cb0ea501f9c0642c37aabf8f1632c1.png)

## 获取部门id

部门ID
在【通讯录】中，添加一个子部门，用于接收告警信息，后面把人加到该部门，部门内的人就能接收到告警信息了。

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/fbd64b21d4934e226f2b0a4533123f53.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f5b17b5a6a9061e7dd187faf2d940e38.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ce64c28c0faa943b632c9da2aa2091cf.png)

### 获取告警AgentId和Secret
告警AgentId和Secret的获取是需要在企业微信后台，在【应用管理】中，创建应用后才能够获得的。

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e12475285c5b450b98b3c22b060efff8.png)
填写相关消息，最后点击创建应用。

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0a56769b60035e463fc84004c0dfda7c.png)
点击刚才创建好的应用Prometheus，就可以看到AgentId和Secret



![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0f72b98b8f6404bd802c4f3cd30082d5.png)

![](https://i-blog.csdnimg.cn/blog_migrate/b3a7e16490fe9e384f4e06344e18a6db.png)
通过以上步骤，我们就获取到配置Alertmanager的全部信息，包括：企业ID，接收告警的部门ID，AgentId和Secret，共四条消息


alertmanager:
  enabled: true
  config:
    global:
        wechat_api_url: 'https://qyapi.weixin.qq.com/cgi-bin/'    # 企业微信的api_url，无需修改
        wechat_api_corp_id: 'xxxxxxxxxx'      # 企业微信中企业ID
       wechat_api_secret: 'xxxxxxxxxxxxx'      # 企业微信中，Prometheus应用的Secret




## 问题

```bash
Error: INSTALLATION FAILED: failed parsing --set data: key "Firing) 0 -}}{{- range " has no value (cannot end with ,)
helm.go:84: [debug] key "Firing) 0 -}}{{- range " has no value (cannot end with ,)
helm.sh/helm/v3/pkg/strvals.(*parser).key
```


参考：

 - [https://www.cnblogs.com/miaocbin/p/13706164.html](https://www.cnblogs.com/miaocbin/p/13706164.html)
 - [https://cloud.tencent.com/developer/article/1655882](https://cloud.tencent.com/developer/article/1655882)
 - [https://huisebug.github.io/2019/08/27/Prometheus-Operator/](https://huisebug.github.io/2019/08/27/Prometheus-Operator/)
 - [https://www.linuxdevops.cn/2021/04/prometheus-operator-configures-alarm-rules-and-adds-pin/](https://www.linuxdevops.cn/2021/04/prometheus-operator-configures-alarm-rules-and-adds-pin/)
 - [https://blog.csdn.net/boling_cavalry/article/details/130655944](https://blog.csdn.net/boling_cavalry/article/details/130655944)
 - [https://system51.github.io/2021/07/12/Alertmanager/](https://system51.github.io/2021/07/12/Alertmanager/)
 - [https://www.cnblogs.com/winstom/p/11940570.html](https://www.cnblogs.com/winstom/p/11940570.html)
 - [https://github.com/feiyu563/PrometheusAlert](https://github.com/feiyu563/PrometheusAlert)
