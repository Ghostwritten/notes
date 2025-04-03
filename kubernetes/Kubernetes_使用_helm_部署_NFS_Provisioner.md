
![](https://i-blog.csdnimg.cn/blog_migrate/5aced646ff2f33e521bbb1e58b2a18bc.png)



## 1. 介绍
[NFS subdir external provisioner](https://github.com/kubernetes-sigs/nfs-subdir-external-provisioner) 使用现有且已配置的NFS 服务器来支持通过持久卷声明动态配置 Kubernetes 持久卷。持久卷配置为`${namespace}-${pvcName}-${pvName}`.

变量配置：
|Variable	|Value|
|--|--|
|nfs_provisioner_namespace|	nfsstorage
|nfs_provisioner_role	|nfs-provisioner-runner
|nfs_provisioner_serviceaccount	|nfs-provisioner
|nfs_provisioner_name	|hpe.com/nfs
|nfs_provisioner_storage_class_name|	nfs
|nfs_provisioner_server_ip	|hpe2-nfs.am2.cloudra.local
|nfs_provisioner_server_share|	/k8s

> 注意：此存储库是从[https://github.com/kubernetes-incubator/external-storage/tree/master/nfs-client](https://github.com/kubernetes-incubator/external-storage/tree/master/nfs-client)迁移的。作为迁移的一部分：容器镜像名称和存储库已分别更改为`registry.k8s.io/sig-storage`和`nfs-subdir-external-provisioner`。为了保持与早期部署文件的向后兼容性，NFS Client Provisioner 的命名保留为nfs-client-provisioner部署 YAML 中的名称
## 2. 预备条件
- CentOS Linux release 7.9.2009 (Core)
- [kubernetes 集群](https://ghostwritten.blog.csdn.net/article/details/131749277)

```bash
$ kubectl get node
NAME      STATUS   ROLES           AGE    VERSION
master1   Ready    control-plane   275d   v1.25.0
node1     Ready    <none>          275d   v1.25.0
node2     Ready    <none>          275d   v1.25.0

```

## 3. 部署 nfs 

- [linux 配置 NFS 共享服务](https://blog.csdn.net/xixihahalelehehe/article/details/105747174)

```bash
[root@master1 helm]# exportfs -s
/app/nfs/k8snfs  192.168.10.0/24(sync,wdelay,hide,no_subtree_check,sec=sys,rw,secure,no_root_squash,no_all_squash)

```

## 4. 部署 NFS subdir external provisioner

```bash
helm repo add nfs-subdir-external-provisioner https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner/
helm repo update
helm install nfs-subdir-external-provisioner nfs-subdir-external-provisioner/nfs-subdir-external-provisioner --set nfs.server=192.168.10.61 --set nfs.path=/app/nfs/k8snfs -n nfs-provisioner --create-namespace
```
报错：`Error: INSTALLATION FAILED: failed to download "nfs-subdir-external-provisioner/nfs-subdir-external-provisioner"`

忘记配置代理无法拉取 helm charts 和 `registry.k8s.io/sig-storage/nfs-subdir-external-provisioner:v4.0.2`

有两种办法，但都需要找到一个专门配置代理的节点

### 4.1 集群配置 containerd 代理

```bash
$ vim /etc/systemd/system/containerd.service.d/http-proxy.conf
[Service]
Environment="HTTP_PROXY=http://192.168.10.105:7890"
Environment="HTTPS_PROXY=http://192.168.10.105:7890"
Environment="NO_PROXY=localhost"

#重启
$ systemctl restart containerd.service
```
这样镜像的问题就解决了。下面解决拉取  `helm charts`的问题

再执行部署 debug ,发现拉取的 `helm charts` 的版本

```bash
$ helm --debug install nfs-subdir-external-provisioner nfs-subdir-external-provisioner/nfs-subdir-external-provisioner --set nfs.server=192.168.10.61 --set nfs.path=/app/nfs/k8snfs -n nfs-provisioner --create-namespace
Error: INSTALLATION FAILED: Get "https://objects.githubusercontent.com/github-production-release-asset-2e65be/250135810/33156d2f-3fef-4b00-bf34-1817d30653bc?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIWNJYAX4CSVEH53A%2F20230716%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20230716T153116Z&X-Amz-Expires=300&X-Amz-Signature=7219da0622fe22795d526f742064ee0da00a5821c37a5e1fe1bb0eb6b046e3c0&X-Amz-SignedHeaders=host&actor_id=0&key_id=0&repo_id=250135810&response-content-disposition=attachment%3B%20filename%3Dnfs-subdir-external-provisioner-4.0.18.tgz&response-content-type=application%2Foctet-stream": read tcp 192.168.10.28:46032->192.168.10.105:7890: read: connection reset by peer
helm.go:84: [debug] Get "https://objects.githubusercontent.com/github-production-release-asset-2e65be/250135810/33156d2f-3fef-4b00-bf34-1817d30653bc?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIWNJYAX4CSVEH53A%2F20230716%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20230716T153116Z&X-Amz-Expires=300&X-Amz-Signature=7219da0622fe22795d526f742064ee0da00a5821c37a5e1fe1bb0eb6b046e3c0&X-Amz-SignedHeaders=host&actor_id=0&key_id=0&repo_id=250135810&response-content-disposition=attachment%3B%20filename%3Dnfs-subdir-external-provisioner-4.0.18.tgz&response-content-type=application%2Foctet-stream": read tcp 192.168.10.28:46032->192.168.10.105:7890: read: connection reset by peer
```
手动去下载 [nfs-subdir-external-provisioner-4.0.18.tgz](https://artifacthub.io/packages/helm/nfs-subdir-external-provisioner/nfs-subdir-external-provisioner)

![](https://i-blog.csdnimg.cn/blog_migrate/33a4bc49f7307d833f7d0b1dae952ce6.png)
再指定本地 helm charts 包执行部署

```bash
helm install nfs-subdir-external-provisioner nfs-subdir-external-provisioner-4.0.18.tgz --set nfs.server=192.168.10.61 --set nfs.path=/app/nfs/k8snfs -n nfs-provisioner --create-namespace
```

### 4.2 配置代理堡垒机通过 kubeconfig 部署

拉取 `registry.k8s.io/sig-storage/nfs-subdir-external-provisioner:v4.0.2`
```bash
$ podman pull registry.k8s.io/sig-storage/nfs-subdir-external-provisioner:v4.0.2
Trying to pull registry.k8s.io/sig-storage/nfs-subdir-external-provisioner:v4.0.2...
Getting image source signatures
Copying blob 528677575c0b done  
Copying blob 60775238382e done  
Copying config 932b0bface done  
Writing manifest to image destination
Storing signatures
932b0bface75b80e713245d7c2ce8c44b7e127c075bd2d27281a16677c8efef3
$ podman save -o nfs-subdir-external-provisioner-v4.0.2.tar  registry.k8s.io/sig-storage/nfs-subdir-external-provisioner:v4.0.2
Getting image source signatures
Copying blob 1a5ede0c966b done  
Copying blob ad321585b8f5 done  
Copying config 932b0bface done  
Writing manifest to image destination
Storing signatures
$ scp nfs-subdir-external-provisioner-v4.0.2.tar root@192.168.10.62:/root
$ scp nfs-subdir-external-provisioner-v4.0.2.tar root@192.168.10.63:/root
```
配置 `kubeconfig`

```bash
$ mkdir kubeconfig
$ vim kubeconfig/61cluster.yaml
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUM2VENDQWRHZ0F3SUJBZ0lCQURBTkJna3Foa2lHOXcwQkFRc0ZBREFWTVJNd0VRWURWUVFERXdwcmRXSmwKY201bGRHVnpNQ0FYRFRJeU1UQXhOREE1TURFeE9Gb1lEekl4TWpJd09USXdNRGt3TVRFNFdqQVZNUk13RVFZRApWUVFERXdwcmRXSmxjbTVsZEdWek1JSUJJakFOQmdrcWhraUc5dzBCQVFFRkFBT0NBUThBTUlJQkNnS0NBUUVBCnR1WTAvblE1OTZXVm5pZFFOdmJpWFNRczJjSVh5WCthZVBMZ0ptUXVpb0pjeGlyQ2dxdStLT0hQTWcwamgra1MKT0RqWS80K3hvZlpjakhydFRDYlg0U1dpUUFqK0diSTJVdmd1ei91U29JVHhhZzNId2JCVnk0REZrUjdpSVUxOQpVVWd0Yy9VYlB6L2I0aGJnT3prYkcyVGo0eDF1b3U4aTErTUVyZnRZRmtyTjJ1bzNTU1RaMVhZejB5d08xbzZvCkxiYktudDB3TUthUmFqKzRKS3lPRkd2dHVMODhjTXRYSXN3KzZ5QndqNWVlYUFnZXVRbUZYcHZ3M1BNRWt3djIKWFN6RTVMRy9SUUhaWTNTeGpWdUNPVXU5SllvNFVWK2RwRUdncUdmRXJDOHNvWHAvcG9PSkhERVhKNFlwdnFDOApJSnErRldaUXE1VEhKYy8rMUFoenhRSURBUUFCbzBJd1FEQU9CZ05WSFE4QkFmOEVCQU1DQXFRd0R3WURWUjBUCkFRSC9CQVV3QXdFQi96QWRCZ05WSFE0RUZnUVVMb0ZPcDZ1cFBHVldUQ1N3WWlpRkpqZkowOWd3RFFZSktvWkkKaHZjTkFRRUxCUUFEZ2dFQkFFV2orRmcxTFNkSnRSM1FFdFBnKzdHbEJHNnJsZktCS3U2Q041TnJGeEN5Y3UwMwpNNG1JUEg3VXREYUMyRHNtQVNUSWwrYXMzMkUrZzBHWXZDK0VWK0F4dG40RktYaHhVSkJ2Smw3RFFsY2VWQTEyCjk0bDExYUk1VE5IOGN5WDVsQ3draXRRMks4ekxTdUgySFlKeG15cTVVK092UVBaS3J4ekN3NFBCdk5Rem1lSFMKR0VuKzdVUjFFamZQaGZ5UTZIdGh5VmZ2MWNtL283L2tCWkJ4OGJmQWt4T0drUnR4eHo4V1JVVTNOUkwwbUt4YwpIc2xPMm43a09BZnB4U3Jya2w3UFRXd0doSEN1VGtxRUdaOEsycW9wK285ajQyS3U5eldqUUlaMjJLcytLMXk2CjFmd3h0Zit2c2hFaFZURGZSU2ZoTDYyUEh3RnAxQklZTFZoVUhJcz0KLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo=
    server: https://192.168.10.61:6443
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    user: kubernetes-admin
  name: kubernetes-admin@kubernetes
current-context: kubernetes-admin@kubernetes
kind: Config
preferences: {}
users:
- name: kubernetes-admin
  user:
    client-certificate-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURGVENDQWYyZ0F3SUJBZ0lJWVNHaHV4c1poUWt3RFFZSktvWklodmNOQVFFTEJRQXdGVEVUTUJFR0ExVUUKQXhNS2EzVmlaWEp1WlhSbGN6QWdGdzB5TWpFd01UUXdPVEF4TVRoYUdBOHlNVEl5TURreU1EQTVNREV5TmxvdwpOREVYTUJVR0ExVUVDaE1PYzNsemRHVnRPbTFoYzNSbGNuTXhHVEFYQmdOVkJBTVRFR3QxWW1WeWJtVjBaWE10CllXUnRhVzR3Z2dFaU1BMEdDU3FHU0liM0RRRUJBUVVBQTRJQkR3QXdnZ0VLQW9JQkFRQ25KeHVSd0FERGw5RkMKMGRtSWVmV05hcE9DL2R1OXUwWWIwTXA5Nzh2eW5IcFJXMEI4QWlTTitkOHZCelNwMi9GdmVZeGlPSUpwbDlTVgpwcTdtSXM0T1A3cXN5Znc0TTBXKzM5c2dEditGYlJ1OUVMUlV6cXg1T1RwZVlDZVRnaFplQXRSU0dOamhKS2N0Cmd1SzA5OHJoNkpSWnZhUk1TYkYzK21GZ0RrbHNpL0Z4c2s1Uzl1Rk9Zb3lxTWdTUjdGTjFlOHVRSmxwU09Zem8KQlBWc3NsQ2FUTUNoQ2RrVnFteThiRVVtdzFvRzhhTGwrYXRuaW1QdEFXaWNzMGZjMGV0Zm9MRUpDcno4Wlo4UApBSnRackVHaDcxM0d0czdGblpXNnJ6RFppc3Z0Zml1WGFyanFQd2Z3a0ZBekJhYlRiYUF1NlJIdWloSWZSZWJxCjB2djR0c2tCQWdNQkFBR2pTREJHTUE0R0ExVWREd0VCL3dRRUF3SUZvREFUQmdOVkhTVUVEREFLQmdnckJnRUYKQlFjREFqQWZCZ05WSFNNRUdEQVdnQlF1Z1U2bnE2azhaVlpNSkxCaUtJVW1OOG5UMkRBTkJna3Foa2lHOXcwQgpBUXNGQUFPQ0FRRUF0akk4c2c3KzlORUNRaStwdDZ5bWVtWjZqOG5SQjFnbm5aU2dGN21GYk03NXdQSUQ0NDJYCkhENnIwOEF6bDZGei9sZEtxbkN0cDJ2QnJWQmxVaWl6Ry9naWVWQTVKa3NIVEtveFFpV1llWEwwYmxsVDA2RDcKV240V1BTKzUvcGZMWktmd25jL20xR0owVWtQQUJHQVdSVTFJSi9kK0dJUlFtNTJTck9VYktLUTIzbHhGa2xqMwpYaDYveEg0eVRUeGsxRjVEVUhwcnFSTVdDTXZRYkRkM0pUaEpvdWNpZWRtcCs1YWV0ZStQaGZLSUtCT1JoMC9OCnIyTWpCZjNNaENyMUMwK0dydGMyeC80eC9PejRwbGRGNmQ1a2c3NmZvOCtiTW1ISmVyaVV6MXZKSkU0bFYxUDEKK21wN1E5Y1BGUVBJdkpNakRBVXdBUkRGcHNNTEhYQ0FYZz09Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K
    client-key-data: LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQpNSUlFb3dJQkFBS0NBUUVBcHljYmtjQUF3NWZSUXRIWmlIbjFqV3FUZ3YzYnZidEdHOURLZmUvTDhweDZVVnRBCmZBSWtqZm5mTHdjMHFkdnhiM21NWWppQ2FaZlVsYWF1NWlMT0RqKzZyTW44T0RORnZ0L2JJQTcvaFcwYnZSQzAKVk02c2VUazZYbUFuazRJV1hnTFVVaGpZNFNTbkxZTGl0UGZLNGVpVVdiMmtURW14ZC9waFlBNUpiSXZ4Y2JKTwpVdmJoVG1LTXFqSUVrZXhUZFh2TGtDWmFVam1NNkFUMWJMSlFta3pBb1FuWkZhcHN2R3hGSnNOYUJ2R2k1Zm1yClo0cGo3UUZvbkxOSDNOSHJYNkN4Q1FxOC9HV2ZEd0NiV2F4Qm9lOWR4cmJPeFoyVnVxOHcyWXJMN1g0cmwycTQKNmo4SDhKQlFNd1dtMDIyZ0x1a1I3b29TSDBYbTZ0TDcrTGJKQVFJREFRQUJBb0lCQUNoY29DS2NtMUtmaVM4NgpYdTIralZXZGc0c2c0M3U0Q2VEVGxPRytFcUE5dXFlRWdsaXZaOFpFck9pOU03RkVZOU5JSldiZVFGZGhDenNyCnFaWDJsNDBIUkh0T3RyR1haK01FU1BRL3l1R2NEQk9tUWZVc2hxY3E4M1l3ZjczMXJwTDYyZXdOQmVtdm9SS3oKUlN6dm5MVGFKV0JhRTU4OE9EZEJaVnY5ZHl0WFoxSkVqWHZTVUowaWY4bWZvMUlxNUdBa1FLZWZuMlVLcTRROApYYzJTTkd5QTZxUThGNGd0ZWJ1WGI2QVFLdko4K05KRlI1b2ppNG9hWVlkcE5yR0MzUnJ5VHVSc29ZNFIxRko5ClA5WjcwZGtCcnExcDlNOVA2aDFxWVlaT1FISDdNRklaaFBra3dHNllVLzdzRVBZS2h1R29LVVNJR0FsUXU1czIKOGFtM0toVUNnWUVBM1AwaDRRd0xoeXFXVDBnRC9CcFRPR21iWjd2enA2Z3B3NTNhQXppWDE1UVRHcjdwaWF5RApFSlI1c01vUkF5ckthdUVVZjR1MzkyeGc0NlY4eVJNN3ZlVzZzZ2ZDSnUva3N6Nkxqa3FWRXBXUktycWVQRzhKClIwZXQ2TXRIaExxRHBDSytIdTJIYXhtbWdzMzB6Wk9EQ0Vma2dOSGY3cmM0ZnlxY2pETEpOZHNDZ1lFQXdhSisKRmhQSmpTdTVBYlJ1d2dVUDJnd0REM2ZiQ09peW5VSHpweGhUMWhRcUNPNm14dE1VaUE4bFJraTgxb1NLVEN2eAoxd1VpcnMwYzVNVFRiUS9kekpSVEtTSlRGZFhWNUdxUXppclc3SE5meGcvS1RkTVUyNDRvZG9WY2E4M0Q5WjJ6CmxybVNQQkEvaS9SOVVSNTRnODdFbHBuVi9Cc21wSDcrbUlkQzZWTUNnWUI3dGZsUlVyemhYaVhuSEJtZTk5MisKcHVBb29qODBqQjlWTXZqbzlMV01LWWpJWURlOHFxWjBrYW5PSGxDSHhWeXJtSFV4TWJZNi9LRUF6NU9idlBpawp4Z1pOdzZvY3dnNzFpUDMzR2lsNXplRUdXcEphb280L0tSRmlVT29vazRFK1VYUzlPNXVqaVNoOThXNHA1M3BqCkdGd0RBWHFxMkViNGFaSlpxZFNhSVFLQmdRQ3A2TTdneW40cVhQcGJUNXQ4cm5wcForN3JqTTFyZE56K2R0ZTUKZ1BSWHZwdmYrS0hwaDJEVnZ3eURMdUpkRGpKWWdwc1VoVklZdHEwcTVMZHRWT1hZVlRMZnZsblBxREttMndlegprUTNFcjd5VGpGbUZqcm9YcWhkQllPWm5Sa2cwWnl3bUR6SU5lR2g2ZzQvUE5ZQ2trRFFhdm1SeGN0V210RFR0ClhJdFBOd0tCZ0RkTnlRRU5pNmptd0tEaDBMeUNJTXBlWVA4TEYyKzVGSHZPWExBSFBuSzFEb2I1djMrMjFZTVoKTmtibGNJNzNBd2RiRnJpRjhqbVBxYXZmdUowNlA4UUJZVGVEbGhiSjZBZW1nWG1kVlRaL2IwTnV1ZktiNFdvVgo0eHA3TUJYa0NYNTUxWVB6djloc2M2RTZkYm5KRHJCajV4M3RsbWdyV2ZmL00weUtTOEF4Ci0tLS0tRU5EIFJTQSBQUklWQVRFIEtFWS0tLS0tCg==
```
测试

```bash
$ kubectl --kubeconfig kubeconfig/61cluster.yaml get node
NAME      STATUS   ROLES           AGE    VERSION
master1   Ready    control-plane   275d   v1.25.0
node1     Ready    <none>          275d   v1.25.0
node2     Ready    <none>          275d   v1.25.0
```
部署 

```bash
$ helm install --kubeconfig kubeconfig/61cluster.yaml  nfs-subdir-external-provisioner nfs-subdir-external-provisioner/nfs-subdir-external-provisioner --set nfs.server=192.168.10.61 --set nfs.path=/app/nfs/k8snfs -n nfs-provisioner --create-namespace
NAME: nfs-subdir-external-provisioner
LAST DEPLOYED: Sun Jul 16 22:51:28 2023
NAMESPACE: nfs-provisioner
STATUS: deployed
REVISION: 1
TEST SUITE: None

$ kubectl --kubeconfig kubeconfig/61cluster.yaml get all -n nfs-provisioner  -owide
NAME                                                   READY   STATUS    RESTARTS   AGE   IP               NODE    NOMINATED NODE   READINESS GATES
pod/nfs-subdir-external-provisioner-688456c5d9-f5xkt   1/1     Running   0          39m   100.108.11.220   node2   <none>           <none>

NAME                                              READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS                        IMAGES                                                               SELECTOR
deployment.apps/nfs-subdir-external-provisioner   1/1     1            1           39m   nfs-subdir-external-provisioner   registry.k8s.io/sig-storage/nfs-subdir-external-provisioner:v4.0.2   app=nfs-subdir-external-provisioner,release=nfs-subdir-external-provisioner

NAME                                                         DESIRED   CURRENT   READY   AGE   CONTAINERS                        IMAGES                                                               SELECTOR
replicaset.apps/nfs-subdir-external-provisioner-688456c5d9   1         1         1       39m   nfs-subdir-external-provisioner   registry.k8s.io/sig-storage/nfs-subdir-external-provisioner:v4.0.2   app=nfs-subdir-external-provisioner,pod-template-hash=688456c5d9,release=nfs-subdir-external-provisioner


$ kubectl --kubeconfig kubeconfig/61cluster.yaml get sc
NAME         PROVISIONER                                     RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
nfs-client   cluster.local/nfs-subdir-external-provisioner   Delete          Immediate           true                   37m

```

遇到这种镜像无法拉取的 helm charts ，我们可以[定制属于自己的 helm charts](https://blog.csdn.net/xixihahalelehehe/article/details/129247543)，方便日常测试使用。

## 部署 MinIO

### 添加仓库
```bash
kubectl create ns minio
helm repo add minio https://helm.min.io/
helm repo update
helm search repo minio/minio
```
### 修改可配置项

```bash
helm show values minio/minio > values.yaml
```
修改内容：

```bash
accessKey: 'minio'
secretKey: 'minio123'
persistence:
  enabled: true
  storageCalss: 'nfs-client'
  VolumeName: ''
  accessMode: ReadWriteOnce
  size: 5Gi

service:
  type: ClusterIP
  clusterIP: ~
  port: 9000
  # nodePort: 32000

resources:
  requests:
    memory: 128M
```


如果你想知道最终生成的模版，可以使用 helm template 命令。

```bash
helm template -f values.yaml --namespace minio minio/minio | tee -a  minio.yaml
```
输出：
```bash
---
# Source: minio/templates/post-install-prometheus-metrics-serviceaccount.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: release-name-minio-update-prometheus-secret
  labels:
    app: minio-update-prometheus-secret
    chart: minio-8.0.10
    release: release-name
    heritage: Helm
---
# Source: minio/templates/serviceaccount.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: "release-name-minio"
  namespace: "minio"
  labels:
    app: minio
    chart: minio-8.0.10
    release: "release-name"
---
# Source: minio/templates/secrets.yaml
apiVersion: v1
kind: Secret
metadata:
  name: release-name-minio
  labels:
    app: minio
    chart: minio-8.0.10
    release: release-name
    heritage: Helm
type: Opaque
data:
  accesskey: "bWluaW8="
  secretkey: "bWluaW8xMjM="
---
# Source: minio/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: release-name-minio
  labels:
    app: minio
    chart: minio-8.0.10
    release: release-name
    heritage: Helm
data:
  initialize: |-
    #!/bin/sh
    set -e ; # Have script exit in the event of a failed command.
    MC_CONFIG_DIR="/etc/minio/mc/"
    MC="/usr/bin/mc --insecure --config-dir ${MC_CONFIG_DIR}"
    
    # connectToMinio
    # Use a check-sleep-check loop to wait for Minio service to be available
    connectToMinio() {
      SCHEME=$1
      ATTEMPTS=0 ; LIMIT=29 ; # Allow 30 attempts
      set -e ; # fail if we can't read the keys.
      ACCESS=$(cat /config/accesskey) ; SECRET=$(cat /config/secretkey) ;
      set +e ; # The connections to minio are allowed to fail.
      echo "Connecting to Minio server: $SCHEME://$MINIO_ENDPOINT:$MINIO_PORT" ;
      MC_COMMAND="${MC} config host add myminio $SCHEME://$MINIO_ENDPOINT:$MINIO_PORT $ACCESS $SECRET" ;
      $MC_COMMAND ;
      STATUS=$? ;
      until [ $STATUS = 0 ]
      do
        ATTEMPTS=`expr $ATTEMPTS + 1` ;
        echo \"Failed attempts: $ATTEMPTS\" ;
        if [ $ATTEMPTS -gt $LIMIT ]; then
          exit 1 ;
        fi ;
        sleep 2 ; # 1 second intervals between attempts
        $MC_COMMAND ;
        STATUS=$? ;
      done ;
      set -e ; # reset `e` as active
      return 0
    }
    
    # checkBucketExists ($bucket)
    # Check if the bucket exists, by using the exit code of `mc ls`
    checkBucketExists() {
      BUCKET=$1
      CMD=$(${MC} ls myminio/$BUCKET > /dev/null 2>&1)
      return $?
    }
    
    # createBucket ($bucket, $policy, $purge)
    # Ensure bucket exists, purging if asked to
    createBucket() {
      BUCKET=$1
      POLICY=$2
      PURGE=$3
      VERSIONING=$4
    
      # Purge the bucket, if set & exists
      # Since PURGE is user input, check explicitly for `true`
      if [ $PURGE = true ]; then
        if checkBucketExists $BUCKET ; then
          echo "Purging bucket '$BUCKET'."
          set +e ; # don't exit if this fails
          ${MC} rm -r --force myminio/$BUCKET
          set -e ; # reset `e` as active
        else
          echo "Bucket '$BUCKET' does not exist, skipping purge."
        fi
      fi
    
      # Create the bucket if it does not exist
      if ! checkBucketExists $BUCKET ; then
        echo "Creating bucket '$BUCKET'"
        ${MC} mb myminio/$BUCKET
      else
        echo "Bucket '$BUCKET' already exists."
      fi
    
    
      # set versioning for bucket
      if [ ! -z $VERSIONING ] ; then
        if [ $VERSIONING = true ] ; then
            echo "Enabling versioning for '$BUCKET'"
            ${MC} version enable myminio/$BUCKET
        elif [ $VERSIONING = false ] ; then
            echo "Suspending versioning for '$BUCKET'"
            ${MC} version suspend myminio/$BUCKET
        fi
      else
          echo "Bucket '$BUCKET' versioning unchanged."
      fi
    
      # At this point, the bucket should exist, skip checking for existence
      # Set policy on the bucket
      echo "Setting policy of bucket '$BUCKET' to '$POLICY'."
      ${MC} policy set $POLICY myminio/$BUCKET
    }
    
    # Try connecting to Minio instance
    scheme=http
    connectToMinio $scheme
---
# Source: minio/templates/pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: release-name-minio
  labels:
    app: minio
    chart: minio-8.0.10
    release: release-name
    heritage: Helm
spec:
  accessModes:
    - "ReadWriteOnce"
  resources:
    requests:
      storage: "1Gi"
  storageClassName: "nfs-client"
---
# Source: minio/templates/post-install-prometheus-metrics-role.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: release-name-minio-update-prometheus-secret
  labels:
    app: minio-update-prometheus-secret
    chart: minio-8.0.10
    release: release-name
    heritage: Helm
rules:
  - apiGroups:
      - ""
    resources:
      - secrets
    verbs:
      - get
      - create
      - update
      - patch
    resourceNames:
      - release-name-minio-prometheus
  - apiGroups:
      - ""
    resources:
      - secrets
    verbs:
      - create
  - apiGroups:
      - monitoring.coreos.com
    resources:
      - servicemonitors
    verbs:
      - get
    resourceNames:
      - release-name-minio
---
# Source: minio/templates/post-install-prometheus-metrics-rolebinding.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: release-name-minio-update-prometheus-secret
  labels:
    app: minio-update-prometheus-secret
    chart: minio-8.0.10
    release: release-name
    heritage: Helm
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: release-name-minio-update-prometheus-secret
subjects:
  - kind: ServiceAccount
    name: release-name-minio-update-prometheus-secret
    namespace: "minio"
---
# Source: minio/templates/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: release-name-minio
  labels:
    app: minio
    chart: minio-8.0.10
    release: release-name
    heritage: Helm
spec:
  type: NodePort
  ports:
    - name: http
      port: 9000
      protocol: TCP
      nodePort: 32000
  selector:
    app: minio
    release: release-name
---
# Source: minio/templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: release-name-minio
  labels:
    app: minio
    chart: minio-8.0.10
    release: release-name
    heritage: Helm
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 100%
      maxUnavailable: 0
  selector:
    matchLabels:
      app: minio
      release: release-name
  template:
    metadata:
      name: release-name-minio
      labels:
        app: minio
        release: release-name
      annotations:
        checksum/secrets: f48e042461f5cd95fe36906895a8518c7f1592bd568c0caa8ffeeb803c36d4a4
        checksum/config: 9ec705e3000d8e1f256b822bee35dc238f149dbb09229548a99c6409154a12b8
    spec:
      serviceAccountName: "release-name-minio"
      securityContext:
        runAsUser: 1000
        runAsGroup: 1000
        fsGroup: 1000
      containers:
        - name: minio
          image: "minio/minio:RELEASE.2021-02-14T04-01-33Z"
          imagePullPolicy: IfNotPresent
          command:
            - "/bin/sh"
            - "-ce"
            - "/usr/bin/docker-entrypoint.sh minio -S /etc/minio/certs/ server /export"
          volumeMounts:
            - name: export
              mountPath: /export            
          ports:
            - name: http
              containerPort: 9000
          env:
            - name: MINIO_ACCESS_KEY
              valueFrom:
                secretKeyRef:
                  name: release-name-minio
                  key: accesskey
            - name: MINIO_SECRET_KEY
              valueFrom:
                secretKeyRef:
                  name: release-name-minio
                  key: secretkey
          resources:
            requests:
              memory: 1Gi      
      volumes:
        - name: export
          persistentVolumeClaim:
            claimName: release-name-minio
        - name: minio-user
          secret:
            secretName: release-name-minio

```


创建 MinIO

```bash
helm install -f values.yaml minio  minio/minio -n minio
```
输出：

```bash
NAME: minio
LAST DEPLOYED: Wed Jul 19 10:56:23 2023
NAMESPACE: minio
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
Minio can be accessed via port 9000 on the following DNS name from within your cluster:
minio.minio.svc.cluster.local

To access Minio from localhost, run the below commands:

  1. export POD_NAME=$(kubectl get pods --namespace minio -l "release=minio" -o jsonpath="{.items[0].metadata.name}")

  2. kubectl port-forward $POD_NAME 9000 --namespace minio

Read more about port forwarding here: http://kubernetes.io/docs/user-guide/kubectl/kubectl_port-forward/

You can now access Minio server on http://localhost:9000. Follow the below steps to connect to Minio server with mc client:

  1. Download the Minio mc client - https://docs.minio.io/docs/minio-client-quickstart-guide

  2. Get the ACCESS_KEY=$(kubectl get secret minio -o jsonpath="{.data.accesskey}" | base64 --decode) and the SECRET_KEY=$(kubectl get secret minio -o jsonpath="{.data.secretkey}" | base64 --decode)

  3. mc alias set minio-local http://localhost:9000 "$ACCESS_KEY" "$SECRET_KEY" --api s3v4

  4. mc ls minio-local

Alternately, you can use your browser or the Minio SDK to access the server - https://docs.minio.io/categories/17

```
查看 minio 状态

```bash
$ kubectl get pod -n minio
NAME                     READY   STATUS    RESTARTS   AGE
minio-66f8b9444b-lml5f   1/1     Running   0          62s
[root@master1 helm]# kubectl get all -n minio
NAME                         READY   STATUS    RESTARTS   AGE
pod/minio-66f8b9444b-lml5f   1/1     Running   0          73s

NAME            TYPE       CLUSTER-IP    EXTERNAL-IP   PORT(S)          AGE
service/minio   NodePort   10.96.0.232   <none>        9000:32000/TCP   73s

NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/minio   1/1     1            1           73s

NAME                               DESIRED   CURRENT   READY   AGE
replicaset.apps/minio-66f8b9444b   1         1         1       73s


$ kubectl get pv,pvc,sc -n minio
NAME                                                        CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM         STORAGECLASS   REASON   AGE
persistentvolume/pvc-667a9c76-7d14-484c-aeeb-6e07cffd2c10   1Gi        RWO            Delete           Bound    minio/minio   nfs-client              2m20s

NAME                          STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
persistentvolumeclaim/minio   Bound    pvc-667a9c76-7d14-484c-aeeb-6e07cffd2c10   1Gi        RWO            nfs-client     2m20s

NAME                                     PROVISIONER                                     RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
storageclass.storage.k8s.io/nfs-client   cluster.local/nfs-subdir-external-provisioner   Delete          Immediate           true                   2d12h

```

## 访问
###  nodepot 

界面访问：http://192.168.10.61:32000
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/9d3093aa80168e07be6dcfa41f4e3a93.png)
![](https://i-blog.csdnimg.cn/blog_migrate/aca06e82f28ef65d28e150f9a475768e.png)

### ingress
修改 `values.yaml` 的`service`

```bash
service:
  type: ClusterIP
  clusterIP: ~
  port: 9000

```
更新

```bash
$ helm upgrade -f values.yaml minio  minio/minio -n minio
Release "minio" has been upgraded. Happy Helming!
NAME: minio
LAST DEPLOYED: Wed Jul 19 11:49:22 2023
NAMESPACE: minio
STATUS: deployed
REVISION: 2
TEST SUITE: None
NOTES:
Minio can be accessed via port 9000 on the following DNS name from within your cluster:
minio.minio.svc.cluster.local

$ kubectl get all -n minio
NAME                         READY   STATUS    RESTARTS   AGE
pod/minio-66f8b9444b-lml5f   1/1     Running   0          53m

NAME            TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)    AGE
service/minio   ClusterIP   10.96.0.232   <none>        9000/TCP   53m

NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/minio   1/1     1            1           53m

NAME                               DESIRED   CURRENT   READY   AGE
replicaset.apps/minio-66f8b9444b   1         1         1       53m
```
service 已经由 `nodePort` 类型改为 `ClusterIP`。

接下来，我们需要配置证书和域名，你需要在集群内  [ 部署 cert-manager ](https://blog.csdn.net/xixihahalelehehe/article/details/129986416)

查看 minio的 secret tls 证书

```bash
$ kubectl get secret -n minio
NAME                          TYPE                 DATA   AGE
minio                         Opaque               2      58m
minio-letsencrypt-tls-fn4vt   Opaque               1      2m47s
```
查看已经创建好的 `cluster-issuer`名称

```bash
$ kubectl get ClusterIssuer
NAME               READY   AGE
letsencrypt-prod   True    33m

```

```bash
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: minio
  namespace: minio
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod # 配置自动生成 https 证书
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  tls:
    - hosts:
        - 'minio.demo.com'
      secretName: minio-letsencrypt-tls
  rules:
    - host: minio.demo.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: minio
                port:
                  number: 9000
```
创建

```bash
kubectl apply -f ingress.yaml
```

域名解析：
- linux 在 `/etc/hosts` 添加 `192.168.10.61  minio.demo.com`
- windows 在 `C:\Windows\System32\drivers\etc\hosts` 添加  `192.168.10.61 minio.demo.com`

参考：

- [Deploying the NFS provisioner for Kubernetes](https://hewlettpackard.github.io/Docker-SimpliVity/storage/nfs-provisioner.html#prerequisites)
- [https://github.com/kubernetes-sigs/nfs-subdir-external-provisioner](https://github.com/kubernetes-sigs/nfs-subdir-external-provisioner)
- [部署 MinIO 以支持对象存储](https://todoit.tech/k8s/minio/)
