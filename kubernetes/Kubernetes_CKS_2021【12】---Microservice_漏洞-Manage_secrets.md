

扩展链接：
[https://blog.csdn.net/xixihahalelehehe/article/details/108602320](https://blog.csdn.net/xixihahalelehehe/article/details/108602320)

## 1. 介绍


![在这里插入图片描述](https://img-blog.csdnimg.cn/20210427143342989.png?shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210427143645951.png?shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210427143714572.png?shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
## 2. Create Simple Secret Scenario
参考链接：[https://kubernetes.io/zh/docs/concepts/configuration/secret/#using-secrets](https://kubernetes.io/zh/docs/concepts/configuration/secret/#using-secrets)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210427143804876.png?shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

```c
root@master:~# k create secret generic secret1 --from-literal user=admin
secret/secret1 created
root@master:~# k create secret generic secret2 --from-literal pass=12345678
secret/secret2 created


root@master:~# k run pod --image=nginx -oyaml --dry-run=client
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: pod
  name: pod
spec:
  containers:
  - image: nginx
    name: pod
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
root@master:~# k run pod --image=nginx -oyaml --dry-run=client > pod.yaml


root@master:~# vim pod.yaml 
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: pod
  name: pod
spec:
  containers:
  - image: nginx
    name: pod
    resources: {}
    env:
      - name: PASSWORD
        valueFrom:
          secretKeyRef:
            name: secret2
            key: pass
    volumeMounts:
    - name: secret1
      mountPath: "/etc/secret1"
      readOnly: true
  volumes:
  - name: secret1
    secret:
      secretName: secret1
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}


root@master:~# k -f pod.yaml create
pod/pod created
root@master:~# k get pods
NAME   READY   STATUS    RESTARTS   AGE
pod    1/1     Running   0          52s

root@master:~# k exec pod -- env |grep PASS
PASSWORD=12345678

root@master:~# k exec pod -- mount |grep secret1
tmpfs on /etc/secret1 type tmpfs (ro,relatime)

root@master:~# k exec pod -- ls /etc/secret1
user
root@master:~# k exec pod -- cat /etc/secret1/user
admin
```


## 3. Hack Secrets in Docker
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210427154725952.png?shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

```c
root@master:~# k get pod -owide
NAME   READY   STATUS    RESTARTS   AGE     IP               NODE    NOMINATED NODE   READINESS GATES
pod    1/1     Running   0          5m28s   10.244.166.137   node1   <none>           <none>

root@node1:~# docker ps |grep nginx
a3c39ce6e40e        nginx                  "/docker-entrypoint.…"   5 minutes ago       Up 5 minutes                                 k8s_pod_pod_default_31010eac-76fe-463d-a20a-c18aec88c174_0
root@node1:~# docker inspect a3
....

  "Env": [
                "PASSWORD=12345678",
                "KUBERNETES_PORT_443_TCP_PORT=443",
                "KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1",
                "KUBERNETES_SERVICE_HOST=10.96.0.1",
                "KUBERNETES_SERVICE_PORT=443",
                "KUBERNETES_SERVICE_PORT_HTTPS=443",
                "KUBERNETES_PORT=tcp://10.96.0.1:443",
                "KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443",
                "KUBERNETES_PORT_443_TCP_PROTO=tcp",
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
                "NGINX_VERSION=1.19.10",
                "NJS_VERSION=0.5.3",
                "PKG_RELEASE=1~buster"
            ],

....

root@node1:~# docker cp a3:/etc/secret1 secret1
root@node1:~# cat secret1/user 
admin
```


## 4. Hack Secrets in ETCD
参考链接：
[https://kubernetes.io/zh/docs/tasks/administer-cluster/configure-upgrade-etcd/#%E5%AE%89%E5%85%A8%E9%80%9A%E4%BF%A1](https://kubernetes.io/zh/docs/tasks/administer-cluster/configure-upgrade-etcd/#%E5%AE%89%E5%85%A8%E9%80%9A%E4%BF%A1)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210427155235982.png?shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

```bash
root@master:~# ETCDCTL_API=3 etcdctl --endpoints https://192.168.211.40:2379   --cert /etc/kubernetes/pki/etcd/server.crt --key /etc/kubernetes/pki/etcd/server.key --cacert /etc/kubernetes/pki/etcd/ca.crt endpoint health
https://192.168.211.40:2379 is healthy: successfully committed proposal: took = 19.341929ms


root@master:~# ETCDCTL_API=3 etcdctl --endpoints https://192.168.211.40:2379   --cert /etc/kubernetes/pki/etcd/server.crt --key /etc/kubernetes/pki/etcd/server.key --cacert /etc/kubernetes/pki/etcd/ca.crt get /registry/secrets/default/secret1
/registry/secrets/default/secret1
k8s


v1SecretƁ
® 
secret1default"*$82fd120e-dccd-4ac5-99c9-d2e5ff2915a12̣z_
kubectl-createUpdateṿFieldsV1:-
+{"f:data":{".":{},"f:user":{}},"f:type":{}} 
useradminOpaque"
Xshellroot@master:~# ETCDCTL_API=3 etcdctl --endpoints https://192.168.211.40:2379   --cert /etc/kubernetes/pki/etcd/server.crt --key ubernetes/pki/etcd/server.key --cacert /etc/kubernetes/pki/etcd/ca.crt get /registry/secrets/default/secret2
/registry/secrets/default/secret2
k8s


v1SecretɁ
® 
secret2default"*$4cf99bbb-3980-40bc-9138-8b23f2a7e2132⣞z_
kubectl-createUpdatev⣞FieldsV1:-
+{"f:data":{".":{},"f:pass":{}},"f:type":{}} 
pas12345678Opaque"

```
## 5. ETCD Encryption
参考链接：
[https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210427161639228.png?shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210427162824572.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210427163026455.png?shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210427163136377.png?xshadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
## 6. Practice - Encrypt ETCD

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210427163838875.png?shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

```c
root@master:~# cd /etc/kubernetes/
root@master:/etc/kubernetes# ls
admin.conf  controller-manager.conf  kubelet.conf  manifests  pki  scheduler.conf
root@master:/etc/kubernetes# mkdir etcd
root@master:/etc/kubernetes# cd etcd/
root@master:/etc/kubernetes/etcd# ls
root@master:/etc/kubernetes/etcd# echo password | base64
cGFzc3dvcmQK
root@master:/etc/kubernetes/etcd# echo password
password
root@master:/etc/kubernetes/etcd# echo -n password
passwordroot@master:/etc/kubernetes/etcd# echo -n password | base64
cGFzc3dvcmQ=
root@master:/etc/kubernetes/etcd# vim ec.yaml
apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources:
  - resources:
    - secrets
    providers:
    - aescbc:
        keys:
        - name: key1
          secret: cGFzc3dvcmQ= 
    - identity: {}


root@master:/etc/kubernetes/etcd# cd ../manifests/
root@master:/etc/kubernetes/manifests# ls
etcd.yaml  kube-apiserver.yaml  kube-controller-manager.yaml  kube-scheduler.yaml
root@master:/etc/kubernetes/manifests# vim kube-apiserver.yaml 
apiVersion: v1
kind: Pod
metadata:
  annotations:
    kubeadm.kubernetes.io/kube-apiserver.advertise-address.endpoint: 192.168.211.40:6443
  creationTimestamp: null
  labels:
    component: kube-apiserver
    tier: control-plane
  name: kube-apiserver
  namespace: kube-system
spec:
  containers:
  - command:
    - kube-apiserver
    - --encryption-provider-config=/etc/kubernetes/etcd/ec.yaml #添加
    - --advertise-address=192.168.211.40
    - --allow-privileged=true
    - --authorization-mode=Node,RBAC
    - --client-ca-file=/etc/kubernetes/pki/ca.crt
    - --enable-admission-plugins=NodeRestriction
    - --enable-bootstrap-token-auth=true
    - --etcd-cafile=/etc/kubernetes/pki/etcd/ca.crt
    - --etcd-certfile=/etc/kubernetes/pki/apiserver-etcd-client.crt
    - --etcd-keyfile=/etc/kubernetes/pki/apiserver-etcd-client.key
    - --etcd-servers=https://127.0.0.1:2379
    - --insecure-port=0
    - --kubelet-client-certificate=/etc/kubernetes/pki/apiserver-kubelet-client.crt
    - --kubelet-client-key=/etc/kubernetes/pki/apiserver-kubelet-client.key
    - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
    - --proxy-client-cert-file=/etc/kubernetes/pki/front-proxy-client.crt
    - --proxy-client-key-file=/etc/kubernetes/pki/front-proxy-client.key
    - --requestheader-allowed-names=front-proxy-client
    - --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.crt
    - --requestheader-extra-headers-prefix=X-Remote-Extra-
    - --requestheader-group-headers=X-Remote-Group
    - --requestheader-username-headers=X-Remote-User
    - --secure-port=6443
    - --service-account-issuer=https://kubernetes.default.svc.cluster.local
    - --service-account-key-file=/etc/kubernetes/pki/sa.pub
    - --service-account-signing-key-file=/etc/kubernetes/pki/sa.key
    - --service-cluster-ip-range=10.96.0.0/12
    - --tls-cert-file=/etc/kubernetes/pki/apiserver.crt
    - --tls-private-key-file=/etc/kubernetes/pki/apiserver.key
    image: k8s.gcr.io/kube-apiserver:v1.20.0
    imagePullPolicy: IfNotPresent
    livenessProbe:
      failureThreshold: 8
      httpGet:
        host: 192.168.211.40
        path: /livez
        port: 6443
        scheme: HTTPS
      initialDelaySeconds: 10
      periodSeconds: 10
      timeoutSeconds: 15
    name: kube-apiserver
    readinessProbe:
      failureThreshold: 3
      httpGet:
        host: 192.168.211.40
        path: /readyz
        port: 6443
        scheme: HTTPS
      periodSeconds: 1
      timeoutSeconds: 15
    resources:
      requests:
        cpu: 250m
    startupProbe:
      failureThreshold: 24
      httpGet:
        host: 192.168.211.40
        path: /livez
        port: 6443
        scheme: HTTPS
      initialDelaySeconds: 10
      periodSeconds: 10
      timeoutSeconds: 15
    volumeMounts:
    - mountPath: /etc/ssl/certs
      name: ca-certs
      readOnly: true
    - mountPath: /etc/ca-certificates
      name: etc-ca-certificates
      readOnly: true
    - mountPath: /etc/kubernetes/pki
      name: k8s-certs
      readOnly: true
    - mountPath: /etc/kubernetes/etcd   #添加
      name: etcd   #添加
      readOnly: true   #添加
    - mountPath: /usr/local/share/ca-certificates
      name: usr-local-share-ca-certificates
      readOnly: true
    - mountPath: /usr/share/ca-certificates
      name: usr-share-ca-certificates
      readOnly: true
  hostNetwork: true
  priorityClassName: system-node-critical
  volumes:
  - hostPath:
      path: /etc/ssl/certs
      type: DirectoryOrCreate
    name: ca-certs
  - hostPath:
      path: /etc/ca-certificates
      type: DirectoryOrCreate
    name: etc-ca-certificates
  - hostPath:
      path: /etc/kubernetes/pki
      type: DirectoryOrCreate
    name: k8s-certs
  - hostPath:   #添加
      path: /etc/kubernetes/etcd  #添加
      type: DirectoryOrCreate   #添加
    name: etcd   #添加
  - hostPath:
      path: /usr/local/share/ca-certificates
      type: DirectoryOrCreate
    name: usr-local-share-ca-certificates
  - hostPath:
      path: /usr/share/ca-certificates
      type: DirectoryOrCreate
    name: usr-share-ca-certificates
status: {}


root@master:/etc/kubernetes/manifests# ps aux |grep apiserver
root      63152  0.0  0.0  14424  1092 pts/1    S+   02:07   0:00 grep --color=auto apiserver


root@master:/etc/kubernetes/manifests# cd /var/log/pods/
root@master:/var/log/pods# ls
kube-system_calico-node-ngbm8_837fbf7e-0060-4f5c-bd62-fdecf5f7e334
kube-system_etcd-master_77699ae6105937dbb48c0a720843ce8e
kube-system_kube-apiserver-master_ae8926e93b50c0469bb29e747b4c459f
kube-system_kube-controller-manager-master_360cd07520ba8dce55b5d403c66acf83
kube-system_kube-proxy-lfkn9_08f4f57e-d10b-4efe-99d7-33509c6492b0
kube-system_kube-scheduler-master_81d2d21449d64d5e6d5e9069a7ca99ed

root@master:/var/log/pods# tail -f kube-system_kube-apiserver-master_ae8926e93b50c0469bb29e747b4c459f/kube-apiserver/4.log 
{"log":"Flag --insecure-port has been deprecated, This flag has no effect now and will be removed in v1.24.\n","stream":"stderr","time":"2021-04-27T09:07:15.165954784Z"}
{"log":"I0427 09:07:15.166039       1 server.go:632] external host was not specified, using 192.168.211.40\n","stream":"stderr","time":"2021-04-27T09:07:15.166135424Z"}
{"log":"I0427 09:07:15.166569       1 server.go:182] Version: v1.20.0\n","stream":"stderr","time":"2021-04-27T09:07:15.166680164Z"}
{"log":"Error: error while parsing encryption provider configuration file \"/etc/kubernetes/etcd/ec.yaml\": error while parsing file: resources[0].providers[0].aescbc.keys[0].secret: Invalid value: \"REDACTED\": secret is not of the expected length, got 8, expected one of [16 24 32]\n","stream":"stderr","time":"2021-04-27T09:07:15.713401195Z"}     #密文长度不合规范


root@master:/var/log/pods# cd /etc/kubernetes/etcd/
root@master:/etc/kubernetes/etcd# ls
ec.yaml
root@master:/etc/kubernetes/etcd# echo -n passwordpassword | base64
cGFzc3dvcmRwYXNzd29yZA==


root@master:/etc/kubernetes/etcd# vim ec.yaml 
root@master:/etc/kubernetes/etcd# cat ec.yaml
apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources:
  - resources:
    - secrets
    providers:
    - aescbc:
        keys:
        - name: key1
          secret: cGFzc3dvcmRwYXNzd29yZA==   #修改此行
    - identity: {}


root@master:/etc/kubernetes/etcd# cd ../manifests/
root@master:/etc/kubernetes/manifests# mv kube-apiserver.yaml ..
root@master:/etc/kubernetes/manifests# ps aux |grep apiserver
root      68173  0.0  0.0  14424  1040 pts/1    S+   02:10   0:00 grep --color=auto apiserver

root@master:/etc/kubernetes/manifests# mv ../kube-apiserver.yaml .
root@master:/etc/kubernetes/manifests# ps aux |grep apiserver
root      69304  0.0  0.0  14424  1108 pts/1    S+   02:11   0:00 grep --color=auto apiserver


root@master:~# k get secret 
NAME                  TYPE                                  DATA   AGE
default-token-2xr8c   kubernetes.io/service-account-token   3      2d15h
root@master:~# k get secret  default-token-2xr8c  -o yaml




root@master:~# ETCDCTL_API=3 etcdctl --endpoints https://192.168.211.40:2379   --cert /etc/kubernetes/pki/etcd/server.crt --key /etc/kubernetes/pki/etcd/server.key --cacert /etc/kubernetes/pki/etcd/ca.crt get  /registry/secrets/default/default-token-2xr8c
/registry/secrets/default/default-token-2xr8c
k8s


v1Secret 
ك
default-token-2xr8cdefault"*$530bbfe5-0397-4526-8b97-995654fb0c792µ¦b-
"kubernetes.io/service-account.namedefaultbI
!kubernetes.io/service-account.uid$8b32bf2f-09c3-4066-bf32-981a3da16aeez 
kube-controller-managerUpdatevµ¦FieldsV1:ǁ
ā{"f:data":{".":{},"f:ca.crt":{},"f:namespace":{},"f:token":{}},"f:metadata":{"f:annotations":{".":{},"f:kubernetes.io/service-account.name":{},"f:kubernetes.io/service-account.uid":{}}},"f:type":{}}µ
ca.crt-----BEGIN CERTIFICATE-----
MIIC5zCCAc+gAwIBAgIBADANBgkqhkiG9w0BAQsFADAVMRMwEQYDVQQDEwprdWJl
cm5ldGVzMB4XDTIxMDQyNTExMzEzN1oXDTMxMDQyMzExMzEzN1owFTETMBEGA1UE
AxMKa3ViZXJuZXRlczCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBANP+
TEuItY/ZoGFW5WNzPIMLxYwUR/TlNN5M8A2+p8tigekLtzHuioVt5sL/1yEZvjYr
eE63SBv1/KLIbnav12GeAkhRKo7DRu16Z0JglQgncZGXciAL9/sQ08UaNZeRpGak
9b42YET1eU4rNn2K+uRq1k71rOpAj+2SRnhi0P6knMfo51pAtlbg4IlKLCv5WHBk
zEKtufWaxjANwy74EtR6fxzRCRwVJlbxlKKX/zOyStruTPFKuMn1T7pvf1BpyVop
0Y+R4Uq72z3EFfJ+nX2DrOBKdnyccQ1VPCSMSP42jIe2IljMfnbXxhYjCfW4yOBB
n2jDaADDbEENk5vqXeMCAwEAAaNCMEAwDgYDVR0PAQH/BAQDAgKkMA8GA1UdEwEB
/wQFMAMBAf8wHQYDVR0OBBYEFPNvWidCN48MhgVQ3tLD/RrO5pI9MA0GCSqGSIb3
DQEBCwUAA4IBAQDN6JnxMaV9qgn4e4XK8pxaIXSy8CJrYW6sQ0EK1BHeaAHLuTm0
l0O8XP+xELeV1hPrwPG24tfiSnWj71Rjma/cXSHTRpSvtROo2rGkpEUMufnPqo9+
kKORg1k6ajTpJPxsakIBzUXLu6ei7t/jzp6pTIXS1Q/dIEgU0ovKTQ1KgWe3pomH
fJAUfbmkOiO7IwluTlDpckx53FGfFB4f5Goi0qb+u6EitXCim98ikzZaHPAqkDc+
ZXdn+9x474sljnUsoZJZICwyraOCiWdOCuB5R7taHirTi3R2vsj1lkRavC5ydZd2
2lSe8x57cpdghjXKMXgK04VD73zG9UXzzA74
-----END CERTIFICATE-----
 
	namespacedefault 
tokeneyJhbGciOiJSUzI1NiIsImtpZCI6IktRNXFhRW5OLW5tdjhVX3Y2SnhuYXNocVF6WXF0ZFdTX0hIdWQwVkphYjgifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJkZWZhdWx0Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZWNyZXQubmFtZSI6ImRlZmF1bHQtdG9rZW4tMnhyOGMiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC5uYW1lIjoiZGVmYXVsdCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6IjhiMzJiZjJmLTA5YzMtNDA2Ni1iZjMyLTk4MWEzZGExNmFlZSIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDpkZWZhdWx0OmRlZmF1bHQifQ.lzHD6W-xBuiqf9CHPB4LIbpRdoJmMv7wSGt0fnp7p-slYEtSpk4ax2kQNQ8j-eEFFWahZpzZhHlanIBsG5LTNcqZP_N9hxK5YgxIz4kJ-nbDvqeqBICM8SVL3C3kID1wb32I0zpXr3BDq33NbKh1e0-19T5zsKFWsnahfhd2HHuV4bnr7zEZYtjIv5Taeyu28F79929a80bQykG5v1Si5xibPP-xPpwXizbdGS-nwbqre4lSgjyFjoNZrjYEUVkYTsfxBmWmdqiCv1EvgFftTQ2RMVSk0e-qpbWowg-uGcYHLmyzx_R4QM0wPnDIwkxtxZIZdV7cVQv11Fw9kQVzeQ#kubernetes.io/service-account-token"



root@master:~# k create secret generic very-secure --from-literal cc=1234
secret/very-secure created
root@master:~# ETCDCTL_API=3 etcdctl --endpoints https://192.168.211.40:2379   --cert /etc/kubernetes/pki/etcd/server.crt --key /etc/kubernetes/pki/etcd/server.key --cacert /etc/kubernetes/pki/etcd/ca.crt get  /registry/secrets/default/very-secure
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210428105618767.png?shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

```c
root@master:~# k get secret very-secure -oyaml
apiVersion: v1
data:
  cc: MTIzNA==
kind: Secret
metadata:
  creationTimestamp: "2021-04-28T02:55:39Z"
  managedFields:
  - apiVersion: v1
    fieldsType: FieldsV1
    fieldsV1:
      f:data:
        .: {}
        f:cc: {}
      f:type: {}
    manager: kubectl-create
    operation: Update
    time: "2021-04-28T02:55:39Z"
  name: very-secure
  namespace: default
  resourceVersion: "14227"
  uid: f175c1d7-83f7-4cd7-8601-64bd814feeab
type: Opaque



root@master:~# echo  MTIzNA== | base64 -d
1234root@master:~# k get secret
NAME                  TYPE                                  DATA   AGE
default-token-2xr8c   kubernetes.io/service-account-token   3      2d15h
very-secure           Opaque                                1      4m28s
root@master:~# k delete secret default-token-2xr8c
secret "default-token-2xr8c" deleted
root@master:~# k get secret
NAME                  TYPE                                  DATA   AGE
default-token-s446z   kubernetes.io/service-account-token   3      1s
very-secure           Opaque                                1      4m43s

```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210428110204654.png?shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

```bash
root@master:/etc/kubernetes/etcd# cat ec.yaml
apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources:
  - resources:
    - secrets
    providers:
    - aescbc:
        keys:
        - name: key1
          secret: cGFzc3dvcmRwYXNzd29yZA==   #修改此行
#    - identity: {}    #注释掉



root@master:/etc/kubernetes/etcd# cd ../manifests/
root@master:/etc/kubernetes/manifests# mv kube-apiserver.yaml ..
root@master:/etc/kubernetes/manifests# ps aux |grep apiserver
root      68173  0.0  0.0  14424  1040 pts/1    S+   02:10   0:00 grep --color=auto apiserver

root@master:/etc/kubernetes/manifests# mv ../kube-apiserver.yaml .
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210428111554483.png)

```bash
root@master:/etc/kubernetes/etcd# cat ec.yaml
apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources:
  - resources:
    - secrets
    providers:
    - aescbc:
        keys:
        - name: key1
          secret: cGFzc3dvcmRwYXNzd29yZA==   #修改此行
    - identity: {}    #取消注释


root@master:/etc/kubernetes/etcd# cd ../manifests/
root@master:/etc/kubernetes/manifests# mv kube-apiserver.yaml ..
root@master:/etc/kubernetes/manifests# ps aux |grep apiserver
root      68173  0.0  0.0  14424  1040 pts/1    S+   02:10   0:00 grep --color=auto apiserver

root@master:/etc/kubernetes/manifests# mv ../kube-apiserver.yaml .


root@master:~# k -n kube-system get secret
NAME                                             TYPE                                  DATA   AGE
attachdetach-controller-token-lgjcx              kubernetes.io/service-account-token   3      2d15h
bootstrap-signer-token-fm96m                     kubernetes.io/service-account-token   3      2d15h
bootstrap-token-twmwqd                           bootstrap.kubernetes.io/token         7      2d15h
calico-kube-controllers-token-km9zk              kubernetes.io/service-account-token   3      2d15h
calico-node-token-zjqnp                          kubernetes.io/service-account-token   3      2d15h
certificate-controller-token-2n29x               kubernetes.io/service-account-token   3      2d15h
clusterrole-aggregation-controller-token-ws62f   kubernetes.io/service-account-token   3      2d15h
coredns-token-r4r2k                              kubernetes.io/service-account-token   3      2d15h
cronjob-controller-token-djsb6                   kubernetes.io/service-account-token   3      2d15h
daemon-set-controller-token-q4zwx                kubernetes.io/service-account-token   3      2d15h
default-token-8lqmr                              kubernetes.io/service-account-token   3      2d15h
deployment-controller-token-5ll65                kubernetes.io/service-account-token   3      2d15h
disruption-controller-token-mptgf                kubernetes.io/service-account-token   3      2d15h
endpoint-controller-token-5x9bl                  kubernetes.io/service-account-token   3      2d15h
endpointslice-controller-token-gcjz8             kubernetes.io/service-account-token   3      2d15h
endpointslicemirroring-controller-token-pbrwx    kubernetes.io/service-account-token   3      2d15h
expand-controller-token-hv5lv                    kubernetes.io/service-account-token   3      2d15h
generic-garbage-collector-token-kvdq6            kubernetes.io/service-account-token   3      2d15h
horizontal-pod-autoscaler-token-z2kw4            kubernetes.io/service-account-token   3      2d15h
job-controller-token-rzd44                       kubernetes.io/service-account-token   3      2d15h
kube-proxy-token-tfljg                           kubernetes.io/service-account-token   3      2d15h
namespace-controller-token-cqf85                 kubernetes.io/service-account-token   3      2d15h
node-controller-token-hlfq7                      kubernetes.io/service-account-token   3      2d15h
persistent-volume-binder-token-s5l7q             kubernetes.io/service-account-token   3      2d15h
pod-garbage-collector-token-2bxvk                kubernetes.io/service-account-token   3      2d15h
pv-protection-controller-token-knqrm             kubernetes.io/service-account-token   3      2d15h
pvc-protection-controller-token-h25mp            kubernetes.io/service-account-token   3      2d15h
replicaset-controller-token-vd8x7                kubernetes.io/service-account-token   3      2d15h
replication-controller-token-2zq5m               kubernetes.io/service-account-token   3      2d15h
resourcequota-controller-token-cxsdh             kubernetes.io/service-account-token   3      2d15h
root-ca-cert-publisher-token-65d6b               kubernetes.io/service-account-token   3      2d15h
service-account-controller-token-ktjjn           kubernetes.io/service-account-token   3      2d15h
service-controller-token-ljjb8                   kubernetes.io/service-account-token   3      2d15h
statefulset-controller-token-9c25f               kubernetes.io/service-account-token   3      2d15h
token-cleaner-token-lspdd                        kubernetes.io/service-account-token   3      2d15h
ttl-controller-token-6vv9d                       kubernetes.io/service-account-token   3      2d15h




root@master:~# k get secret -A -oyaml | kubectl replace -f -
secret/default-token-s446z replaced
secret/very-secure replaced
secret/default-token-wt6q2 replaced
secret/default-token-nh879 replaced
secret/attachdetach-controller-token-lgjcx replaced
secret/bootstrap-signer-token-fm96m replaced
secret/bootstrap-token-twmwqd replaced
secret/calico-kube-controllers-token-km9zk replaced
secret/calico-node-token-zjqnp replaced
secret/certificate-controller-token-2n29x replaced
secret/clusterrole-aggregation-controller-token-ws62f replaced
secret/coredns-token-r4r2k replaced
secret/cronjob-controller-token-djsb6 replaced
secret/daemon-set-controller-token-q4zwx replaced
secret/default-token-8lqmr replaced
secret/deployment-controller-token-5ll65 replaced
secret/disruption-controller-token-mptgf replaced
secret/endpoint-controller-token-5x9bl replaced
secret/endpointslice-controller-token-gcjz8 replaced
secret/endpointslicemirroring-controller-token-pbrwx replaced
secret/expand-controller-token-hv5lv replaced
secret/generic-garbage-collector-token-kvdq6 replaced
secret/horizontal-pod-autoscaler-token-z2kw4 replaced
secret/job-controller-token-rzd44 replaced
secret/kube-proxy-token-tfljg replaced
secret/namespace-controller-token-cqf85 replaced
secret/node-controller-token-hlfq7 replaced
secret/persistent-volume-binder-token-s5l7q replaced
secret/pod-garbage-collector-token-2bxvk replaced
secret/pv-protection-controller-token-knqrm replaced
secret/pvc-protection-controller-token-h25mp replaced
secret/replicaset-controller-token-vd8x7 replaced
secret/replication-controller-token-2zq5m replaced
secret/resourcequota-controller-token-cxsdh replaced
secret/root-ca-cert-publisher-token-65d6b replaced
secret/service-account-controller-token-ktjjn replaced
secret/service-controller-token-ljjb8 replaced
secret/statefulset-controller-token-9c25f replaced
secret/token-cleaner-token-lspdd replaced
secret/ttl-controller-token-6vv9d replaced

```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210428112001247.png?shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

