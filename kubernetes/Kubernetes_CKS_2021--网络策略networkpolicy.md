#  Kubernetes NetworkPolicy 实战
tags: NetworkPolicy
<!--  catalog: ~NetworkPolicy~ -->




## 1. NetworkPolicy 策略
k8s官网： [network policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/)

![在这里插入图片描述](https://img-blog.csdnimg.cn/2021041311404622.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16)

![在这里插入图片描述](https://img-blog.csdnimg.cn/2021041311401950.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210413114213734.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
## 2. Practice - Frontend to Backend traffic
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210413114444451.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210420144613907.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
```C
root@master:~# k run frontend --image=nginx
pod/frontend created
root@master:~# k run backend --image=nginx
pod/backend created
root@master:~# k expose pod frontend --port 80
service/frontend exposed
root@master:~# k expose pod backend --port 80
service/backend exposed
root@master:~# k get pods,svc
NAME           READY   STATUS    RESTARTS   AGE
pod/backend    1/1     Running   0          49s
pod/frontend   1/1     Running   0          58s

NAME                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
service/backend      ClusterIP   10.104.232.138   <none>        80/TCP    14s
service/frontend     ClusterIP   10.111.199.32    <none>        80/TCP    23s
service/kubernetes   ClusterIP   10.96.0.1        <none>        443/TCP   91d

root@master:~# k exec frontend -- curl backend
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   612  100   612    0     0  11547      0 --:--:-- --:--:-- --:--:-- 11547
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>

root@master:~# k exec backend -- curl frontend
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   612  100   612    0     0   119k      0 --:--:-- --:--:-- --:--:--  119k
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>


root@master:~# vim default-deny.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny
  namespace: default
spec:
  podSelector: {}
  policyTypes:
  - Egress
  - Ingress




root@master:~# k create -f default-deny.yaml
networkpolicy.networking.k8s.io/deny created

root@master:~# vim frontend.yaml
# allows frontend pods to communicate with backend pods
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: frontend
  namespace: default
spec:
  podSelector:
    matchLabels:
      run: frontend
  policyTypes:
  - Egress
  egress:
  - to:
    - podSelector:
        matchLabels:
          run: backend

root@master:~# k -f frontend.yaml create
networkpolicy.networking.k8s.io/frontend created
root@master:~# k exec frontend -- curl backend
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:--  0:00:08 --:--:--     0


root@master:~# vim backend.yaml
# allows backend pods to have incoming traffic from frontend pods
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: backend
  namespace: default
spec:
  podSelector:
    matchLabels:
      run: backend
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          run: frontend

root@master:~# k -f backend.yaml create
networkpolicy.networking.k8s.io/backend created
root@master:~# k exec frontend -- curl backend
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:--  0:00:08 --:--:--     0


root@master:~# k get pods --show-labels -owide
NAME       READY   STATUS    RESTARTS   AGE   IP                NODE    NOMINATED NODE   READINESS GATES   LABELS
backend    1/1     Running   0          29m   192.168.104.27    node2   <none>           <none>            run=backend
frontend   1/1     Running   0          30m   192.168.166.179   node1   <none>           <none>            run=frontend

root@master:~# k exec frontend -- curl 192.168.104.27
  % Total    % Received % X<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
ferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   612  100   612    0     0  18545      0 --:--:-- --:--:-- --:--:-- 19125



root@master:~# k exec backend -- curl 192.168.166.179 
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
100   612  100   612    0     0   298k      0 --:--:-- --:--:-- --:--:--  298k

```


## 3. Practice - Backend to Database traffic
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210420145522837.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

```bash
kubectl create ns cassandra
kubectl edit ns cassandra	
```


```bash
apiVersion: v1
kind: Namespace
metadata:
  creationTimestamp: "2021-04-20T07:19:22Z"
  name: cassandra
  resourceVersion: "533198"
  uid: 766ae069-4dc9-4acd-a4db-ce852c293cc6
  labels:  #添加
    ns: cassandra #添加
spec:
  finalizers:
  - kubernetes
status:
  phase: Active
```

```bash
root@master:~# k  -n cassandra run cassandra --image=nginx
pod/cassandra created
root@master:~# k -n cassandra get pod -owide
NAME        READY   STATUS    RESTARTS   AGE   IP               NODE    NOMINATED NODE   READINESS GATES
cassandra   1/1     Running   0          73m   192.168.104.26   node2   <none>           <none>
root@master:~# k exec backend -- curl 192.168.104.26
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   612  100   612    0     0   597k      0 --:--:-- --:--:-- --:--:--  0

```

```bash
vim backend.yaml
```

```bash
# allows backend pods to have incoming traffic from frontend pods and to cassandra namespace
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: backend
  namespace: default
spec:
  podSelector:
    matchLabels:
      run: backend
  policyTypes:
    - Ingress
    - Egress
  ingress:
    - from:
      - podSelector:
          matchLabels:
            run: frontend
  egress:
    - to:
      - namespaceSelector:
          matchLabels:
            ns: cassandra


root@master:~# k apply -f backend.yaml
Warning: resource networkpolicies/backend is missing the kubectl.kubernetes.io/last-applied-configuration annotation which is required by kubectl apply. kubectl apply should only be used on resources created declaratively by either kubectl create --save-config or kubectl apply. The missing annotation will be patched automatically.
networkpolicy.networking.k8s.io/backend configured


root@master:~# k exec backend -- curl 192.168.104.26
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   612  100   612    0     0   597k      0 --:--:-- --:--:-- --:--:--  597k
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>

root@master:~# cat cassandra-deny.yaml
# deny all incoming and outgoing traffic from all pods in namespace cassandra
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: cassandra-deny
  namespace: cassandra
spec:
  podSelector: {}
  policyTypes:
    - Ingress
    - Egress


root@master:~# k create -f cassandra-deny.yaml 
networkpolicy.networking.k8s.io/cassandra-deny created

root@master:~# k exec backend -- curl 192.168.104.26
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:--  0:00:04 --:--:--     0^C


# allows cassandra pods having incoming connection from backend namespace
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: cassandra
  namespace: cassandra
spec:
  podSelector:
    matchLabels:
      run: cassandra
  policyTypes:
    - Ingress
  ingress:
    - from:
      - namespaceSelector:
          matchLabels:
            ns: default


root@master:~# k create -f cassandra.yaml 
networkpolicy.networking.k8s.io/cassandra created
root@master:~# k edit ns default
apiVersion: v1
kind: Namespace
metadata:
  creationTimestamp: "2021-01-19T03:27:58Z"
  labels: #添加
    ns: default   #添加
  name: default
  resourceVersion: "541475"
  uid: 2d566715-f0a4-49b3-b590-dfa7df30d0ba
spec:
  finalizers:
  - kubernetes
status:
  phase: Active



root@master:~# k exec backend -- curl 192.168.104.26
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   612  100   612    0     0   298k      0 --:--:-- --:--:-- --:--:--  597k
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>

```

参考：
 - [k8s networkpolicy 网络策略详解](https://ghostwritten.blog.csdn.net/article/details/108422856)
 - [Kubernetes  NetworkPolicy](https://kubernetes.io/zh/docs/concepts/services-networking/network-policies/)
 - [Get started with Kubernetes network policy](https://projectcalico.docs.tigera.io/security/kubernetes-network-policy)
 - [Network Policy](https://feisky.gitbooks.io/kubernetes/content/concepts/network-policy.html)
 - [Deep Dive into Network Policy](https://networkpolicy.io/)
 - [Kubernetes Network Policies: A Practitioner's Guide](https://loft.sh/blog/kubernetes-network-policies-a-practitioners-guide)

