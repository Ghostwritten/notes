

## 下载介质

下载 kubevirt 指定版本：[https://github.com/kubevirt/kubevirt/releases](https://github.com/kubevirt/kubevirt/releases)


```bash
export VERSION=$(curl -s https://api.github.com/repos/kubevirt/kubevirt/releases | grep tag_name | grep -v -- '-rc' | sort -r | head -1 | awk -F': ' '{print $2}' | sed 's/,//' | xargs)
echo $VERSION
wget  https://github.com/kubevirt/kubevirt/releases/download/${VERSION}/kubevirt-operator.yaml
wget https://github.com/kubevirt/kubevirt/releases/download/${VERSION}/kubevirt-cr.yaml
```
下载镜像入库

```bash
#!/bin/bash

# kubevirt组件版本
version='v1.1.0-alpha.0'

# 私有镜像仓库
registry='harbor.ghostwritten.com'

# 私有镜像仓库的namespace
namespace=kubevirt

kubevirtRegistry="quay.io/kubevirt"

readonly APPLIST=(
    virt-operator
    virt-api
    virt-controller
    virt-launcher
    virt-handler
)

for app in "${APPLIST[@]}"; do
    # 拉取镜像
    docker pull ${kubevirtRegistry}/${app}:${version}
    # 重命名
    docker tag ${kubevirtRegistry}/${app}:${version} ${registry}/${namespace}/${app}:${version}
    # 推送镜像
    docker push ${registry}/${namespace}/${app}:${version}
done
```


##. 修改 kubevirt-operator.yaml

```bash
        env:
        - name: VIRT_OPERATOR_IMAGE
          value: harbor.ghostwritten.com/kubevirt/virt-operator:v1.1.0-alpha.0
        - name: WATCH_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.annotations['olm.targetNamespaces']
        - name: KUBEVIRT_VERSION
          value: v1.1.0-alpha.0
        image: harbor.ghostwritten.com/kubevirt/virt-operator:v1.1.0-alpha.0
```

## 部署

```bash
kubectl apply -f kubevirt-operator.yaml
kubectl apply -f kubevirt-cr.yaml
kubectl -n kubevirt patch kubevirt kubevirt --type=merge --patch '{"spec":{"configuration":{"developerConfiguration":{"useEmulation":true}}}}'
```
