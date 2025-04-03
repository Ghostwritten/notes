问题

- [Kube-Proxy endpoint connection refused](https://github.com/helm/charts/issues/16476)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a1c738c8e6cbfbce37111edb557abfd2.png)


解决方法：

```bash
$ kubectl edit cm/kube-proxy -n kube-system
## Change from
    metricsBindAddress: 127.0.0.1:10249 ### <--- Too secure
## Change to
    metricsBindAddress: 0.0.0.0:10249
$ kubectl delete pod -l k8s-app=kube-proxy -n kube-system
```

原因：

- [Expose kube-proxy metrics on 0.0.0.0 by default](https://github.com/kubernetes/kubernetes/pull/74300)

