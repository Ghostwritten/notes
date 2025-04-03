![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/863e7f280a2345be95cf18a5ba075989.png)





## 1. 普通用户
创建用户demo

```bash
useradd demo

```

##  2. 创建私钥
下面的脚本展示了如何生成 PKI 私钥和 CSR。 设置 CSR 的 CN 和 O 属性很重要。CN 是用户名，O 是该用户归属的组。
```bash
openssl genrsa -out demo.key 2048
openssl req -new -key demo.key -out demo.csr -subj "/CN=demo"
```

```bash
$ cat demo.csr | base64 | tr -d "\n"
LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlJQ1ZEQ0NBVHdDQVFBd0R6RU5NQXNHQTFVRUF3d0VlV0Z1WnpDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRApnZ0VQQURDQ0FRb0NnZ0VCQU9iblNkNEs1K1BybjJQcnFidkd5a1pKQnpVMm5DbGt4bU13SXd0NzFiRjlaTDNNCjN0Q0ZMOG51NzBZQ2xnTm9DL2tIc1NaYWxsYkZQYTlETlhycll3UFZ3Qy9YNERGTTJyK1lsYWJNZ3ZXVHhxdngKUjMwRkhhS2VIaVkwZFdvUlBXcEtobWZZdlVtME9ZbFZkeUdkZXFPZXE0T3lGanZCVWgxZ2RoTmp1S0FoQzBlOQpnRjFZL3FtcmM3Q1EzVHI0QndmM1J0b1B3dW9TbldLQU9hUGNLMXhMRkE5Qy8yYnExVHFKaG5EcFR5eXlOYlFFCjJma2hMRGUvTi9oMGdkUzV3UnNqNzN2L29zdi83Z1VISGY2OHUzZWMxMkIvR1h3akhUdlE3VWRCaFBzbVBVQTAKTlVKbnNkdUdoRk1CYWZacTloRHVTV00zUmZSRnQ5dHgwWCsvQXUwQ0F3RUFBYUFBTUEwR0NTcUdTSWIzRFFFQgpDd1VBQTRJQkFRQVNuSFlJeW1zNjlXUUgyUUVzbmx1NVRIK1dtaTZIOEkvZTgwZWNGa3ZaK0c3NkJHeGo0ZE83ClJvTFdoaEZDZ0ZtVjhkMlhQK1FLVmRNaUNJckNqaXpqbDJwWVJ5YUs3WFNSV2ltMXdzejlHSUdPTis1Y1FQVngKQ2VDdm1xTS9wUlpnK1hQRFlWaG83Y21jeVpGc1RiOFl3cGZyT2Z5S0xtQ1FxZmU3RjZZaTVxSFBHSjB5aVRiMwp6b3RJblRmWTJKb2VCUm92U1hBN3E5bUNVeS94WTFJbTZSY2ZTNFNIN0lrNTkzaytiR3NkYjlEZlU5enVpNmlLClJQUkNEdXZEUU03NkNiZTFWM2tKelcwdWlycU14b1hLemJTOFBSNlpPOWxEaDF6bjNxTkg3TzZxVGVMdHAwcFMKQU1kWFlaMzZRd3JiVlR6WHBkMko1SE1YSTgyOFdJOG4KLS0tLS1FTkQgQ0VSVElGSUNBVEUgUkVRVUVTVC0tLS0tCg==
```

## 3. 创建 CertificateSigningRequest
创建一个 CertificateSigningRequest， 并通过 kubectl 将其提交到 Kubernetes 集群。 下面是生成 CertificateSigningRequest 的脚本

```bash
cat <<EOF | kubectl apply -f -
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: myuser
spec:
  request: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlJQ1ZEQ0NBVHdDQVFBd0R6RU5NQXNHQTFVRUF3d0VlV0Z1WnpDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRApnZ0VQQURDQ0FRb0NnZ0VCQU9iblNkNEs1K1BybjJQcnFidkd5a1pKQnpVMm5DbGt4bU13SXd0NzFiRjlaTDNNCjN0Q0ZMOG51NzBZQ2xnTm9DL2tIc1NaYWxsYkZQYTlETlhycll3UFZ3Qy9YNERGTTJyK1lsYWJNZ3ZXVHhxdngKUjMwRkhhS2VIaVkwZFdvUlBXcEtobWZZdlVtME9ZbFZkeUdkZXFPZXE0T3lGanZCVWgxZ2RoTmp1S0FoQzBlOQpnRjFZL3FtcmM3Q1EzVHI0QndmM1J0b1B3dW9TbldLQU9hUGNLMXhMRkE5Qy8yYnExVHFKaG5EcFR5eXlOYlFFCjJma2hMRGUvTi9oMGdkUzV3UnNqNzN2L29zdi83Z1VISGY2OHUzZWMxMkIvR1h3akhUdlE3VWRCaFBzbVBVQTAKTlVKbnNkdUdoRk1CYWZacTloRHVTV00zUmZSRnQ5dHgwWCsvQXUwQ0F3RUFBYUFBTUEwR0NTcUdTSWIzRFFFQgpDd1VBQTRJQkFRQVNuSFlJeW1zNjlXUUgyUUVzbmx1NVRIK1dtaTZIOEkvZTgwZWNGa3ZaK0c3NkJHeGo0ZE83ClJvTFdoaEZDZ0ZtVjhkMlhQK1FLVmRNaUNJckNqaXpqbDJwWVJ5YUs3WFNSV2ltMXdzejlHSUdPTis1Y1FQVngKQ2VDdm1xTS9wUlpnK1hQRFlWaG83Y21jeVpGc1RiOFl3cGZyT2Z5S0xtQ1FxZmU3RjZZaTVxSFBHSjB5aVRiMwp6b3RJblRmWTJKb2VCUm92U1hBN3E5bUNVeS94WTFJbTZSY2ZTNFNIN0lrNTkzaytiR3NkYjlEZlU5enVpNmlLClJQUkNEdXZEUU03NkNiZTFWM2tKelcwdWlycU14b1hLemJTOFBSNlpPOWxEaDF6bjNxTkg3TzZxVGVMdHAwcFMKQU1kWFlaMzZRd3JiVlR6WHBkMko1SE1YSTgyOFdJOG4KLS0tLS1FTkQgQ0VSVElGSUNBVEUgUkVRVUVTVC0tLS0tCg==
  signerName: kubernetes.io/kube-apiserver-client
  expirationSeconds: 864000000  # one day
  usages:
  - client auth
EOF
```

需要注意的几点：
- usage 字段必须是 'client auth'
- expirationSeconds 可以设置为更长（例如 864000 是十天）或者更短（例如 3600 是一个小时）
- request 字段是 CSR 文件内容的 base64 编码值， 要得到该值，可以执行命令：


## 4. 批准 CertificateSigningRequest
使用 kubectl 创建 CSR 并批准。

获取 CSR 列表：

```bash
kubectl get csr
```

批准 CSR：

```bash
kubectl certificate approve demo
```

证书的内容使用 base64 编码，存放在字段 status.certificate。

从 CertificateSigningRequest 导出颁发的证书：

```bash
kubectl get csr demo -o jsonpath='{.status.certificate}'| base64 -d > demo.crt
```


## 5. 创建 kubeconfig

```bash
cp  /root/.kube/config /root/backup/config 
kubectl config set-credentials demo --client-key=demo.key --client-certificate=demo.crt --embed-certs=true --kubeconfig=demo.kubeconfig
kubectl config set-context demo --cluster=a-t-k8sv2-cluster --user=demo --namespace=demo  --kubeconfig=demo.kubeconfig
```
拷贝/root/.kube/config的cluster 内容至 demo.kubeconfig

完整内容。



```bash
$ cat demo.kubeconfig
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURuakNDQW9hZ0F3SUJBZ0lVTWlJZVZDZTlCZFdhbDZhY1dHbW9xRS9FMTUwd0RRWUpLb1pJaHZjTkFRRUwKQlFBd1p6RUxNQWtHQTFVRUJoTUNRMDR4RVRBUEJnTlZCQWdUQ0ZOb1lXNW5hR0ZwTVJFd0R3WURWUVFIRXdoVAphR0Z1WjJoaGFURU1NQW9HQTFVRUNoTURhemh6TVE4d0RRWURWUVFMRXdaVGVYTjBaVzB4RXpBUkJnTlZCQU1UCkNtdDFZbVZ5Ym1WMFpYTXdIaGNOTWpRd016RTRNRFl4T0RBd1doY05Namt3TXpFM01EWXhPREF3V2pCbk1Rc3cKQ1FZRFZRUUdFd0pEVGpFUk1BOEdBMVVFQ0JNSVUyaGhibWRvWVdreEVUQVBCZ05WQkFjVENGTm9ZVzVuYUdGcApNUXd3Q2dZRFZRUUtFd05yT0hNeER6QU5CZ05WQkFzVEJsTjVjM1JsYlRFVE1CRUdBMVVFQXhNS2EzVmlaWEp1ClpYUmxjekNDQVNJd0RRWUpLb1pJaHZjTkFRRUJCUUFEZ2dFUEFEQ0NBUW9DZ2dFQkFNS3BvMTRvbm9hR1hTaDIKRGdMelQvNE8zK21lbUxQUlNkYUFpdHpHTFFZeHBUMjdtUjJvM21IenZUc3l0NkRld2VWd0paUVZvaE9STGRWQQpadTk5a3RBcUFLT2prSHc4RlZmNjBYNHh0NEs2T0tFVXZkYjNTMmZLbjI1WEsrbHNOeEM1aVBBRFI5eUZ3Ukd4CkNzRVh0cngzaWo2aldDdlZUckw1Q3p3YTRHclpqZkh5WlIzalNIZGp5V3NMaXZjMGU4TytkSjYrMzZtMkl6bmgKRUxpWE9NcSs0Q0Via3V0NG1DWHZoMkpvR2ZkbkxxVnRaYkhndGIrcmtRbms4YkViaFUxN3A1elV1K1czLzJzawptTkVMWmVrdElSR0RSOU9NcTZaVjdjNVJ3eTRrWnArcXRWbjAvYWpkR3N0dmJldXJ6T0FSMVdhOUt5bVJRakczCkphN0F4YkVDQXdFQUFhTkNNRUF3RGdZRFZSMFBBUUgvQkFRREFnRUdNQThHQTFVZEV3RUIvd1FGTUFNQkFmOHcKSFFZRFZSME9CQllFRk1SMXdUbldTbDh2aEEveUpUQjVvdno0dVE1Rk1BMEdDU3FHU0liM0RRRUJDd1VBQTRJQgpBUUJOYWFnTzM0WnNCbE53MENWeG5HTm14ZWNnaytqVUZJcng1OGxKRGtMalpvTnMyS3R0WFE1eThrd2lSVzJFClUzaERWWHVtU2pRWjE0SlZseGNiRjdrYm1GMFVZQ2dUWlYzbVUvbWlGSDBHaWM3a0FkU28wT3BIWFFqcTVLYWcKZGtXcS9vT1VVYmRtMkozRW9pcTQzN0E3bUYxcG45aVE0ck52emxjaDd4TSt4MitYNDFScmtTeDZVWFNTdXRVagpsdTdXRTJmcGZ2K1NJMnJTQzJsL3BxeHU2LzRuak9QNlJFdnk5ZkYxV1RDZHo2Y0lPUWNienZhTEpkVDgrVld6ClNpais5Z3lSdU1yNkY2WEtaNnFSUzNxVlZBMGxlRnJuUzAvOHJOa1Zxdk5RZjJuaVZsWiszUU1uWmRQT0p1VjMKK3BzZmZHQUt5bFBuRU92ck1EVlozOThzCi0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K
    server: https://192.168.118.20:6443
  name: a-t-k8sv2-cluster
contexts:
- context:
    cluster: a-t-k8sv2-cluster
    namespace: demo
    user: demo
  name: demo
current-context: demo
kind: Config
preferences: {}
users:
- name: demo
  user:
    client-certificate-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURSekNDQWkrZ0F3SUJBZ0lSQU50aGhVbmZHRVNFakxhUlVGSTRtQTR3RFFZSktvWklodmNOQVFFTEJRQXcKWnpFTE1Ba0dBMVVFQmhNQ1EwNHhFVEFQQmdOVkJBZ1RDRk5vWVc1bmFHRnBNUkV3RHdZRFZRUUhFd2hUYUdGdQpaMmhoYVRFTU1Bb0dBMVVFQ2hNRGF6aHpNUTh3RFFZRFZRUUxFd1pUZVhOMFpXMHhFekFSQmdOVkJBTVRDbXQxClltVnlibVYwWlhNd0hoY05NalV3TWpFNE1EY3dPREE0V2hjTk1qa3dNekUzTURZeE9EQXdXakFQTVEwd0N3WUQKVlFRREV3UjVZVzVuTUlJQklqQU5CZ2txaGtpRzl3MEJBUUVGQUFPQ0FROEFNSUlCQ2dLQ0FRRUE1dWRKM2dybgo0K3VmWSt1cHU4YktSa2tITlRhY0tXVEdZekFqQzN2VnNYMWt2Y3plMElVdnllN3ZSZ0tXQTJnTCtRZXhKbHFXClZzVTlyME0xZXV0akE5WEFMOWZnTVV6YXY1aVZwc3lDOVpQR3EvRkhmUVVkb3A0ZUpqUjFhaEU5YWtxR1o5aTkKU2JRNWlWVjNJWjE2bzU2cmc3SVdPOEZTSFdCMkUyTzRvQ0VMUjcyQVhWaitxYXR6c0pEZE92Z0hCL2RHMmcvQwo2aEtkWW9BNW85d3JYRXNVRDBML1p1clZPb21HY09sUExMSTF0QVRaK1NFc043ODMrSFNCMUxuQkd5UHZlLytpCnkvL3VCUWNkL3J5N2Q1elhZSDhaZkNNZE85RHRSMEdFK3lZOVFEUTFRbWV4MjRhRVV3RnA5bXIyRU81Sll6ZEYKOUVXMzIzSFJmNzhDN1FJREFRQUJvMFl3UkRBVEJnTlZIU1VFRERBS0JnZ3JCZ0VGQlFjREFqQU1CZ05WSFJNQgpBZjhFQWpBQU1COEdBMVVkSXdRWU1CYUFGTVIxd1RuV1NsOHZoQS95SlRCNW92ejR1UTVGTUEwR0NTcUdTSWIzCkRRRUJDd1VBQTRJQkFRQUpwSllRazFwakIyY3VyS2pCenp5QXdLTkxIbU9LcS83Z2UxMW5OMENTR0dNSlRGSlYKbDB1RzlkTkdsQ2htS3EwaElibWVTL0JCY0phOE1RTHZOblZVZVZtQWlIbVlhbTJoU2dZdlEwbEZlSkt5K0xPSgp5bmltMi9DUnVxNisySW5QdC9MZjYwSmRWQTNRT0U5M2ZHZHMxUTEweHVBOGg1NHJrQU1HL1dROS81WkQ3NENaClZxNGdwT3FmR3FSeWR5MGNwM2g0UVY0ekFTOE1xNFNoSUYxNzgvL1MzMjIrRHJQY0hkN09Bak5pUnpCaFlzY20KU2wyK2dkeXhsMk80YzJWQ1FBMGZ6ZHpaVzJFbGc4SkRjaHc5ay94dG5UREkyYzdCd0ZmdnlFcVhiNEhxRllXNQo4bUVaR0p0V0k4dmZuNmtsVEFnLzBYNHM0SE9TMytBaGxGZVcKLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo=
    client-key-data: LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQpNSUlFcGdJQkFBS0NBUUVBNXVkSjNncm40K3VmWSt1cHU4YktSa2tITlRhY0tXVEdZekFqQzN2VnNYMWt2Y3plCjBJVXZ5ZTd2UmdLV0EyZ0wrUWV4SmxxV1ZzVTlyME0xZXV0akE5WEFMOWZnTVV6YXY1aVZwc3lDOVpQR3EvRkgKZlFVZG9wNGVKalIxYWhFOWFrcUdaOWk5U2JRNWlWVjNJWjE2bzU2cmc3SVdPOEZTSFdCMkUyTzRvQ0VMUjcyQQpYVmorcWF0enNKRGRPdmdIQi9kRzJnL0M2aEtkWW9BNW85d3JYRXNVRDBML1p1clZPb21HY09sUExMSTF0QVRaCitTRXNONzgzK0hTQjFMbkJHeVB2ZS8raXkvL3VCUWNkL3J5N2Q1elhZSDhaZkNNZE85RHRSMEdFK3lZOVFEUTEKUW1leDI0YUVVd0ZwOW1yMkVPNUpZemRGOUVXMzIzSFJmNzhDN1FJREFRQUJBb0lCQVFDWTdVUTZBRVVXNmY0Vgp3b1lXN0pFWlBkSzlScDdrdDI3QlVLZFZPcjRNSUgxeFMxZWpDU0xlZmhZUTZ6T0pyQWFKOTdNM01MWHpZQXo3Cm4rcXlyMlZUcStUdWl1NWNHSWVrYjUxbDdIc3J4S1RIU2pUTHMzQ3VQRVRCanVyWXhmUkVDbm04RzRzNlhsZmgKQWk2MW44UVN5OHJTblVjbWFtSkZQSmJYUHF1UHVURFR4V2Fsb25JMUpET1FFb1JGNzJpa3lndzhHMDFuVFpyRQp3MmE5eXEyQkM3TGprTkRlMzk4R3haRmQ2TDFucm1mQWs1aDBERDRuTW5QSW5xOEp6RGxwRzFDRXRRdkNSUDBlClBHUERMRzdPbS95ZUw0TTRsSi8xUW5wNk44b1NSWlRQZkRObHRKU2RLZEs5d3BrUjllb256b1B1UE1rcjNRQnYKN05ENEZNMEJBb0dCQVBjNFBtYVdqKzJMcGJhME9wck5PTXBKMG5LRzFHTzVkbTNFTG5zMTQvOWtnRmhaa1ErZAo2U2RLd295MkFuaTZCNjhtREJpT0tuVWN6S1hiNkxzTjN4bDVJaUV2K2d3Y1RkNnZRUUkzb3labFVDNkNoZmc4CkthUDE0ajg0MEgrQ3VrQko0SmxJU1BIUm5MMDR2cXcrMEU3MlU4OWoyb0E4cC9HWnZwTWpxdmt0QW9HQkFPOGEKc2dEYUF4d0Izb0JBeFMrVkJmUysxZk5DUnpBMERmQnArVjlBM3c1OFc5S3B0OVQvb29GSGtJWmhuTGpybkVBQwpheVZYd1lWRDNTWkk5R2hzczl6anB4a0hsQzRwUSt6b1Q1LzFnUFlqdGgwSnFoRE16T3hsVFFmZ3ZpRWFGSmtRCkF4WGV5Z2xEWE1nK1M0dzJaMnFyWFVaemRBT0JrUXQvdGFnT3BzakJBb0dCQU81MjRqb3lvVUtSb1pkSzRmelEKV0NkSWJpYnF4NVFxSVlKZjZqWVBGWTRVYzNqRmJKZVR5b0tNS24xd1U1SUFYOGtpK2lmMWVoN2RXTW5rQmVubwp4M3JhellFVnRpeFlZUVNjS0NqcllnUjNWWkNIZHBLcjliNmlQMHFja3dGc0tCdzdKdHEwVHloeStLM05QcDhICk9BZnlzNFVvM0dzMkZ3bUZNNzdhZU9GQkFvR0JBTWlLZEZlQWd2RWZwRFdmbllNZUUyUEdGMzR5emJCaFNIdW0KOW8vc3dlak5adHBXbktmYVRMcnZnZ2tqbjZYOWZ3eTB1cGNVZG14R2tocUZQL0RCazAybDVzVjRkTkVPclRqcgpVN1ZPM1A0VXo2NmxKMjExeUQ1UmJIMDZBMTJTR1VxVGduTDZiQ3Urd3ZmMFA3cjIrbUFlSUZweGhSRlh2NGFNCmM1amp5UUZCQW9HQkFKRTF0MXdjU0szY2x0cUlpNWIwdG5VVjloQ2x1WHlBeHRHOFVxV0puNWhkU1AzSlBMUUMKeUJvUXdIZE1HbU9CRHQwOXVYVC9QeGdQcitKRjVZMjVOelhRVk01bitMZE02WVh4Q1BqYUltWnVma3Z4cDB3dwpKMnBnY3FWSU8xM2FwMHRYRFE5cmZpcmcxQnNNN0MzamhybjVuRTMzN2FZNDdMdlB0Ujg4UW9VbgotLS0tLUVORCBSU0EgUFJJVkFURSBLRVktLS0tLQo=

```
拷贝demo.kubeconfig至demo用户下/home/demo/.kube/config。

```bash
cp demo.kubeconfig /home/demo/.kube/config
```




## 6. 创建角色和角色绑定 
创建了证书之后，为了让这个用户能访问 Kubernetes 集群资源，现在就要创建 Role 和 RoleBinding 了。


```bash

kubectl apply -f - <<EOF
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: $NAMESPACE
  name: $USER-role
rules:
  # 只允许在指定命名空间内对以下资源进行操作
  - apiGroups: [""]
    resources: ["pods", "services", "deployments", "configmaps", "replicasets"]
    verbs: ["get", "list", "create", "delete", "update", "patch", "watch"]
  - apiGroups: ["apps"]
    resources: ["deployments", "replicasets"]
    verbs: ["get", "list", "create", "delete", "update", "patch", "watch"]
  - apiGroups: ["batch"]
    resources: ["jobs", "cronjobs"]
    verbs: ["get", "list", "create", "delete", "update", "patch", "watch"]
  - apiGroups: ["networking.k8s.io"]
    resources: ["ingresses"]
    verbs: ["get", "list", "create", "delete", "update", "patch", "watch"]
  - apiGroups: ["rbac.authorization.k8s.io"]
    resources: ["rolebindings", "roles"]
    verbs: ["get", "list", "create", "delete", "update", "patch", "watch"]
EOF

# 创建 RoleBinding，将 ServiceAccount 与 Role 绑定
echo "Creating RoleBinding to link ServiceAccount '$USER' and Role in '$NAMESPACE'..."
kubectl apply -f - <<EOF
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: $USER-binding
  namespace: $NAMESPACE
subjects:
  - kind: User
    name: $USER
    apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: $USER-role
  apiGroup: rbac.authorization.k8s.io
EOF

# 设置命名空间资源配额限制 (CPU 和 Memory)
echo "Setting resource quota for namespace '$NAMESPACE'..."
kubectl apply -f - <<EOF
apiVersion: v1
kind: ResourceQuota
metadata:
  name: $USER-quota
  namespace: $NAMESPACE
spec:
  hard:
    requests.cpu: "30"
    limits.cpu: "30"
    requests.memory: "100Gi"
    limits.memory: "100Gi"
EOF
```

## 7. 测试

demo用户只能查看demo命名空间内容。

```bash
[demo@a-t-k8sv2-master01 ~]$ kubectl get node
Error from server (Forbidden): nodes is forbidden: User "demo" cannot list resource "nodes" in API group "" at the cluster scope
[demo@a-t-k8sv2-master01 ~]$ kubectl get pod
Error from server (Forbidden): pods is forbidden: User "demo" cannot list resource "pods" in API group "" in the namespace "default"
[demo@a-t-k8sv2-master01 ~]$ kubectl get pod -n demo
NAME                                        READY   STATUS    RESTARTS   AGE
ngtest-5c4449c45d-7vqmw                     1/1     Running   0          74d
spring-webmvc-75f4b5f98-lbn2z               1/1     Running   0          75d
uat-milvus-v1-etcd-0                        1/1     Running   0          48d
uat-milvus-v1-minio-78665b8967-bblzr        1/1     Running   0          48d
uat-milvus-v1-standalone-76dd9c67b8-snrsb   1/1     Running   0          48d
[demo@a-t-k8sv2-master01 ~]$ kubectl get ns
Error from server (Forbidden): namespaces is forbidden: User "demo" cannot list resource "namespaces" in API group "" at the cluster scope

```


参考： 

- [https://kubernetes.io/zh-cn/docs/reference/access-authn-authz/certificate-signing-requests/](https://kubernetes.io/zh-cn/docs/reference/access-authn-authz/certificate-signing-requests/)
