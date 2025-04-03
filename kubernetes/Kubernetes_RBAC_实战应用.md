#  Kubernetes RBAC 实战应用




## 1. 介绍
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/483ea073b627cb137b1908d1b7969547.png)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6f2986587b5e22f122d294b0ed1cd397.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/bb38d286857006a8d42332b4a7865f27.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c9a6d1cd198405a903898c97db6f961d.png)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/09a2e7b464e9772e7e4c30da3c3a78cc.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/5c42fb0e2bb4cb04483f97a4fe72b0e3.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e386b377b8b2425f02b951b2c6fa6841.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/5d69c34e5f145e763dc336676843e642.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/7e76cd44fe799f220e8516d6460a052b.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/dc1dc83593484e1e6742c9c5cb56ff2f.png)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c164a2a2ce6fd166c3b02b9f237c519b.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/aa46de7fb75e2ed673096a2de4f5d5e4.png)
## 2. Practice - Role and Rolebinding
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/2a9b80090b576f24a9f80eb9631960d8.png)

```c
root@master:~/k8slib# k create ns red
namespace/red created
root@master:~/k8slib# k create ns blue
namespace/blue created


root@master:~/k8slib# k -n red create role secret-manager --verb=get --resource=secrets -oyaml --dry-run=client
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  creationTimestamp: null
  name: secret-manager
  namespace: red
rules:
- apiGroups:
  - ""
  resources:
  - secrets
  verbs:
  - get


root@master:~/k8slib# k -n red create role secret-manager --verb=get --resource=secrets 
role.rbac.authorization.k8s.io/secret-manager created

root@master:~/k8slib# k -n red create rolebinding secret-manager --role=secret-manager --user=jane -oyaml --dry-run=client
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  creationTimestamp: null
  name: secret-manager
  namespace: red
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: secret-manager
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: User
  name: jane
root@master:~/k8slib# k -n red create rolebinding secret-manager --role=secret-manager --user=jane
rolebinding.rbac.authorization.k8s.io/secret-manager created



root@master:~/k8slib# k -n blue create role secret-manager --verb=get --verb=list --resource=secrets 
role.rbac.authorization.k8s.io/secret-manager created
root@master:~/k8slib# k -n blue create rolebinding secret-manager --role=secret-manager --user=jane
rolebinding.rbac.authorization.k8s.io/secret-manager created


#测试
root@master:~/k8slib# k -n red auth can-i get secrets --as jane
yes
root@master:~/k8slib# k -n red auth can-i get secrets --as tom
no
root@master:~/k8slib# k -n red auth can-i delete secrets --as jane
no
root@master:~/k8slib# k -n red auth can-i list secrets --as jane
no
root@master:~/k8slib# k -n blue auth can-i list secrets --as jane
yes
root@master:~/k8slib# k -n blue auth can-i get secrets --as jane
yes
root@master:~/k8slib# k -n blue auth can-i get pods --as jane
no

```
## 3. Practice - ClusterRole and ClusterRoleBinding
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8e2d33682488cf0d818bb2623321089f.png)

```c
root@master:~/k8slib# k create clusterrole deploy-deleter --verb delete --resource deployments
clusterrole.rbac.authorization.k8s.io/deploy-deleter created
root@master:~/k8slib# k create clusterrolebinding deploy-deleter --user jane --clusterrole deploy-deleter
clusterrolebinding.rbac.authorization.k8s.io/deploy-deleter created
root@master:~/k8slib# k -n red create rolebinding deploy-deleter  --user jim --clusterrole deploy-deleter
rolebinding.rbac.authorization.k8s.io/deploy-deleter created
root@master:~/k8slib# k auth can-i delete deployments --as jane
yes
root@master:~/k8slib# k auth can-i delete deployments --as jane -n default
yes
root@master:~/k8slib# k auth can-i delete deployments --as jane -n red
yes
root@master:~/k8slib# k auth can-i delete pods --as jane -n red
no
root@master:~/k8slib# k auth can-i delete deployments --as jim -n default
no
root@master:~/k8slib# k auth can-i delete deployments --as jim -A
no
root@master:~/k8slib# k auth can-i delete deployments --as jim -n red
yes

```
## 4. Accounts and Users
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/9dc19f999d43d5a6139010dcd79d7d0d.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/9e455ac254fef781d53b3cbed6714784.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4658b2ddc8b41f5bc6f82e5e0a1c9ebe.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a9825f17cf980c8a4bcf0e2b1da95e63.png)
## 5. Practice - CertificateSigningRequests
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c9805878587fde045b64817dbbd6abb9.png)

```c
root@master:~/cks/RBAC# openssl genrsa -out jane.key 2048
Generating RSA private key, 2048 bit long modulus
..................................+++
............................................................+++
e is 65537 (0x10001)
root@master:~/cks/RBAC# openssl req -new -key jane.key -out jane.csr
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:
State or Province Name (full name) [Some-State]:
Locality Name (eg, city) []:
Organization Name (eg, company) [Internet Widgits Pty Ltd]:
Organizational Unit Name (eg, section) []:
Common Name (e.g. server FQDN or YOUR name) []:jane
Email Address []:

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:
root@master:~/cks/RBAC# ls
jane.csr  jane.key
```

参考链接：
[https://kubernetes.io/docs/reference/access-authn-authz/certificate-signing-requests/#create-certificatesigningrequest](https://kubernetes.io/docs/reference/access-authn-authz/certificate-signing-requests/#create-certificatesigningrequest)

```c
root@master:~/cks/RBAC# vim csr.yaml
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: jane
spec:
  groups:
  - system:authenticated
  request: todo 
  signerName: kubernetes.io/kube-apiserver-client
  usages:
  - client auth


root@master:~/cks/RBAC# cat jane.csr | base64 -w 0
LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlJQ21UQ0NBWUVDQVFBd1ZERUxNQWtHQTFVRUJoTUNRVlV4RXpBUkJnTlZCQWdNQ2xOdmJXVXRVM1JoZEdVeApJVEFmQmdOVkJBb01HRWx1ZEdWeWJtVjBJRmRwWkdkcGRITWdVSFI1SUV4MFpERU5NQXNHQTFVRUF3d0VhbUZ1ClpUQ0NBU0l3RFFZSktvWklodmNOQVFFQkJRQURnZ0VQQURDQ0FRb0NnZ0VCQU83Z2lEUDNxU3U2Qll3SVVkeDgKU2RERUxOd1c3emVvbkZXZWlxNGFHOWhoMzV6TWFYc09wbjE4V0tmU1pYcjMwU0dVUVcwZjdwYUdKOXlTVVdRNwpnUGw2ejU0VjlpZDdxOWswTDMvcnlNMmJOdEdOQnB0QUJYMjVoOXZaSHpiUzQ0ekZkRlA0T2x0b2orcFpLVnpoCm56T2xBNUZwTmxKRGtzeHRBYVJsYzhwa2liZGpObVVJVGNCT3dkRFp5M1JJUEhsU0FQam10QlJqaUNXQVdYZFUKU01PRXRRUUdkVFFreGo3TDJxbSsyVkhGVUFHOEFKWkMwaFFHQStoTlZMNis2Uk5YTFlrWDR5VnVwMlBNWG5YVQorRUpUVnArZFJjU0FwUXlPbFNITTFpNU5sOWdPdEN4S2VZd245R3VlZnpVNlB5cEFrVitINnRlNHVSKzZYcWpUCkdzc0NBd0VBQWFBQU1BMEdDU3FHU0liM0RRRUJDd1VBQTRJQkFRQWkzaVp5TnlYRFdsMVdJQjRta3BpczcwazAKeHl4VW50ZDg1UTJERzJ6K0h1b0YvQlhNZ2hvTXpyWGtndVNraFpRYnRhS1JRMGtOSjNYYUlUQzNGeWN4OW0yVgp3blZZS3VoYnRPTXVlRzJMWngvUTFwNjJZYTJyc1NPbGhyRlhqY1RkUUk0dllpSGNMUE0vMi9FTGFUQU9mQ1dzClUvN0FpNGhQRlk3S0JOVkt4b013VHBFbzgxNXBtMEJtU1orWUZGSU5vQlV6VEI2VUxUd2Yva2FFRkRuc0FtcVEKOVNPRzNvRkx4RmUyVnNwcjhsQ0NOaHV2M21ZZkZlL0JOS3ZvMUdHVDN0SjcwVldLODJZUUdpWjNSbExHaXBKTgpQWXczNmdTdjAydVNxYm9wNWxvSzdUdHhPaU9HSnZJdjVNRnR0OTdVNjlSaHd4dmNHbk5jQXkycVdnRzcKLS0tLS1FTkQgQ0VSVElGSUNBVEUgUkVRVUVTVC0tLS0tCg==root@master:~/cks/RBAC# 




root@master:~/cks/RBAC# vim csr.yaml 
root@master:~/cks/RBAC# cat csr.yaml 
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: jane
spec:
  groups:
  - system:authenticated
  request: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlJQ21UQ0NBWUVDQVFBd1ZERUxNQWtHQTFVRUJoTUNRVlV4RXpBUkJnTlZCQWdNQ2xOdmJXVXRVM1JoZEdVeApJVEFmQmdOVkJBb01HRWx1ZEdWeWJtVjBJRmRwWkdkcGRITWdVSFI1SUV4MFpERU5NQXNHQTFVRUF3d0VhbUZ1ClpUQ0NBU0l3RFFZSktvWklodmNOQVFFQkJRQURnZ0VQQURDQ0FRb0NnZ0VCQU83Z2lEUDNxU3U2Qll3SVVkeDgKU2RERUxOd1c3emVvbkZXZWlxNGFHOWhoMzV6TWFYc09wbjE4V0tmU1pYcjMwU0dVUVcwZjdwYUdKOXlTVVdRNwpnUGw2ejU0VjlpZDdxOWswTDMvcnlNMmJOdEdOQnB0QUJYMjVoOXZaSHpiUzQ0ekZkRlA0T2x0b2orcFpLVnpoCm56T2xBNUZwTmxKRGtzeHRBYVJsYzhwa2liZGpObVVJVGNCT3dkRFp5M1JJUEhsU0FQam10QlJqaUNXQVdYZFUKU01PRXRRUUdkVFFreGo3TDJxbSsyVkhGVUFHOEFKWkMwaFFHQStoTlZMNis2Uk5YTFlrWDR5VnVwMlBNWG5YVQorRUpUVnArZFJjU0FwUXlPbFNITTFpNU5sOWdPdEN4S2VZd245R3VlZnpVNlB5cEFrVitINnRlNHVSKzZYcWpUCkdzc0NBd0VBQWFBQU1BMEdDU3FHU0liM0RRRUJDd1VBQTRJQkFRQWkzaVp5TnlYRFdsMVdJQjRta3BpczcwazAKeHl4VW50ZDg1UTJERzJ6K0h1b0YvQlhNZ2hvTXpyWGtndVNraFpRYnRhS1JRMGtOSjNYYUlUQzNGeWN4OW0yVgp3blZZS3VoYnRPTXVlRzJMWngvUTFwNjJZYTJyc1NPbGhyRlhqY1RkUUk0dllpSGNMUE0vMi9FTGFUQU9mQ1dzClUvN0FpNGhQRlk3S0JOVkt4b013VHBFbzgxNXBtMEJtU1orWUZGSU5vQlV6VEI2VUxUd2Yva2FFRkRuc0FtcVEKOVNPRzNvRkx4RmUyVnNwcjhsQ0NOaHV2M21ZZkZlL0JOS3ZvMUdHVDN0SjcwVldLODJZUUdpWjNSbExHaXBKTgpQWXczNmdTdjAydVNxYm9wNWxvSzdUdHhPaU9HSnZJdjVNRnR0OTdVNjlSaHd4dmNHbk5jQXkycVdnRzcKLS0tLS1FTkQgQ0VSVElGSUNBVEUgUkVRVUVTVC0tLS0tCg== 
  signerName: kubernetes.io/kube-apiserver-client
  usages:
  - client auth




root@master:~/cks/RBAC# k create -f csr.yaml 
certificatesigningrequest.certificates.k8s.io/jane created
root@master:~/cks/RBAC# k get csr
NAME   AGE   SIGNERNAME                            REQUESTOR          CONDITION
jane   7s    kubernetes.io/kube-apiserver-client   kubernetes-admin   Pending
root@master:~/cks/RBAC# k certificate approve jane
certificatesigningrequest.certificates.k8s.io/jane approved
root@master:~/cks/RBAC# k get csr
NAME   AGE   SIGNERNAME                            REQUESTOR          CONDITION
jane   73s   kubernetes.io/kube-apiserver-client   kubernetes-admin   Approved,Issued

root@master:~/cks/RBAC# k get csr -o yaml
....
status:
    certificate: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURPakNDQWlLZ0F3SUJBZ0lSQU5OZmhpSEswMjcrbVZ6UU5ZUGZyT0F3RFFZSktvWklodmNOQVFFTEJRQXcKRlRFVE1CRUdBMVVFQXhNS2EzVmlaWEp1WlhSbGN6QWVGdzB5TVRBME1qVXdPRE0zTXpOYUZ3MHlNakEwTWpVdwpPRE0zTXpOYU1GUXhDekFKQmdOVkJBWVRBa0ZWTVJNd0VRWURWUVFJRXdwVGIyMWxMVk4wWVhSbE1TRXdId1lEClZRUUtFeGhKYm5SbGNtNWxkQ0JYYVdSbmFYUnpJRkIwZVNCTWRHUXhEVEFMQmdOVkJBTVRCR3BoYm1Vd2dnRWkKTUEwR0NTcUdTSWIzRFFFQkFRVUFBNElCRHdBd2dnRUtBb0lCQVFEdTRJZ3o5NmtydWdXTUNGSGNmRW5ReEN6YwpGdTgzcUp4Vm5vcXVHaHZZWWQrY3pHbDdEcVo5ZkZpbjBtVjY5OUVobEVGdEgrNldoaWZja2xGa080RDVlcytlCkZmWW5lNnZaTkM5LzY4ak5temJSalFhYlFBVjl1WWZiMlI4MjB1T014WFJUK0RwYmFJL3FXU2xjNFo4enBRT1IKYVRaU1E1TE1iUUdrWlhQS1pJbTNZelpsQ0UzQVRzSFEyY3QwU0R4NVVnRDQ1clFVWTRnbGdGbDNWRWpEaExVRQpCblUwSk1ZK3k5cXB2dGxSeFZBQnZBQ1dRdElVQmdQb1RWUyt2dWtUVnkySkYrTWxicWRqekY1MTFQaENVMWFmCm5VWEVnS1VNanBVaHpOWXVUWmZZRHJRc1NubU1KL1Jybm44MU9qOHFRSkZmaCtyWHVMa2Z1bDZvMHhyTEFnTUIKQUFHalJqQkVNQk1HQTFVZEpRUU1NQW9HQ0NzR0FRVUZCd01DTUF3R0ExVWRFd0VCL3dRQ01BQXdId1lEVlIwagpCQmd3Rm9BVUs0dktoTHROQ1lpNHZHSVQzVFJBYWgyOWl4OHdEUVlKS29aSWh2Y05BUUVMQlFBRGdnRUJBTDhYCkdONzAwc2RZb0gvMS96Q0VYSjZwanNUZk16eXBTZDc4MVg5SERSNVh6T0pVWFZ0WDFBRTNqcjQ3M251cGE0OXUKOWhuSG5DMWFEODVnbFhiNE9wR2ttWUw5M241TEZSLzU2eng1ZDdwUDlHSzNtT0JYc2IvZGJ4ZEFlMzZJVG1Ucgo4ajdlcTVEcGZQcUpOWlozRDBCK2ljc1htU0NUNDdzNTRQUWtON3cwUHBvNDFkWS8xYTFndnYxNUJPQXc3bVluCmtNUXlubFZMN0cyQ25SaTlnM2R5S2tueUcyOGkyWmFOOGs5MkJXdmNmM1JTR3FudFM4SUxPYi9jWUpYemY1ZWcKQkMyVHhjeGo2dHJJVkVIV0xuZ2NEZmVhbnZ4Ylgwbk41cGlQYVYzaFpRcWo0aFVQZ201RzJQWmNUWUV4Mk4wZgpCckhOWnZRZElPY2RDRVorY2hrPQotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==
....




root@master:~/cks/RBAC# echo LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURPakNDQWlLZ0F3SUJBZ0lSQU5OZmhpSEswMjcrbVZ6UU5ZUGZyT0F3RFFZSktvWklodmNOQVFFTEJRQXcKRlRFVE1CRUdBMVVFQXhNS2EzVmlaWEp1WlhSbGN6QWVGdzB5TVRBME1qVXdPRE0zTXpOYUZ3MHlNakEwTWpVdwpPRE0zTXpOYU1GUXhDekFKQmdOVkJBWVRBa0ZWTVJNd0VRWURWUVFJRXdwVGIyMWxMVk4wWVhSbE1TRXdId1lEClZRUUtFeGhKYm5SbGNtNWxkQ0JYYVdSbmFYUnpJRkIwZVNCTWRHUXhEVEFMQmdOVkJBTVRCR3BoYm1Vd2dnRWkKTUEwR0NTcUdTSWIzRFFFQkFRVUFBNElCRHdBd2dnRUtBb0lCQVFEdTRJZ3o5NmtydWdXTUNGSGNmRW5ReEN6YwpGdTgzcUp4Vm5vcXVHaHZZWWQrY3pHbDdEcVo5ZkZpbjBtVjY5OUVobEVGdEgrNldoaWZja2xGa080RDVlcytlCkZmWW5lNnZaTkM5LzY4ak5temJSalFhYlFBVjl1WWZiMlI4MjB1T014WFJUK0RwYmFJL3FXU2xjNFo4enBRT1IKYVRaU1E1TE1iUUdrWlhQS1pJbTNZelpsQ0UzQVRzSFEyY3QwU0R4NVVnRDQ1clFVWTRnbGdGbDNWRWpEaExVRQpCblUwSk1ZK3k5cXB2dGxSeFZBQnZBQ1dRdElVQmdQb1RWUyt2dWtUVnkySkYrTWxicWRqekY1MTFQaENVMWFmCm5VWEVnS1VNanBVaHpOWXVUWmZZRHJRc1NubU1KL1Jybm44MU9qOHFRSkZmaCtyWHVMa2Z1bDZvMHhyTEFnTUIKQUFHalJqQkVNQk1HQTFVZEpRUU1NQW9HQ0NzR0FRVUZCd01DTUF3R0ExVWRFd0VCL3dRQ01BQXdId1lEVlIwagpCQmd3Rm9BVUs0dktoTHROQ1lpNHZHSVQzVFJBYWgyOWl4OHdEUVlKS29aSWh2Y05BUUVMQlFBRGdnRUJBTDhYCkdONzAwc2RZb0gvMS96Q0VYSjZwanNUZk16eXBTZDc4MVg5SERSNVh6T0pVWFZ0WDFBRTNqcjQ3M251cGE0OXUKOWhuSG5DMWFEODVnbFhiNE9wR2ttWUw5M241TEZSLzU2eng1ZDdwUDlHSzNtT0JYc2IvZGJ4ZEFlMzZJVG1Ucgo4ajdlcTVEcGZQcUpOWlozRDBCK2ljc1htU0NUNDdzNTRQUWtON3cwUHBvNDFkWS8xYTFndnYxNUJPQXc3bVluCmtNUXlubFZMN0cyQ25SaTlnM2R5S2tueUcyOGkyWmFOOGs5MkJXdmNmM1JTR3FudFM4SUxPYi9jWUpYemY1ZWcKQkMyVHhjeGo2dHJJVkVIV0xuZ2NEZmVhbnZ4Ylgwbk41cGlQYVYzaFpRcWo0aFVQZ201RzJQWmNUWUV4Mk4wZgpCckhOWnZRZElPY2RDRVorY2hrPQotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg== | base64 -d
-----BEGIN CERTIFICATE-----
MIIDOjCCAiKgAwIBAgIRANNfhiHK027+mVzQNYPfrOAwDQYJKoZIhvcNAQELBQAw
FTETMBEGA1UEAxMKa3ViZXJuZXRlczAeFw0yMTA0MjUwODM3MzNaFw0yMjA0MjUw
ODM3MzNaMFQxCzAJBgNVBAYTAkFVMRMwEQYDVQQIEwpTb21lLVN0YXRlMSEwHwYD
VQQKExhJbnRlcm5ldCBXaWRnaXRzIFB0eSBMdGQxDTALBgNVBAMTBGphbmUwggEi
MA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQDu4Igz96krugWMCFHcfEnQxCzc
Fu83qJxVnoquGhvYYd+czGl7DqZ9fFin0mV699EhlEFtH+6WhifcklFkO4D5es+e
FfYne6vZNC9/68jNmzbRjQabQAV9uYfb2R820uOMxXRT+DpbaI/qWSlc4Z8zpQOR
aTZSQ5LMbQGkZXPKZIm3YzZlCE3ATsHQ2ct0SDx5UgD45rQUY4glgFl3VEjDhLUE
BnU0JMY+y9qpvtlRxVABvACWQtIUBgPoTVS+vukTVy2JF+MlbqdjzF511PhCU1af
nUXEgKUMjpUhzNYuTZfYDrQsSnmMJ/Rrnn81Oj8qQJFfh+rXuLkful6o0xrLAgMB
AAGjRjBEMBMGA1UdJQQMMAoGCCsGAQUFBwMCMAwGA1UdEwEB/wQCMAAwHwYDVR0j
BBgwFoAUK4vKhLtNCYi4vGIT3TRAah29ix8wDQYJKoZIhvcNAQELBQADggEBAL8X
GN700sdYoH/1/zCEXJ6pjsTfMzypSd781X9HDR5XzOJUXVtX1AE3jr473nupa49u
9hnHnC1aD85glXb4OpGkmYL93n5LFR/56zx5d7pP9GK3mOBXsb/dbxdAe36ITmTr
8j7eq5DpfPqJNZZ3D0B+icsXmSCT47s54PQkN7w0Ppo41dY/1a1gvv15BOAw7mYn
kMQynlVL7G2CnRi9g3dyKknyG28i2ZaN8k92BWvcf3RSGqntS8ILOb/cYJXzf5eg
BC2Txcxj6trIVEHWLngcDfeanvxbX0nN5piPaV3hZQqj4hUPgm5G2PZcTYEx2N0f
BrHNZvQdIOcdCEZ+chk=
-----END CERTIFICATE-----




root@master:~/cks/RBAC# k config view
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://192.168.211.40:6443
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
    client-certificate-data: REDACTED
    client-key-data: REDACTED


root@master:~/cks/RBAC# k config view -o yaml > view.yaml
root@master:~/cks/RBAC# k config set-credentials jane --client-key=jane.key --client-certificate=jane.crt
User "jane" set.
root@master:~/cks/RBAC# k config view
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://192.168.211.40:6443
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
- name: jane
  user:
    client-certificate: /root/cks/RBAC/jane.crt
    client-key: /root/cks/RBAC/jane.key
- name: kubernetes-admin
  user:
    client-certificate-data: REDACTED
    client-key-data: REDACTED



root@master:~/cks/RBAC# k config set-credentials jane --client-key=jane.key --client-certificate=jane.crt --embed-certs
User "jane" set.
root@master:~/cks/RBAC# k config view
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://192.168.211.40:6443
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
- name: jane
  user:
    client-certificate-data: REDACTED
    client-key-data: REDACTED
- name: kubernetes-admin
  user:
    client-certificate-data: REDACTED
    client-key-data: REDACTED




root@master:~/cks/RBAC# k config view --raw
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUM1ekNDQWMrZ0F3SUJBZ0lCQURBTkJna3Foa2lHOXcwQkFRc0ZBREFWTVJNd0VRWURWUVFERXdwcmRXSmwKY201bGRHVnpNQjRYRFRJeE1ERXhPVEF6TWpjME5Gb1hEVE14TURFeE56QXpNamMwTkZvd0ZURVRNQkVHQTFVRQpBeE1LYTNWaVpYSnVaWFJsY3pDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRGdnRVBBRENDQVFvQ2dnRUJBTWdPClppYVBnYXJlekxVWGV3TGVnUGJiVUtFb2kwUW5MdG5xbVhJOVk0VXpEOUp1cEtNd3FURDVQK3BLRldjVXZrOHYKd1BzYk9hcVhaMm1rajFZTnkrRXFGSmJ2d1Y4M0l1VEkreHFlcEhXa0EzRUhkb1VYVjJwNjA3S0dtZE11V2ZISwplWTlXTHlVSEF5QTFBeDJFTVlid2VtQUhBb2tYaUdvVkU1TTlEaGJRY1dCblVqM3hFWUJYLzEwM1JvSEQxYWtvCmJFbng2dGpQZWluSHJmaEJoVFYydmRGMG9BNUtkQWYyUHByYzlDSVRwTWZnczRUWW1kd3YyTzRaQk1NVDdpbnkKbVB0SmpYVUpobG8zM3JzdkJ2WGZLUjQxZXlXVkFCRThvRE0rUWdHL2U5MCtua0IxNVNPWGtLTFFZWGxIbFg3MgpPalNzZ1EvOG13U3cyekQrdDhzQ0F3RUFBYU5DTUVBd0RnWURWUjBQQVFIL0JBUURBZ0trTUE4R0ExVWRFd0VCCi93UUZNQU1CQWY4d0hRWURWUjBPQkJZRUZDdUx5b1M3VFFtSXVMeGlFOTAwUUdvZHZZc2ZNQTBHQ1NxR1NJYjMKRFFFQkN3VUFBNElCQVFDRVYxazhpMGZtWlBFQzdpTk5IMzMwbXErUnRyWnJyeVRPUFAycXZkOWRTVVV2QWo1SwowbWlYZmVKUGw5ZFpodHhkTGZ5RmVQa2t2SFZ5eVJtTnh2Z3R2Sk1laEdrQXFYeTZEanFmSE4zZUxWOUdjN2daCkJYdlRTMjRIMzY2cmhrVHQydWM3ZXBSTkpQZThqWG14RE8zVHZoaTc4Tll4c1ZIb1VjekQ3Nk83UmV1c3dyWnUKN1NWclhHN0Flay9RVlgxSGR6Q0c3NndCTEdzZ2VOb3B4eVViMmJ6eU80bUI4eDU0OWNBaTBjVkVqZlRoMDRnbwoyQ09pdVZiWEMzOGpQd090bnVFbHNYaW9ZSHYrcHI5RysxREZ4R2ZhY0NtWFFlT2ZYdlNmOEFFeGFLclZIOWYxCkhJQTd4TzQ2TXJ4WmswRXdDQ2d0cERPV25HM2s2d29XN1MyUAotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==
    server: https://192.168.211.40:6443
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
- name: jane
  user:
    client-certificate-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURPakNDQWlLZ0F3SUJBZ0lSQU5OZmhpSEswMjcrbVZ6UU5ZUGZyT0F3RFFZSktvWklodmNOQVFFTEJRQXcKRlRFVE1CRUdBMVVFQXhNS2EzVmlaWEp1WlhSbGN6QWVGdzB5TVRBME1qVXdPRE0zTXpOYUZ3MHlNakEwTWpVdwpPRE0zTXpOYU1GUXhDekFKQmdOVkJBWVRBa0ZWTVJNd0VRWURWUVFJRXdwVGIyMWxMVk4wWVhSbE1TRXdId1lEClZRUUtFeGhKYm5SbGNtNWxkQ0JYYVdSbmFYUnpJRkIwZVNCTWRHUXhEVEFMQmdOVkJBTVRCR3BoYm1Vd2dnRWkKTUEwR0NTcUdTSWIzRFFFQkFRVUFBNElCRHdBd2dnRUtBb0lCQVFEdTRJZ3o5NmtydWdXTUNGSGNmRW5ReEN6YwpGdTgzcUp4Vm5vcXVHaHZZWWQrY3pHbDdEcVo5ZkZpbjBtVjY5OUVobEVGdEgrNldoaWZja2xGa080RDVlcytlCkZmWW5lNnZaTkM5LzY4ak5temJSalFhYlFBVjl1WWZiMlI4MjB1T014WFJUK0RwYmFJL3FXU2xjNFo4enBRT1IKYVRaU1E1TE1iUUdrWlhQS1pJbTNZelpsQ0UzQVRzSFEyY3QwU0R4NVVnRDQ1clFVWTRnbGdGbDNWRWpEaExVRQpCblUwSk1ZK3k5cXB2dGxSeFZBQnZBQ1dRdElVQmdQb1RWUyt2dWtUVnkySkYrTWxicWRqekY1MTFQaENVMWFmCm5VWEVnS1VNanBVaHpOWXVUWmZZRHJRc1NubU1KL1Jybm44MU9qOHFRSkZmaCtyWHVMa2Z1bDZvMHhyTEFnTUIKQUFHalJqQkVNQk1HQTFVZEpRUU1NQW9HQ0NzR0FRVUZCd01DTUF3R0ExVWRFd0VCL3dRQ01BQXdId1lEVlIwagpCQmd3Rm9BVUs0dktoTHROQ1lpNHZHSVQzVFJBYWgyOWl4OHdEUVlKS29aSWh2Y05BUUVMQlFBRGdnRUJBTDhYCkdONzAwc2RZb0gvMS96Q0VYSjZwanNUZk16eXBTZDc4MVg5SERSNVh6T0pVWFZ0WDFBRTNqcjQ3M251cGE0OXUKOWhuSG5DMWFEODVnbFhiNE9wR2ttWUw5M241TEZSLzU2eng1ZDdwUDlHSzNtT0JYc2IvZGJ4ZEFlMzZJVG1Ucgo4ajdlcTVEcGZQcUpOWlozRDBCK2ljc1htU0NUNDdzNTRQUWtON3cwUHBvNDFkWS8xYTFndnYxNUJPQXc3bVluCmtNUXlubFZMN0cyQ25SaTlnM2R5S2tueUcyOGkyWmFOOGs5MkJXdmNmM1JTR3FudFM4SUxPYi9jWUpYemY1ZWcKQkMyVHhjeGo2dHJJVkVIV0xuZ2NEZmVhbnZ4Ylgwbk41cGlQYVYzaFpRcWo0aFVQZ201RzJQWmNUWUV4Mk4wZgpCckhOWnZRZElPY2RDRVorY2hrPQotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==
    client-key-data: LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQpNSUlFb3dJQkFBS0NBUUVBN3VDSU0vZXBLN29GakFoUjNIeEowTVFzM0Jidk42aWNWWjZLcmhvYjJHSGZuTXhwCmV3Nm1mWHhZcDlKbGV2ZlJJWlJCYlIvdWxvWW4zSkpSWkR1QStYclBuaFgySjN1cjJUUXZmK3ZJelpzMjBZMEcKbTBBRmZibUgyOWtmTnRMampNVjBVL2c2VzJpUDZsa3BYT0dmTTZVRGtXazJVa09TekcwQnBHVnp5bVNKdDJNMgpaUWhOd0U3QjBObkxkRWc4ZVZJQStPYTBGR09JSllCWmQxUkl3NFMxQkFaMU5DVEdQc3ZhcWI3WlVjVlFBYndBCmxrTFNGQVlENkUxVXZyN3BFMWN0aVJmakpXNm5ZOHhlZGRUNFFsTlduNTFGeElDbERJNlZJY3pXTGsyWDJBNjAKTEVwNWpDZjBhNTUvTlRvL0trQ1JYNGZxMTdpNUg3cGVxTk1heXdJREFRQUJBb0lCQVFDbW5kWmk2UXdHZytuNgprcE1HeDJwMVEyQkc0M2hYeWpQQlJLUldhNytnWGlRcXFpbW91NzlGSjhadXlFSWdVMXA3b1gxQk1GU3FpVWlrCmdTcGtUMXpXcHVMSjBXZXdnb0tMTGVzenZyS0JOeEkxZDdoejhXUGpIZFcxY3V4aXdSWVd5bU1wYnFyRnQxa3EKaktaZE1zSm9zMkNadkZrM2FBcXNyQnZKSHpwMG4vS3MybzhRNVpoYkpEa0tUY2w1QndoUFBUSHBiRlluSEJUeApHQkxtRWg2YlRueWpSRVpnaUxBdy8xbUFuRWk0dW00ODJJZS9FSjdiMHJvMTcxYjBCOGtkNWdGWUsyUC9sNXRiClFvZG1FckFXWURjazRJeHVlQy92YnFpNUQ0TndSNUVFWUJzWXc2cFFyWlhpTFVBeW53dmsvL21zRUJCY0txN3gKUHp1MHdMMEJBb0dCQVByRDdNS2k5VHdoNCtvbmo1eDdoNm9nTHE5VHFFRzNqVkV6dE03aXl3S3hJTW5UVnptVQpCSThyd1U0cXZvM0F6ZXM1eVhJL05SWkFNV2NTZUtpSVpEQU80d3dPcklXVGcxY056c3JiazdDTXp5UUQ4d2Z4CkVLMEpsdHNYaWw1UzdvTmx6d0dscTZ1UWVZTXhja0IrQ2FCU1BSWlJhSk5yUTFiVXp4STJkTDhUQW9HQkFQUGQKRTdzYjFRSzloMzRRcDdXYjYwNDJzdkxPajdocC93RzhPbk1HVGttcm9WZkg3b2QrV2FYRmNrQ0RBRDR2NkRZbApYZmRRWUVWajU5Z0Q3Rzl4RFdqNEFPZFZ5aGxSeVhpN1FXUnNKRTJwYXdoZ050TFgremtjZzBqcWZsZjFlV3FVCk05OWVOelJnY1RTbjY3ZHAzWkRBMUVROVNocFZveHdReHFyZUY5UnBBb0dBYm9UelVFVXArRHFuakllckQ3aVIKN2pVSTNsVHNqeW9xcW1NemlRc0Rsa2dpdjFEWjNKS1QvOVcwK0pKMk1WdU1aZU91R1NBcWNZZ1JQZkF5SlhVWApVdWI4d2srbFVhblY5UVFzNDlNcW9HRXUyaHl6ZkFpTzVQU1kvQzYvMlJxTDdIVnVhcmR0bGN1ekFsTkVtNC94CkJpdTRxS0Z3aWFoNG9VaGhpeEZkR3VrQ2dZQjV2WHNWSkk3UllHYWNvNW5seXVITVdQZzZ5SzNzNVZWOXUwYisKbHo1TC90ZDc1LzZITzZkclgwZHJOenJPME1HL0RpWjd5VzlXRk1yd0J2MW9vT3FONVlrbDg2a0J2TmUwWXQ4QgpVQTlMaWZFNTdEWlNTYXBMMTVVZXVKbThOWHFZbjBYS0U5SEJYd2dFdm5PcFM3dGxnUzQycHRZd2tXSHRKOTdWCi9DdXZTUUtCZ0JibVVKVmJMSlFOamcxZzNpMHZSUEJCakhkV0VETzMvbU1JQXdxREJMY0FUZ1hZWm5RWmZkQWgKaUJzZXJvaWl0M0FuV1FXTi8rZEhzMFhtdlcxbFUzbEJJc3Jvd1Y3YjZaSzlHcktxRGUwVS9CemJYUmpVczVibwowcEFJaWxLY0ZFNUdLM3FqSmlLM2RKZ2R5a3ZBYldRUDM0UmJ3YUdldHdFbDRWK1pSRnRNCi0tLS0tRU5EIFJTQSBQUklWQVRFIEtFWS0tLS0tCg==
- name: kubernetes-admin
  user:
    client-certificate-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURFekNDQWZ1Z0F3SUJBZ0lJR2w5NUYxYVh3NEl3RFFZSktvWklodmNOQVFFTEJRQXdGVEVUTUJFR0ExVUUKQXhNS2EzVmlaWEp1WlhSbGN6QWVGdzB5TVRBeE1Ua3dNekkzTkRSYUZ3MHlNakF4TVRrd016STNORFphTURReApGekFWQmdOVkJBb1REbk41YzNSbGJUcHRZWE4wWlhKek1Sa3dGd1lEVlFRREV4QnJkV0psY201bGRHVnpMV0ZrCmJXbHVNSUlCSWpBTkJna3Foa2lHOXcwQkFRRUZBQU9DQVE4QU1JSUJDZ0tDQVFFQW9SSHZVUnBiVkdDb2VqNXoKWFgzYkpxN2FBUTY1amh2YisrNkt2ak1laS9LbWtNSlZWVzEvRWFlVzV3Q3V5aXk4eFQ2Z1FjUjhXNE9YaDk5RwpNcHFvcUhCNEZoNnhlY2xjUjJLVkN1TTVVbk1aZUZDZEJOanQvRnRhVnRkcW90dFJ3eTg1ajh2SmdvL0tRMnFSCkVBTDFJZ3pkTXZ0cjBkQnQ4RjhVUVQ5QVZ2SmJtZzFTelEyeHptSlluQ1EySTFYTnZjV1g4UWlJVGRPdTlUY1YKQjFrMVRjK2x5Sm9mdGlyZmlVVVNBUS82UTlnV05XdEp1d1EwdHVkR2RzZnlobXNCbFV5QkNOSlhEaXRHMUxWawpXTmRFc3d5UVdsVGdZdU9rWkczU3k4RzYyZmlpZDAxb2VqbVFHTExHaVFYcHNWTytyQzdjSnowL2hQdC9GaEgwCmpOWnpZUUlEQVFBQm8wZ3dSakFPQmdOVkhROEJBZjhFQkFNQ0JhQXdFd1lEVlIwbEJBd3dDZ1lJS3dZQkJRVUgKQXdJd0h3WURWUjBqQkJnd0ZvQVVLNHZLaEx0TkNZaTR2R0lUM1RSQWFoMjlpeDh3RFFZSktvWklodmNOQVFFTApCUUFEZ2dFQkFJdE9meXRWT0pOdytTNU1LWWlQWDBCS0lFTGkvcFBHL0NJUUU3Zjg1WlRPSXIvS05JTXcwYURxCmp6M0ZBU0F5bTloVEp4dEtUN3dzeWEvbGcwcnlIMEdGQ0dHTjdDOFNhUVVLNDRxUjd1RnUzRlo1OXNteDc1MU8KQTlOMnYxWlJJa3pQYXVNNGZ3T2NLOFUzN0lkOGVYZkxJMjV3c3p0RXdickdRS1Jia2owMnkzSEpEWWJobjFiUgoyYXpJOU1FaThrMVdtVTk4S1B2bFBFMk9md3BkSFp2TEVJTGdBOGZLZU1WZnduNGlDcVN1L3R1aGJ6RFhqOEFXCkpMcWNyZ1FqT0pxS0hqS2VEWFZIVm14MHFYMnhpRnRUM0prSDdWTFplK1JlUlZ4dEFWekkwc2NoeVFzY04vaXEKY3JNcHQ1NlhPM1BSVWc1bnplWnp5TTE4ajhqL3hIbz0KLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo=
    client-key-data: LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQpNSUlFb2dJQkFBS0NBUUVBb1JIdlVScGJWR0NvZWo1elhYM2JKcTdhQVE2NWpodmIrKzZLdmpNZWkvS21rTUpWClZXMS9FYWVXNXdDdXlpeTh4VDZnUWNSOFc0T1hoOTlHTXBxb3FIQjRGaDZ4ZWNsY1IyS1ZDdU01VW5NWmVGQ2QKQk5qdC9GdGFWdGRxb3R0Und5ODVqOHZKZ28vS1EycVJFQUwxSWd6ZE12dHIwZEJ0OEY4VVFUOUFWdkpibWcxUwp6UTJ4em1KWW5DUTJJMVhOdmNXWDhRaUlUZE91OVRjVkIxazFUYytseUpvZnRpcmZpVVVTQVEvNlE5Z1dOV3RKCnV3UTB0dWRHZHNmeWhtc0JsVXlCQ05KWERpdEcxTFZrV05kRXN3eVFXbFRnWXVPa1pHM1N5OEc2MmZpaWQwMW8KZWptUUdMTEdpUVhwc1ZPK3JDN2NKejAvaFB0L0ZoSDBqTlp6WVFJREFRQUJBb0lCQUgzcHAwdWZid1huQ2MyRwpSR2t4bWNBRHNDaGplbXE5SEpzMVB3Q3d0WkJ4Z0FScDVvdUJyWFAvcnRlbWtQMDdPOVoxdnBHcktBdmlNdkxrCmQ5dlhTMEZocW42Z1A5MFVyQzZod2lGZ3Y4N1VhM1RDai96YUdERE91VEJwOWRLWjRMRFVtZ3J2SS9nTXIvRkQKdldMbTdQcFJWQm9tc1lLempUMzdGYnByMThBZk9ERFoyTVlJekxVVlJENWNkM2VRdzN3Q2hlMnpmelJabEhsMwprRG1FQk9YaWRObnpkL0lmSGxWQk4yN2VQYmtVUjIzd0tSbWY2ejE4aDhqV2FveTJwSVdoVGU0THVBM0h1cWpFCnZYSzVvUUY2aEFTL1ZRc2hYVUZ4OXQ3Z0VFZlB5TnBSTURjeWlRVWF2S01tNUlPYXQreklyRnFTeUhNdURwdjIKcXBDSVFvRUNnWUVBeHdXMERTRUgvc1ltR3k1eUhXRStybkVuaG1teFBTL0lzcDl0NzIyWjZ3VVdoeU42QnowYwo0T3pwalBiQVpiY2x3QTlqY0ZLaS9Pelh4Wnl5WU56WHJVaGJraTBwbHF2VHdJQXpqZW9FR1dFNGpzYWN2V005CnVybCs0MU90U3hMWFVwQTRDWDlCSVdJYkZxamlhZ3pxOGp0OG0zMHI1TzBmT0dXblpTQ1F4SnNDZ1lFQXp5NjQKMkNRWmJ1emg3Z1VlRklBcm5UeVk4U1hWdGJ0Q3Z5aDVUdTNwMU40eFpBOUI1NnpvTFFvMkNpZ2FseWJBRmR1UApLMk0zeDQxcmEyWUJmQTl0QnUxem9qQndIY3hiVWZYWmJvcGZUWkFOT1JEbmY0ZEtrV09LdDJ0K052bFdvcTR3ClYyeFRhaDR3WWdQSFc1eTg3YWwxTkg3bUttNzVOWWVkaVNhZkliTUNnWUJDQUd3enBtNm1XVVF0NDN0SXJ3VkEKaUpvWkExZ1orSXpRWC9ydldpT2ZRekt6WWxxSHFBYTV1UmZDL2RuVVlhYU5TUTBySk55VWtGOEdVKzc4SElFUwpJRnJ0NFRoWGxXaEdBTDRZSkRGejBVQVdhVnQxbTBIUGVORFJ4dUJEYzE0aExWN0lGNEdiOXBNUk1yVFRnckV2CjMvWjFBay9hUGFFSzdQdFVtRFlxWFFLQmdFem01TW1scktNVjNrN0JLNGNraEF2YklGSHlYejhUZ1JUL2F2ZTMKSzZKTnp6dDZ4bFcrUW5mbFlHV291U1g5eGpMV3ltK3FabHYxekRlVEoxM3JRK2JjWUoyRktUaUdVQ2MrQURVZAp1MzVJeC8rMG5Ka2ptTFFhcExTc2U2N2dJaDVFVmNFOWZrRFhiOUlSNFAvS1QvNVBkaWZFS3A3NWpoc21lWDBkCkR0Z3RBb0dBVWI0TDFwZ2VBOXZsTWlzMkh1WUphdFd5Z3A0UDN5ODM3L0ZQMGcxa2pBdm43a2dCM2FWeE01QzUKeEUrWklWbm1WcXJycUtpejVjMVNTbTRSc2p2dUFsK2c4cVJtTG9zNEVKNWxzaThLbTZmUlhiSm5EeUZQSHZsRQpGdHBvRVZUT2FzQ045Ylg0NEtoSzZxcFhxTEhpaEwzU0MyVldibllBYnIxd2FJLzQzSWc9Ci0tLS0tRU5EIFJTQSBQUklWQVRFIEtFWS0tLS0tCg==





root@master:~/cks/RBAC# k config set-context jane --user=jane --cluster=kubernetes
Context "jane" created.
root@master:~/cks/RBAC# k config get-contexts
CURRENT   NAME                          CLUSTER      AUTHINFO           NAMESPACE
          jane                          kubernetes   jane               
*         kubernetes-admin@kubernetes   kubernetes   kubernetes-admin   
root@master:~/cks/RBAC# k config use-context jane
Switched to context "jane".
root@master:~/cks/RBAC# k get ns
Error from server (Forbidden): namespaces is forbidden: User "jane" cannot list resource "namespaces" in API group "" at the cluster scope

root@master:~/cks/RBAC# k auth can-i delete deployments 
yes
root@master:~/cks/RBAC# k auth can-i delete pods
no
```
----


✈<font color=	#FF4500 size=4 style="font-family:Courier New">推荐阅读：</font>

 - [kubernetes RBAC【1】相遇--介绍、常规用法、集群默认](https://blog.csdn.net/xixihahalelehehe/article/details/110784398?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522164734727616780265468496%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fblog.%2522%257D&request_id=164734727616780265468496&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_ecpm_v1~rank_v31_ecpm-1-110784398.nonecase&utm_term=rbac&spm=1018.2226.3001.4450)
 - [Kubernetes RBAC【2】实战应用](https://blog.csdn.net/xixihahalelehehe/article/details/116131268?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522164734727616780265468496%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fblog.%2522%257D&request_id=164734727616780265468496&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_ecpm_v1~rank_v31_ecpm-2-116131268.nonecase&utm_term=rbac&spm=1018.2226.3001.4450)
 - [kubernetes RBAC【3】婚姻--深刻的理解原理与运用过程作用](https://blog.csdn.net/xixihahalelehehe/article/details/119216802?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522164734727616780265468496%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fblog.%2522%257D&request_id=164734727616780265468496&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_ecpm_v1~rank_v31_ecpm-3-119216802.nonecase&utm_term=rbac&spm=1018.2226.3001.4450)
 - [https://thorsten-hans.com/custom-resource-definitions-with-rbac-for-serviceaccounts](https://thorsten-hans.com/custom-resource-definitions-with-rbac-for-serviceaccounts)
 - [https://www.cncf.io/blog/2018/08/01/demystifying-rbac-in-kubernetes/](https://www.cncf.io/blog/2018/08/01/demystifying-rbac-in-kubernetes/)


---
