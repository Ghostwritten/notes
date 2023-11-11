问题：
k8s 1.11版本
```bash
root@master:~/pods/14# kubectl top pod  nginx-app
Error from server (NotFound): the server could not find the requested resource (get services http:heapster:)
```

解决方法：

```bash
git clone https://github.com/kodekloudhub/kubernetes-metrics-server.git
kubectl create -f kubernetes-metrics-server/
```

