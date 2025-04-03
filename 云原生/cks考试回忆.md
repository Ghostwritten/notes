## 1. secret 
寻找某个项目空间的secret的key 和value

## 2. kube-bench
 kube-bench 但是kube-bench master不对

## 3. Falco

```bash
cat /var/log/syslog | grep falco | grep nginx | grep process
service falco stop
grep -r "Package management process launched" .


service falco stop
falco
```

## 4. authorization-mode

```bash
--authorization-mode=RBAC,Node,AlwaysAllow

kube-controller-manager 

Kubelet --authorization-mode=Webhook
```

##  5. PodSecurityPolicy

```bash
- --enable-admission-plugins=NodeRestriction,PodSecurityPolicy      # change

k -n team-red create clusterrole psp-mount --verb=use \
--resource=podsecuritypolicies --resource-name=psp-mount

k -n team-red create rolebinding psp-mount --clusterrole=psp-mount --group system:serviceaccounts
```

## 6. 版本

## 7. Open Policy Agent

```bash
k get crd
k get constraint
k edit constrainttemplates blacklistimages
```

##  8. AppArmor Profile

```bash
scp /opt/course/9/profile cluster1-worker1:~/
ssh cluster1-worker1
 apparmor_parser -q ./profile
 apparmor_status
 k label node cluster1-worker1 security=apparmor
      annotations:                                                                 # add
        container.apparmor.security.beta.kubernetes.io/c1: localhost/very-secure   # add
```

##  9. Container Runtime Sandbox gVisor

```bash
vim /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
# /etc/default/kubelet
KUBELET_EXTRA_ARGS="--container-runtime remote --container-runtime-endpoint unix:///run/containerd/containerd.sock"
```

```c
# 10_rtc.yaml
apiVersion: node.k8s.io/v1
kind: RuntimeClass
metadata:
  name: gvisor
handler: runsc
k -f 10_rtc.yaml create
And the required Pod:
```

```bash
vim 10_pod.yaml
# 10_pod.yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: gvisor-test
  name: gvisor-test
  namespace: team-purple
spec:
  nodeName: cluster1-worker2 # add
  runtimeClassName: gvisor   # add
  containers:
  - image: nginx:1.19.2
    name: gvisor-test
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

##  10. Restrict access to Metadata Server Network Policy

Create a NetworkPolicy named metadata-deny which prevents egress to 192.168.100.21 for all Pods but still allows access to everything else
Create a NetworkPolicy named metadata-allow which allows Pods having label role: metadata-accessor to access endpoint 192.168.100.21
There are existing Pods in the target Namespace with which you can test your policies, but don’t change their labels.

```bash
vim 13_metadata-deny.yaml
# 13_metadata-deny.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: metadata-deny
  namespace: metadata-access
spec:
  podSelector: {}
  policyTypes:
  - Egress
  egress:
  - to:
    - ipBlock:
        cidr: 0.0.0.0/0
        except:
        - 192.168.100.21/32
```


```bash
vim 13_metadata-allow.yaml
# 13_metadata-allow.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: metadata-allow
  namespace: metadata-access
spec:
  podSelector:
    matchLabels:
      role: metadata-accessor
  policyTypes:
  - Egress
  egress:
  - to:
    - ipBlock:
        cidr: 192.168.100.21/32
```

##  11. Syscall Activity
k -n team-yellow get pod -owide
docker ps | grep collector3
ps aux | grep collector2-process
strace -p 10438

##    12. TLS

```c
k -n team-pink create secret tls tls-secret --key tls.key --cert tls.crt


spec:
  tls:                            # add
    - hosts:                      # add
      - secure-ingress.test       # add
      secretName: tls-secret      # add
```

## 13. dockerfile
[https://github.com/couchbase/docker/blob/master/community/couchbase-server/6.6.0/Dockerfile](https://github.com/couchbase/docker/blob/master/community/couchbase-server/6.6.0/Dockerfile)

##   14. Audit Log Policy

```bash
  - command:
    - kube-apiserver
    - --audit-policy-file=/etc/kubernetes/audit/policy.yaml
    - --audit-log-path=/etc/kubernetes/audit/logs/audit.log
    - --audit-log-maxsize=5
    - --audit-log-maxbackup=1                                    # CHANGE
```


```bash
# /etc/kubernetes/audit/policy.yaml
apiVersion: audit.k8s.io/v1
kind: Policy
rules:
​
# log Secret resources audits, level Metadata
- level: Metadata
  resources:
  - group: ""
    resources: ["secrets"]
​
# log node related audits, level RequestResponse
- level: RequestResponse
  userGroups: ["system:nodes"]
​
# for everything else don't log anything
- level: None
```

```bash
mv ../kube-apiserver.yaml .

 cat audit.log | grep "p.auster" | grep Secret | grep list | wc -l
```


##  15. filesystem

```c
vim /opt/course/19/immutable-deployment-new.yaml
# /opt/course/19/immutable-deployment-new.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: team-purple
  name: immutable-deployment
  labels:
    app: immutable-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: immutable-deployment
  template:
    metadata:
      labels:
        app: immutable-deployment
    spec:
      containers:
      - image: busybox:1.32.0
        command: ['sh', '-c', 'tail -f /dev/null']
        imagePullPolicy: IfNotPresent
        name: busybox
        securityContext:                  # add
          readOnlyRootFilesystem: true    # add
        volumeMounts:                     # add
        - mountPath: /tmp                 # add
          name: temp-vol                  # add
      volumes:                            # add
      - name: temp-vol                    # add
        emptyDir: {}                      # add
      restartPolicy: Always
```


## 16. trivy
