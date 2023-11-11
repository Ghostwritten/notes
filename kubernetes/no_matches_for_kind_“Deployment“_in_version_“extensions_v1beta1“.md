问题
```bash
$ kubectl apply -f kube-state-metrics-deploy.yaml 
error: unable to recognize "kube-state-metrics-deploy.yaml": no matches for kind "Deployment" in version "extensions/v1beta1"
```
解决方法

确认kubernetes 集群版本
```bash
$ kubectl get node
NAME     STATUS   ROLES                  AGE     VERSION
master   Ready    control-plane,master   5h28m   v1.20.1
node1    Ready    <none>                 5h26m   v1.20.1
node2    Ready    <none>                 5h25m   v1.20.1
```
查看支持apiversion版本，当前是`apps/v1`

```bash
$ kubectl api-resources  |grep deployment
deployments                       deploy       apps/v1                                true         Deployment
```
修改文件`kube-state-metrics-deploy.yaml` 

```bash
apiVersion: apps/v1
```

