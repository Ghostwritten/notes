


----
## 1. 介绍
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210524151812443.png?shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210524151938281.png?shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/2021052415210412.png?shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
## 2. 容器安全加固方法
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210524152220276.png?shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210524152324383.png?shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210524152450373.png?shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/2021052415250598.png?shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210524152633930.png?shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)
## 3. StartupProbe探针
官方k8s:[https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210524152703757.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)

```c
root@master:~/cks/runtime-security# k run immutable --image=httpd -oyaml --dry-run=client > pod.yaml
root@master:~/cks/runtime-security# cat pod.yaml 
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: immutable
  name: immutable
spec:
  containers:
  - image: httpd
    name: immutable
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
root@master:~/cks/runtime-security# k create -f pod.yaml 
pod/immutable created
root@master:~/cks/runtime-security# k get pods immutable 
NAME        READY   STATUS    RESTARTS   AGE
immutable   1/1     Running   0          26s
root@master:~/cks/runtime-security# k exec -it immutable -- bash
root@immutable:/usr/local/apache2# touch test
root@immutable:/usr/local/apache2# exit
exit


#更新配置pod.yaml删除touch
root@master:~/cks/runtime-security# cat pod.yaml 
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: immutable
  name: immutable
spec:
  containers:
  - image: httpd
    name: immutable
    resources: {}
    startupProbe:
      exec:
        command:
        - rm
        - /bin/touch
      initialDelaySeconds: 5
      periodSeconds: 5
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
root@master:~/cks/runtime-security# k -f pod.yaml delete --force --grace-period 0
warning: Immediate deletion does not wait for confirmation that the running resource has been terminated. The resource may continue to run on the cluster indefinitely.
pod "immutable" force deleted

root@master:~/cks/runtime-security# k -f  pod.yaml create 
pod/immutable created
root@master:~/cks/runtime-security# k get pods immutable
NAME        READY   STATUS    RESTARTS   AGE
immutable   0/1     Running   0          38s
root@master:~/cks/runtime-security# k exec -ti immutable -- bash
#命令已被删除
root@immutable:/usr/local/apache2# touch test
bash: touch: command not found


#更新配置删除bash
root@master:~/cks/runtime-security# cat pod.yaml 
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: immutable
  name: immutable
spec:
  containers:
  - image: httpd
    name: immutable
    resources: {}
    startupProbe:
      exec:
        command:
        - rm
        - /bin/bash
      initialDelaySeconds: 1
      periodSeconds: 5
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}


root@master:~/cks/runtime-security# k -f pod.yaml delete --force --grace-period 0
warning: Immediate deletion does not wait for confirmation that the running resource has been terminated. The resource may continue to run on the cluster indefinitely.
pod "immutable" force deleted
root@master:~/cks/runtime-security# k -f  pod.yaml create 
pod/immutable created
root@master:~/cks/runtime-security# k get pods immutable
NAME        READY   STATUS    RESTARTS   AGE
immutable   1/1     Running   0          39s
#bash命令已被删除
root@master:~/cks/runtime-security# k exec -ti immutable -- bash
OCI runtime exec failed: exec failed: container_linux.go:346: starting container process caused "exec: \"bash\": executable file not found in $PATH": unknown
command terminated with exit code 126
```
## 4. SecurityContext
[emptydir](https://kubernetes.io/docs/concepts/storage/volumes/#emptydir)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210524155251774.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpeGloYWhhbGVsZWhlaGU=,size_16,color_FFFFFF,t_70)


```c
root@master:~/cks/runtime-security# cat pod.yaml 
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: immutable
  name: immutable
spec:
  containers:
  - image: httpd
    name: immutable
    resources: {}
    securityContext:
      readOnlyRootFilesystem: true
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}



root@master:~/cks/runtime-security# k create -f pod.yaml 
pod/immutable created
root@master:~/cks/runtime-security# k get pods
NAME        READY   STATUS              RESTARTS   AGE
immutable   0/1     ContainerCreating   0          5s
root@master:~/cks/runtime-security# k get pods -w
NAME        READY   STATUS              RESTARTS   AGE
immutable   0/1     ContainerCreating   0          8s
immutable   0/1     Error               0          25s
^Croot@master:~/cks/runtime-security# k logs immutable
AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using 10.244.104.9. Set the 'ServerName' directive globally to suppress this message
AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using 10.244.104.9. Set the 'ServerName' directive globally to suppress this message
[Mon May 24 07:45:45.344808 2021] [core:error] [pid 1:tid 140225292526720] (30)Read-only file system: AH00099: could not create /usr/local/apache2/logs/httpd.pid
[Mon May 24 07:45:45.344947 2021] [core:error] [pid 1:tid 140225292526720] AH00100: httpd: could not log pid to file /usr/local/apache2/logs/httpd.pid


# 更新配置挂载emptydir
root@master:~/cks/runtime-security# cat pod.yaml 
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: immutable
  name: immutable
spec:
  containers:
  - image: httpd
    name: immutable
    resources: {}
    securityContext:
      readOnlyRootFilesystem: true
    volumeMounts:
    - mountPath: /usr/local/apache2/logs
      name: cache-volume
  volumes:
  - name: cache-volume
    emptyDir: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}


root@master:~/cks/runtime-security# k -f pod.yaml delete --force --grace-period 0
warning: Immediate deletion does not wait for confirmation that the running resource has been terminated. The resource may continue to run on the cluster indefinitely.
pod "immutable" force deleted
root@master:~/cks/runtime-security# k create -f pod.yaml 
pod/immutable created

root@master:~/cks/runtime-security# k get pod -w
NAME        READY   STATUS              RESTARTS   AGE
immutable   0/1     ContainerCreating   0          4s
immutable   1/1     Running             0          21s
^Croot@master:~/cks/runtime-security# k exec -ti immutable -- bash

#其他目录无法创建，只有/usr/local/apache2/logs/可以创建文件
root@immutable:/usr/local/apache2# touch test
touch: cannot touch 'test': Read-only file system
root@immutable:/usr/local/apache2# touch /usr/local/apache2/logs/test
root@immutable:/usr/local/apache2# ls /usr/local/apache2/logs/    
httpd.pid  test

```


