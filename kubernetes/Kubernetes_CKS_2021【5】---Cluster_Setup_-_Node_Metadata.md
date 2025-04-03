

---

 - [Kubernetes安全专家认证 (CKS)考试动员](https://ghostwritten.blog.csdn.net/article/details/112358241)
 - [云原生圣经](https://ghostwritten.blog.csdn.net/article/details/108562082)

----
## 1. 介绍
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0c0035f1e628c387d06959d1e8dec708.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c1be000e171434d7cf769bb7163d9c50.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/19a37f5f1596cb5349f7a1bd426e6a68.png)
## 2. Practice: Access Node Metadata
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/085739c4ceb0569ca81eee981abd68aa.png)

参考链接：
[https://cloud.google.com/compute/docs/storing-retrieving-metadata](https://cloud.google.com/compute/docs/storing-retrieving-metadata)
```bash
curl "http://metadata.google.internal/computeMetadata/v1/instance/disks/" -H "Metadata-Flavor: Google"


curl "http://metadata.google.internal/computeMetadata/v1/instance/disks/0/" -H "Metadata-Flavor: Google"

root@master:~/clash# k run nginx --image=nginx
pod/nginx created
root@master:~/clash# k get pods
NAME      READY   STATUS    RESTARTS   AGE
backend   1/1     Running   0          43h
nginx     1/1     Running   0          22s
pod1      1/1     Running   0          20h
pod2      1/1     Running   0          20h
root@master:~/clash# k exec -ti nginx bash
root@nginx:/# curl "http://metadata.google.internal/computeMetadata/v1/instance/disks/" -H "Metadata-Flavor: Google"


```
## 3. Practice: Protect Node Metadata via NetworkPolicy

```c
root@master:~/cks/metadata# cat deny.yaml
# all pods in namespace cannot access metadata endpoint
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: cloud-metadata-deny
  namespace: default
spec:
  podSelector: {}
  policyTypes:
  - Egress
  egress:
  - to:
    - ipBlock:
        cidr: 0.0.0.0/0
        except:
        - 169.254.169.254/32


root@master:~/cks/metadata#  k create -f deny.yaml 
networkpolicy.networking.k8s.io/cloud-metadata-deny created
root@master:~/clash# k exec -ti nginx bash
root@nginx:/# curl "http://metadata.google.internal/computeMetadata/v1/instance/disks/" -H "Metadata-Flavor: Google"         ## 卡住



root@master:~/cks/metadata# cat allow.yaml
# only pods with label are allowed to access metadata endpoint
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: cloud-metadata-allow
  namespace: default
spec:
  podSelector:
    matchLabels:
      role: metadata-accessor
  policyTypes:
  - Egress
  egress:
  - to:
    - ipBlock:
        cidr: 169.254.169.254/32


root@master:~/cks/metadata# k create -f allow.yaml 
networkpolicy.networking.k8s.io/cloud-metadata-allow created

root@master:~/cks/metadata# k label pod nginx role=metadata-accessor
pod/nginx labeled


root@master:~/cks/metadata# k get pods nginx --show-labels
NAME    READY   STATUS    RESTARTS   AGE   LABELS
nginx   1/1     Running   0          10m   role=metadata-accessor,run=nginx
root@master:~/clash# k exec -ti nginx bash
root@nginx:/# curl "http://metadata.google.internal/computeMetadata/v1/instance/disks/" -H "Metadata-Flavor: Google"  #正常访问

```

测试删除metadata中的role
```c
root@master:~/cks/metadata# k edit pod nginx
metadata:
  annotations:
    cni.projectcalico.org/podIP: 192.168.104.31/32
  creationTimestamp: "2021-04-22T03:17:45Z"
  labels:
    role: metadata-accessor   #删除
    run: nginx
  name: nginx
  namespace: default

root@master:~/clash# k exec -ti nginx bash
root@nginx:/# curl "http://metadata.google.internal/computeMetadata/v1/instance/disks/" -H "Metadata-Flavor: Google"  #卡住无法访问
```

