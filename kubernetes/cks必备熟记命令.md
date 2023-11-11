



----
## 1. 集群安装：10%
```bash
k run frontend --image=nginx
k expose pod frontend --port 80


k -n kubernets-dashboard create rolebinding insecure --serviceaccount kubernetes-dashboard:kubernetes-dashboard --clusterrole view

k run pod1 --image=nginx
k run pod2 --image=httpd
k expose pod pod1 --port 80 --name service1
k expose pod pod2 --port 80 --name service2
curl  https://192.168.211.40:32300/service1 -kv

openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -days 365 -nodes

k create secret tls secure-ingress --cert=cert.pem --key=key.pem

curl  https://secure-ingress.com:32300/service2 -kv --resolv secure-ingress.com:32300:192.168.211.41


k label pod nginx role=metadata-accessor

#根据CIS标准检查
docker run --pid=host -v /etc:/etc:ro -v /var:/var:ro -t aquasec/kube-bench:latest master --version 1.20

#确认版本是否一至
sha512sum kubernetes-server-linux-arm64.tar.gz 
tar zxf kubernetes-server-linux-arm64.tar.gz 
ls kubernetes/server/bin/kube-apiserver
sha512sum kubernetes/server/bin/kube-apiserver

docker ps |grep apiserver
docker cp 0fb5321dfd57:/ container-fs
ls container-fs/
find container-fs/ | grep kube-apiserver
sha512sum container-fs/usr/local/bin/kube-apiserver
```

##  2. 集群强化：15%

```bash
curl https://localhost:6443
curl https://localhost:6443 -k

vim /etc/kubernetes/manifests/kube-apiserver.yaml

- --anonymous-auth=true
- --insecure-port=8080 
- --enable-admission-plugins=NodeRestriction

curl https://192.168.211.40:6443 --cacert ca --cert  ca.crt --key ca.key

k edit svc
curl https://192.168.211.40:30300 -l
k config view --raw >config
k --kubeconfig config get ns

k label node master cks/test=yes



k create sa accessor
k get sa,secrets
k describe secret accessor-token-bnd4s
k run accessor --image=nginx --dry-run=client -oyaml
serviceAccountName: accessor  #添加此行
k exec -ti accessor -- bash
mount |grep sec
cd /run/secrets/kubernetes.io/serviceaccount
cat token 
 curl https://kubernetes
 curl https://kubernetes -k
 curl https://kubernetes -k -H "Authorization: Bearer eyJ。。。。。

automountServiceAccountToken: false  #添加此行
k -f accessor.yaml replace --force

 k auth can-i delete secrets --as system:serviceaccount:default:accessor
 k create clusterrolebinding accessor --clusterrole edit --serviceaccount default:accessor
 k auth can-i delete secrets --as system:serviceaccount:default:accessor


k create ns red
k create ns blue
k -n red create role secret-manager --verb=get --resource=secrets -oyaml --dry-run=client
k -n red create rolebinding secret-manager --role=secret-manager --user=jane

k -n blue create role secret-manager --verb=get --verb=list --resource=secrets 
k -n blue create rolebinding secret-manager --role=secret-manager --user=jane

k -n red auth can-i get secrets --as jane


openssl genrsa -out jane.key 2048


cat jane.csr | base64 -w 0

k certificate approve jane
k config view -o yaml > view.yaml

k config set-credentials jane --client-key=jane.key --client-certificate=jane.crt

k config set-credentials jane --client-key=jane.key --client-certificate=jane.crt --embed-certs

k config view --raw

k config set-context jane --user=jane --cluster=kubernetes

k config get-contexts


k drain   master --ignore-daemonsets
apt-cache show kubeadm  |grep -e '1.20'
apt-get install kubeadm=1.20.2-00 kubectl=1.20.2-00 kubelet=1.20.2-00
kubeadm upgrade plan

kubeadm upgrade apply v1.20.6
k uncordon master
k get node
```

## 3. 系统强化：15%

```bash
netstat -natlp
ps aux
lsof -i :22
apt-get install snapd
systemctl start snapd
systemctl status snapd
systemctl list-units --type=service --state=running |grep snap

apt-get install -y vsftpd samba
systemctl status vsftpd
systemctl status smbd
ps aux |grep smbd
whoami


 k run pod --image=busybox --command -oyaml --dry-run=client > pod.yaml -- sh -c 'sleep 1d'
 root@master:~/cks/securitytext# vim pod.yaml 
....
spec:
  securityContext:
    runAsUser: 1000
    runAsGroup: 3000
  containers:
  - command:
    - sh
    - -c
    - sleep 1d
    image: busybox
    name: pod
    resources: {}
    securityContext:
      runAsNonRoot: true
      privileged: true
      allowPrivilegeEscalation: true 
.....

sysctl kernel.hostname=attacker

kubectl -f pod.yaml delete --force --grace-period=0


root@master:~/cks/securitytext# vim /etc/kubernetes/manifests/kube-apiserver.yaml
---
    - --enable-admission-plugins=NodeRestriction,PodSecurityPolicy
---


 k create role psp-access --verb=use --resource=podsecuritypolicies
k create rolebinding psp-access --role=psp-access --serviceaccount=default:default


aa-status
apt-get install apparmor-utils
aa-genprof curl  
cd /etc/apparmor.d/
aa-logprof 
 cat usr.bin.curl 
 curl killer.sh -v
k run secure --image=nginx -oyaml --dry-run=client > pod.yaml
root@master:~/cks/apparmor# cat pod.yaml 
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  annotations:    #添加此行
    container.apparmor.security.beta.kubernetes.io/secure: localhost/hello  #添加此行
root@master:~/cks/apparmor# k get pods secure
NAME     READY   STATUS    RESTARTS   AGE
secure   0/1     Blocked   0          6s
cat pod.yaml 
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  annotations: 
    container.apparmor.security.beta.kubernetes.io/secure: localhost/docker-nginx  #修改此行



root@master:~/cks/apparmor# cat pod2.yaml 
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: secure
  name: secure
spec:
  securityContext:
    seccompProfile:
      type: Localhost
      localhostProfile: profiles/audit.json
      root@master:~/cks/apparmor# k get pods  -w
NAME       READY   STATUS                 RESTARTS   AGE
accessor   1/1     Running                0          26h
secure     0/1     CreateContainerError   0          23s
root@master:~/cks/apparmor# cat pod2.yaml 
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: secure
  name: secure
spec:
  securityContext:
    seccompProfile:
      type: Localhost
      localhostProfile: default.json


```


## 4. 微服务漏洞最小化：20%

```bash
root@master:~/cks/opa# vim template.yaml
apiVersion: templates.gatekeeper.sh/v1beta1
kind: ConstraintTemplate
metadata:
  name: k8salwaysdeny
spec:
  crd:
    spec:
      names:
        kind: K8sAlwaysDeny
      validation:
        # Schema for the `parameters` field
        openAPIV3Schema:
          properties:
            message:
              type: string
  targets:
    - target: admission.k8s.gatekeeper.sh
      rego: |
        package k8salwaysdeny

        violation[{"msg": msg}] {
          1 > 0
          msg := input.parameters.message
        }


root@master:~/cks/opa# vim constraint.yaml 
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sAlwaysDeny
metadata:
  name: pod-always-deny
spec:
  match:
    kinds:
      - apiGroups: [""]
        kinds: ["Pod"]
  parameters:
    message: "ACCESS DENIED!"



root@master:~/cks/opa# vim template_label.yaml 
apiVersion: templates.gatekeeper.sh/v1beta1
kind: ConstraintTemplate
metadata:
  name: k8srequiredlabels
spec:
  crd:
    spec:
      names:
        kind: K8sRequiredLabels
      validation:
        # Schema for the `parameters` field
        openAPIV3Schema:
          properties:
            labels:
              type: array
              items: string
  targets:
    - target: admission.k8s.gatekeeper.sh
      rego: |
        package k8srequiredlabels

        violation[{"msg": msg, "details": {"missing_labels": missing}}] {
          provided := {label | input.review.object.metadata.labels[label]}
          required := {label | label := input.parameters.labels[_]}
          missing := required - provided
          count(missing) > 0
          msg := sprintf("you must provide labels: %v", [missing])
        }

root@master:~/cks/opa# vim all_ns_must_have_cks.yaml 
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sRequiredLabels
metadata:
  name: ns-must-have-cks
spec:
  match:
    kinds:
      - apiGroups: [""]
        kinds: ["Namespace"]
  parameters:
    labels: ["cks"]




k create secret generic secret1 --from-literal user=admin
k create secret generic secret2 --from-literal pass=12345678

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


k exec pod -- env |grep PASS
k exec pod -- mount |grep secret1
k exec pod -- ls /etc/secret1
k exec pod -- cat /etc/secret1/user


ETCDCTL_API=3 etcdctl --endpoints https://192.168.211.40:2379   --cert /etc/kubernetes/pki/etcd/server.crt --key /etc/kubernetes/pki/etcd/server.key --cacert /etc/kubernetes/pki/etcd/ca.crt endpoint health


ETCDCTL_API=3 etcdctl --endpoints https://192.168.211.40:2379   --cert /etc/kubernetes/pki/etcd/server.crt --key /etc/kubernetes/pki/etcd/server.key --cacert /etc/kubernetes/pki/etcd/ca.crt get /registry/secrets/default/secret1


cd /etc/kubernetes/
mkdir etcd
cd etcd/
echo -n password | base64
echo -n passwordpassword | base64
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

im kube-apiserver.yaml 
.....
    - kube-apiserver
    - --encryption-provider-config=/etc/kubernetes/etcd/ec.yaml #添加

    - mountPath: /etc/kubernetes/etcd   #添加
      name: etcd   #添加
      readOnly: true   #添加

  - hostPath:   #添加
      path: /etc/kubernetes/etcd  #添加
      type: DirectoryOrCreate   #添加
    name: etcd   #添加

cd /var/log/pods/
 tail -f kube-system_kube-api
.......

ETCDCTL_API=3 etcdctl --endpoints https://192.168.211.40:2379   --cert /etc/kubernetes/pki/etcd/server.crt --key /etc/kubernetes/pki/etcd/server.key --cacert /etc/kubernetes/pki/etcd/ca.crt get  /registry/secrets/default/default-token-2xr8c


k create secret generic very-secure --from-literal cc=1234

ETCDCTL_API=3 etcdctl --endpoints https://192.168.211.40:2379   --cert /etc/kubernetes/pki/etcd/server.crt --key /etc/kubernetes/pki/etcd/server.key --cacert /etc/kubernetes/pki/etcd/ca.crt get  /registry/secrets/default/very-secure

echo  MTIzNA== | base64 -d


docker ps
crictl ps
crictl pods


 k run app --image=bash --command -oyaml --dry-run=client > app.yaml -- sh -c 'ping baidu.com'

root@master:~/cks/securitytext# cat app.yaml 
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: app
  name: app
spec:
  containers:
  - command:
    - sh
    - -c
    - sleep 1d
    image: busybox
    name: pod
    resources: {}
  - name: proxy
    image: ubuntu
    command:
    - sh
    - -c
    - 'apt-get update && apt-get install iptables -y && iptables -L && sleep 1d'
    securityContext:
      capabilities:
        add: ["NET_ADMIN"]      
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}

```
## 5. 供应链安全：20%

```bash
# build container stage 1
FROM ubuntu:20.04
ARG DEBIAN_FRONTEND=noninteractive
RUN apt-get update && apt-get install -y golang-go=2:1.13~1ubuntu2
COPY app.go .
RUN pwd
RUN CGO_ENABLED=0 go build app.go

# app container stage 2
FROM alpine:3.12.0
RUN chmod a-w /etc
RUN addgroup -S appgroup && adduser -S appuser -G appgroup -h /home/appuser
COPY --from=0 /app /home/appuser/
USER appuser
CMD ["/home/appuser/app"]



docker  build --network=host -t app .


k get pods -A -oyaml |grep image: |grep -v f: |grep api
k get pod -n kube-system kube-apiserver-master -oyaml |grep image
vim /etc/kubernetes/manifests/kube-apiserver.yaml


mkdir /etc/kubernetes/admission
cat /etc/kubernetes/manifests/kube-apiserver.yaml 
    - kube-apiserver
    - --admission-control-config-file=/etc/kubernetes/admission/admission_config.yaml #添加此行


root@master:~# cat /etc/kubernetes/admission/admission_config.yaml
apiVersion: apiserver.config.k8s.io/v1
kind: AdmissionConfiguration
plugins:
  - name: ImagePolicyWebhook
    configuration:
      imagePolicy:
        kubeConfigFile: /etc/kubernetes/admission/kubeconf
        allowTTL: 50
        denyTTL: 50
        retryBackoff: 500
        defaultAllow: true    #修改此行为true

.......




root@node1:~/cks/static-analysis/conftest/kubernetes# ls
deploy.yaml  policy  run.sh


root@node1:~/cks/static-analysis/conftest/kubernetes# cat deploy.yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: test
  name: test
spec:
  replicas: 1
  selector:
    matchLabels:
      app: test
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: test
    spec:
      containers:
        - image: httpd
          name: httpd
          resources: {}
status: {}


root@node1:~/cks/static-analysis/conftest/kubernetes# cat policy/deployment.rego 
# from https://www.conftest.dev
package main

deny[msg] {
  input.kind = "Deployment"
  not input.spec.template.spec.securityContext.runAsNonRoot = true
  msg = "Containers must not run as root"
}

deny[msg] {
  input.kind = "Deployment"
  not input.spec.selector.matchLabels.app
  msg = "Containers must provide app label for pod selectors"
}



root@node1:~/cks/static-analysis/conftest/kubernetes# cat run.sh 
docker run --rm -v $(pwd):/project openpolicyagent/conftest test deploy.yaml




docker run ghcr.io/aquasecurity/trivy:latest image nginx:latest
docker run --net=host  ghcr.io/aquasecurity/trivy:latest image nginx:latest
docker run --net=host  ghcr.io/aquasecurity/trivy:latest image nginx:latest |grep CRITICAL
docker run --net=host  ghcr.io/aquasecurity/trivy:latest image nginx:1.16-alpine

```


##   6. 监控、日志记录和运行时安全：20%

```bash
 strace ls
 strace -cw ls /
 echo hello > test
 cat test 
 strace cat test
 docker ps |grep etcd
 ps -ef |grep etcd
 strace -p 118382 -f
 strace -p 118382 -f -cw
 ls /proc/118382/
 ls -l /proc/118382/exe
 ls  /proc/118382/fd
 ls -l /proc/118382/fd
 tail 7
 k create secret generic credit-card --from-literal cc=111222333444 
 cat 7 | strings | grep 111222333444
 cat 7 | strings | grep 111222333444  -A 10 -B 10

k run apache --image=httpd -oyaml --dry-run=client > pod.yaml

root@master:~/cks/runtime-security# vim pod.yaml 
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: apache
  name: apache
spec:
  containers:
  - image: httpd
    name: apache
    resources: {}
    env: 
    - name: SECRET
      value: "5555666677778888"


k get pods -owide |grep apache
ps aux |grep httpd
pstree -p
cd /proc/123888
cat environ 

tail /var/log/syslog|grep falco


root@node2:~# cd /etc/falco/
root@node2:/etc/falco# ls
falco_rules.local.yaml  falco_rules.yaml  falco.yaml  k8s_audit_rules.yaml  rules.available  rules.d


root@node2:/etc/falco# grep -r "A shell was spawned in a container with an attached terminal" *
falco_rules.yaml:    A shell was spawned in a container with an attached terminal (user=%user.name user_loginuid=%user.loginuid %container.info

#更新配置
root@node2:/etc/falco# cat falco_rules.local.yaml
- rule: Terminal shell in container
  desc: A shell was used as the entrypoint/exec point into a container with an attached terminal.
  condition: >
    spawned_process and container
    and shell_procs and proc.tty != 0
    and container_entrypoint
    and not user_expected_terminal_shell_in_container_conditions
  output: >
    %evt.time,%user.name,%container.name,%container.id
    shell=%proc.name parent=%proc.pname cmdline=%proc.cmdline terminal=%proc.tty container_id=%container.id image=%container.image.repository)
  priority: WARNING
  tags: [container, shell, mitre_execution]



falco


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



mkdir /etc/kubernetes/auditing
mkidr /etc/kubernetes/audit/logs
cat /etc/kubernetes/audit/policy.yaml 
apiVersion: audit.k8s.io/v1
kind: Policy
rules:
- level: Metadata


root@master:/etc/kubernetes/manifests# cat kube-apiserver.yaml 
.........
    - kube-apiserver
    - --audit-policy-file=/etc/kubernetes/audit/policy.yaml       # add
    - --audit-log-path=/etc/kubernetes/audit/logs/audit.log       # add
    - --audit-log-maxsize=500                                     # add
......
    - mountPath: /etc/kubernetes/audit      # add
      name: audit                           # add
 
 
  volumes:
  - hostPath:                               # add
      path: /etc/kubernetes/audit           # add
      type: DirectoryOrCreate               # add
    name: audit                             # add
  - hostPath:


tail /etc/kubernetes/audit/logs/audit.log 

k create secret generic very-secure --from-literal=user=admin
cat /etc/kubernetes/audit/logs/audit.log |grep very-secure | jq .


root@master:/etc/kubernetes/audit# cat policy.yaml 
apiVersion: audit.k8s.io/v1
kind: Policy
omitStages:
  - "RequestReceived"
rules:
- level: Metadata

- level: None
  verbs: ["get","list","watch"]

- level: Metadata
  resources:
  - group: ""
    resources: ["secrets"]

- level: RequestResponse


mv /etc/kubernetes/manifests/kube-apiserver.yaml /etc/kubernetes/
ps aux |grep api
mv /etc/kubernetes/kube-apiserver.yaml /etc/kubernetes/manifests/kube-apiserver.yaml
ps aux |grep api
tail /etc/kubernetes/audit/logs/audit.log | jq .

 k create sa very-crazy-sa
 k get secret
 cat /etc/kubernetes/audit/logs/audit.log |grep very-crazy-sa
 k run accessor --image=nginx --dry-run=client -oyaml > pod.yaml
 root@master:~/cks/runtime-security# vim pod3.yaml 
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: accessor
  name: accessor
spec:
  serviceAccountName: very-crazy-sa   #添加此行
  containers:
  - image: nginx
    name: accessor
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}


k get pod accessor -w
 cat /etc/kubernetes/audit/logs/audit.log |grep accessor

```



