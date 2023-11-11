## 原因
这是由命名空间控制器无法删除的命名空间中仍存在的资源引起的。

此命令（使用kubectl 1.11+）将显示命名空间中保留的资源：

```bash
kubectl api-resources --verbs=list --namespaced -o name  | xargs -n 1 kubectl get --show-kind --ignore-not-found -n <namespace>
```

## 方法
```bash
#!/bin/bash

NAMESPACE=ckad-prep
kubectl proxy &
kubectl get namespace $NAMESPACE -o json |jq '.spec = {"finalizers":[]}' >temp.json
curl -k -H "Content-Type: application/json" -X PUT --data-binary @temp.json 127.0.0.1:8001/api/v1/namespaces/$NAMESPACE/finalize
```



删除 Terminating namespace

```bash
for ns in $(kubectl get ns --field-selector status.phase=Terminating -o jsonpath='{.items[*].metadata.name}')
do
  kubectl get ns $ns -ojson | jq '.spec.finalizers = []' | kubectl replace --raw "/api/v1/namespaces/$ns/finalize" -f -
done

for ns in $(kubectl get ns --field-selector status.phase=Terminating -o jsonpath='{.items[*].metadata.name}')
do
  kubectl get ns $ns -ojson | jq '.metadata.finalizers = []' | kubectl replace --raw "/api/v1/namespaces/$ns/finalize" -f -
done
```

删除 Terminating pod

```bash
kubectl delete pods <pod> --grace-period=0 --force
```
删除pv、pvc

```bash
kubectl patch pv xxx -p '{"metadata":{"finalizers":null}}'
kubectl patch pvc xxx -p '{"metadata":{"finalizers":null}}'
```

