


---
## 1. 介绍
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210421135110812.png?shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210421135426748.png?shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

## 2. Practice - create an Ingress
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210421135529616.png?shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
部署链接：[https://kubernetes.github.io/ingress-nginx/deploy/#bare-metal](https://kubernetes.github.io/ingress-nginx/deploy/#bare-metal)

```c
root@master:~/cks/nginx-ingress# kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.40.2/deploy/static/provider/baremetal/deploy.yaml
namespace/ingress-nginx created
serviceaccount/ingress-nginx created
configmap/ingress-nginx-controller created
clusterrole.rbac.authorization.k8s.io/ingress-nginx created
clusterrolebinding.rbac.authorization.k8s.io/ingress-nginx created
role.rbac.authorization.k8s.io/ingress-nginx created
rolebinding.rbac.authorization.k8s.io/ingress-nginx created
service/ingress-nginx-controller-admission created
service/ingress-nginx-controller created
deployment.apps/ingress-nginx-controller created
validatingwebhookconfiguration.admissionregistration.k8s.io/ingress-nginx-admission created
serviceaccount/ingress-nginx-admission created
clusterrole.rbac.authorization.k8s.io/ingress-nginx-admission created
clusterrolebinding.rbac.authorization.k8s.io/ingress-nginx-admission created
role.rbac.authorization.k8s.io/ingress-nginx-admission created
rolebinding.rbac.authorization.k8s.io/ingress-nginx-admission created
job.batch/ingress-nginx-admission-create created
job.batch/ingress-nginx-admission-patch created


root@master:~/cks/nginx-ingress# k get pod,svc -n ingress-nginx
NAME                                            READY   STATUS      RESTARTS   AGE
pod/ingress-nginx-admission-create-2zdxl        0/1     Completed   0          41s
pod/ingress-nginx-admission-patch-tnwlh         0/1     Completed   1          41s
pod/ingress-nginx-controller-548df9766d-6m74x   1/1     Running     0          41s

NAME                                         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
service/ingress-nginx-controller             NodePort    10.104.236.124   <none>        80:31459/TCP,443:30640/TCP   42s
service/ingress-nginx-controller-admission   ClusterIP   10.97.93.128     <none>        443/TCP                      42s


```
应用ingress链接：[https://kubernetes.io/zh/docs/concepts/services-networking/ingress/](https://kubernetes.io/zh/docs/concepts/services-networking/ingress/)

```c
root@master:~/cks/nginx-ingress# cat secure-ingress.yaml 
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: secure-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - http:
      paths:
      - path: /service1
        pathType: Prefix
        backend:
          service:
            name: service1
            port:
              number: 80
      - path: /service2
        pathType: Prefix
        backend:
          service:
            name: service2
            port:
              number: 80


root@master:~/cks/nginx-ingress# k create -f secure-ingress.yaml 
ingress.networking.k8s.io/secure-ingress created
root@master:~/cks/nginx-ingress# k get ing
NAME             CLASS    HOSTS   ADDRESS          PORTS   AGE
secure-ingress   <none>   *       192.168.211.41   80      30s

root@master:~/cks/nginx-ingress# k run pod1 --image=nginx
root@master:~/cks/nginx-ingress# k run pod2 --image=httpd

root@master:~/cks/nginx-ingress# k expose pod pod1 --port 80 --name service1
service/service1 exposed
root@master:~/cks/nginx-ingress# k expose pod pod2 --port 80 --name service2
service/service2 exposed

root@master:~/cks/nginx-ingress# curl  http://192.168.211.40:31459/service1
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



root@master:~/cks/nginx-ingress# curl  http://192.168.211.40:31459/service2
<html><body><h1>It works!</h1></body></html>


```

## 3. Practice - Secure an Ingress
参考链接：[https://kubernetes.io/docs/concepts/services-networking/ingress/#tls](https://kubernetes.io/docs/concepts/services-networking/ingress/#tls)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210421154945142.png?shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

```c
root@master:~/cks/nginx-ingress# curl  https://192.168.211.40:32300/service1 -kv
*   Trying 192.168.211.40...
* TCP_NODELAY set
* Connected to 192.168.211.40 (192.168.211.40) port 32300 (#0)
* ALPN, offering h2
* ALPN, offering http/1.1
* successfully set certificate verify locations:
*   CAfile: /etc/ssl/certs/ca-certificates.crt
  CApath: /etc/ssl/certs
* TLSv1.3 (OUT), TLS handshake, Client hello (1):
* TLSv1.3 (IN), TLS handshake, Server hello (2):
* TLSv1.3 (IN), TLS Unknown, Certificate Status (22):
* TLSv1.3 (IN), TLS handshake, Unknown (8):
* TLSv1.3 (IN), TLS Unknown, Certificate Status (22):
* TLSv1.3 (IN), TLS handshake, Certificate (11):
* TLSv1.3 (IN), TLS Unknown, Certificate Status (22):
* TLSv1.3 (IN), TLS handshake, CERT verify (15):
* TLSv1.3 (IN), TLS Unknown, Certificate Status (22):
* TLSv1.3 (IN), TLS handshake, Finished (20):
* TLSv1.3 (OUT), TLS change cipher, Client hello (1):
* TLSv1.3 (OUT), TLS Unknown, Certificate Status (22):
* TLSv1.3 (OUT), TLS handshake, Finished (20):
* SSL connection using TLSv1.3 / TLS_AES_256_GCM_SHA384
* ALPN, server accepted to use h2
* Server certificate:                                   #认证失败，没有tls证书
*  subject: O=Acme Co; CN=Kubernetes Ingress Controller Fake Certificate
*  start date: Apr 21 07:16:50 2021 GMT
*  expire date: Apr 21 07:16:50 2022 GMT
*  issuer: O=Acme Co; CN=Kubernetes Ingress Controller Fake Certificate
*  SSL certificate verify result: unable to get local issuer certificate (20), continuing anyway.
* Using HTTP2, server supports multi-use
* Connection state changed (HTTP/2 confirmed)
* Copying HTTP/2 data in stream buffer to connection buffer after upgrade: len=0
* TLSv1.3 (OUT), TLS Unknown, Unknown (23):
* TLSv1.3 (OUT), TLS Unknown, Unknown (23):
* TLSv1.3 (OUT), TLS Unknown, Unknown (23):
* Using Stream ID: 1 (easy handle 0x558de9977430)
* TLSv1.3 (OUT), TLS Unknown, Unknown (23):
> GET /service1 HTTP/2
> Host: 192.168.211.40:32300
> User-Agent: curl/7.58.0
> Accept: */*
> 
* TLSv1.3 (IN), TLS Unknown, Certificate Status (22):
* TLSv1.3 (IN), TLS handshake, Newsession Ticket (4):
* TLSv1.3 (IN), TLS Unknown, Certificate Status (22):
* TLSv1.3 (IN), TLS handshake, Newsession Ticket (4):
* TLSv1.3 (IN), TLS Unknown, Unknown (23):
* Connection state changed (MAX_CONCURRENT_STREAMS updated)!
* TLSv1.3 (OUT), TLS Unknown, Unknown (23):
* TLSv1.3 (IN), TLS Unknown, Unknown (23):
* TLSv1.3 (IN), TLS Unknown, Unknown (23):
< HTTP/2 200 
< date: Wed, 21 Apr 2021 07:51:52 GMT
< content-type: text/html
< content-length: 612
< last-modified: Tue, 13 Apr 2021 15:13:59 GMT
< etag: "6075b537-264"
< accept-ranges: bytes
< strict-transport-security: max-age=15724800; includeSubDomains
< 
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
* TLSv1.3 (IN), TLS Unknown, Unknown (23):
* Connection #0 to host 192.168.211.40 left intact
```
```c

#创建证书
root@master:~/cks/nginx-ingress# openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -days 365 -nodes
Generating a 4096 bit RSA private key
....++
........................................................................................................................................................................++
writing new private key to 'key.pem'
-----
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
Common Name (e.g. server FQDN or YOUR name) []:secure-ingress.com
Email Address []:
root@master:~/cks/nginx-ingress# ls
cert.pem  deploy_0.40.2.yaml  deploy_0.43.0.yaml  key.pem  secure-ingress.yaml

#创建secret
root@master:~/cks/nginx-ingress# k create secret tls secure-ingress --cert=cert.pem --key=key.pem
secret/secure-ingress created
root@master:~/cks/nginx-ingress# k get sec
error: the server doesn't have a resource type "sec"
root@master:~/cks/nginx-ingress# k get secret
NAME                  TYPE                                  DATA   AGE
default-token-rkqr7   kubernetes.io/service-account-token   3      92d
secure-ingress        kubernetes.io/tls                     2      9s



root@master:~/cks/nginx-ingress# vim secure-ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: secure-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  tls:
  - hosts:
      - secure-ingress.com
    secretName: secure-ingress
  rules:
  - host: secure-ingress.com  
    http:
      paths:
      - path: /service1
        pathType: Prefix
        backend:
          service:
            name: service1
            port:
              number: 80

      - path: /service2
        pathType: Prefix
        backend:
          service:
            name: service2
            port:
              number: 80
root@master:~/cks/nginx-ingress# k apply -f secure-ingress.yaml 
ingress.networking.k8s.io/secure-ingress configured



root@master:~/cks/nginx-ingress# curl  https://secure-ingress.com:32300/service2 -kv --resolv secure-ingress.com:32300:192.168.211.41
* Added secure-ingress.com:32300:192.168.211.41 to DNS cache
* Hostname secure-ingress.com was found in DNS cache
*   Trying 192.168.211.41...
* TCP_NODELAY set
* Connected to secure-ingress.com (192.168.211.41) port 32300 (#0)
* ALPN, offering h2
* ALPN, offering http/1.1
* successfully set certificate verify locations:
*   CAfile: /etc/ssl/certs/ca-certificates.crt
  CApath: /etc/ssl/certs
* TLSv1.3 (OUT), TLS handshake, Client hello (1):
* TLSv1.3 (IN), TLS handshake, Server hello (2):
* TLSv1.3 (IN), TLS Unknown, Certificate Status (22):
* TLSv1.3 (IN), TLS handshake, Unknown (8):
* TLSv1.3 (IN), TLS Unknown, Certificate Status (22):
* TLSv1.3 (IN), TLS handshake, Certificate (11):
* TLSv1.3 (IN), TLS Unknown, Certificate Status (22):
* TLSv1.3 (IN), TLS handshake, CERT verify (15):
* TLSv1.3 (IN), TLS Unknown, Certificate Status (22):
* TLSv1.3 (IN), TLS handshake, Finished (20):
* TLSv1.3 (OUT), TLS change cipher, Client hello (1):
* TLSv1.3 (OUT), TLS Unknown, Certificate Status (22):
* TLSv1.3 (OUT), TLS handshake, Finished (20):
* SSL connection using TLSv1.3 / TLS_AES_256_GCM_SHA384
* ALPN, server accepted to use h2
* Server certificate:                        #认证成功
*  subject: C=AU; ST=Some-State; O=Internet Widgits Pty Ltd; CN=secure-ingress.com
*  start date: Apr 21 07:55:47 2021 GMT
*  expire date: Apr 21 07:55:47 2022 GMT
*  issuer: C=AU; ST=Some-State; O=Internet Widgits Pty Ltd; CN=secure-ingress.com
*  SSL certificate verify result: self signed certificate (18), continuing anyway.
* Using HTTP2, server supports multi-use
* Connection state changed (HTTP/2 confirmed)
* Copying HTTP/2 data in stream buffer to connection buffer after upgrade: len=0
* TLSv1.3 (OUT), TLS Unknown, Unknown (23):
* TLSv1.3 (OUT), TLS Unknown, Unknown (23):
* TLSv1.3 (OUT), TLS Unknown, Unknown (23):
* Using Stream ID: 1 (easy handle 0x55b3c09e2430)
* TLSv1.3 (OUT), TLS Unknown, Unknown (23):
> GET /service2 HTTP/2
> Host: secure-ingress.com:32300
> User-Agent: curl/7.58.0
> Accept: */*
> 
* TLSv1.3 (IN), TLS Unknown, Certificate Status (22):
* TLSv1.3 (IN), TLS handshake, Newsession Ticket (4):
* TLSv1.3 (IN), TLS Unknown, Certificate Status (22):
* TLSv1.3 (IN), TLS handshake, Newsession Ticket (4):
* TLSv1.3 (IN), TLS Unknown, Unknown (23):
* Connection state changed (MAX_CONCURRENT_STREAMS updated)!
* TLSv1.3 (OUT), TLS Unknown, Unknown (23):
* TLSv1.3 (IN), TLS Unknown, Unknown (23):
* TLSv1.3 (IN), TLS Unknown, Unknown (23):
< HTTP/2 200 
< date: Wed, 21 Apr 2021 08:13:30 GMT
< content-type: text/html
< content-length: 45
< last-modified: Mon, 11 Jun 2007 18:53:14 GMT
< etag: "2d-432a5e4a73a80"
< accept-ranges: bytes
< strict-transport-security: max-age=15724800; includeSubDomains
< 
<html><body><h1>It works!</h1></body></html>
* TLSv1.3 (IN), TLS Unknown, Unknown (23):
* Connection #0 to host secure-ingress.com left intact

```




