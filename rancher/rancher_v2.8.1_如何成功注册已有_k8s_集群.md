


需要加入的集群为rke2部署的双节点集群

```bash
$ kubectl get node
NAME           STATUS   ROLES                              AGE   VERSION
rke-master01   Ready    control-plane,etcd,master,worker   94d   v1.26.8+rke2r1
rke-master02   Ready    control-plane,etcd,master,worker   93d   v1.26.8+rke2r1
```

登陆 rancher
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/409721be786f091c0d9b2bc2e1852d31.png)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/1543366ec27fecfbd2dacafc7a8314dd.png)

**注意：直接执行截图中的命令，不要改动yaml内容。当执行完后注册集群的agent会报错，随后，我们通过
`kubectl edit deploy cattle-cluster-agent -n cattle-system`命令添加`hostAlias`,其他方式会注册失败，例如：先修改yaml再apply执行。**




格式如下：

```bash
      hostAliases:
      - hostnames:
        - rancher02.demo.com
        ip: 192.168.23.80
```

完整内容：

```bash
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: proxy-clusterrole-kubeapiserver
rules:
- apiGroups: [""]
  resources:
  - nodes/metrics
  - nodes/proxy
  - nodes/stats
  - nodes/log
  - nodes/spec
  verbs: ["get", "list", "watch", "create"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: proxy-role-binding-kubernetes-master
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: proxy-clusterrole-kubeapiserver
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: User
  name: kube-apiserver
---
apiVersion: v1
kind: Namespace
metadata:
  name: cattle-system

---

apiVersion: v1
kind: ServiceAccount
metadata:
  name: cattle
  namespace: cattle-system

---

apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: cattle-admin-binding
  namespace: cattle-system
  labels:
    cattle.io/creator: "norman"
subjects:
- kind: ServiceAccount
  name: cattle
  namespace: cattle-system
roleRef:
  kind: ClusterRole
  name: cattle-admin
  apiGroup: rbac.authorization.k8s.io

---

apiVersion: v1
kind: Secret
metadata:
  name: cattle-credentials-535d46a
  namespace: cattle-system
type: Opaque
data:
  url: "aHR0cHM6Ly9yYW5jaGVyMDIuZGVtby5jb20="
  token: "Z2hwZmJkbHBzbTk1NTJ4cXZuYmQ3NW5yOXA1N3d0ZnN4bGJ2dDd6cmwyY3Zwc3BxbGc5NWY1"
  namespace: ""

---

apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cattle-admin
  labels:
    cattle.io/creator: "norman"
rules:
- apiGroups:
  - '*'
  resources:
  - '*'
  verbs:
  - '*'
- nonResourceURLs:
  - '*'
  verbs:
  - '*'

---

apiVersion: apps/v1
kind: Deployment
metadata:
  name: cattle-cluster-agent
  namespace: cattle-system
  annotations:
    management.cattle.io/scale-available: "2"
spec:
  selector:
    matchLabels:
      app: cattle-cluster-agent
  template:
    metadata:
      labels:
        app: cattle-cluster-agent
    spec:
      affinity:
        nodeAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - preference:
              matchExpressions:
              - key: node-role.kubernetes.io/controlplane
                operator: In
                values:
                - "true"
            weight: 100
          - preference:
              matchExpressions:
              - key: node-role.kubernetes.io/control-plane
                operator: In
                values:
                - "true"
            weight: 100
          - preference:
              matchExpressions:
              - key: node-role.kubernetes.io/master
                operator: In
                values:
                - "true"
            weight: 100
          - preference:
              matchExpressions:
              - key: cattle.io/cluster-agent
                operator: In
                values:
                - "true"
            weight: 1
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: beta.kubernetes.io/os
                operator: NotIn
                values:
                - windows
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - podAffinityTerm:
              labelSelector:
                matchExpressions:
                - key: app
                  operator: In
                  values:
                  - cattle-cluster-agent
              topologyKey: kubernetes.io/hostname
            weight: 100
      serviceAccountName: cattle
      hostAliases:
      - hostnames:
        - rancher02.demo.com
        ip: 192.168.23.80
      tolerations:
      # No taints or no controlplane nodes found, added defaults
      - effect: NoSchedule
        key: node-role.kubernetes.io/controlplane
        value: "true"
      - effect: NoSchedule
        key: "node-role.kubernetes.io/control-plane"
        operator: "Exists"
      - effect: NoSchedule
        key: "node-role.kubernetes.io/master"
        operator: "Exists"
      containers:
        - name: cluster-register
          imagePullPolicy: IfNotPresent
          env:
          - name: CATTLE_IS_RKE
            value: "false"
          - name: CATTLE_SERVER
            value: "https://rancher02.demo.com"
          - name: CATTLE_CA_CHECKSUM
            value: "d818528e6c91a42ed9573c1cbe4b6e3df067d3ebca8b57efccb8e463306e3760"
          - name: CATTLE_CLUSTER
            value: "true"
          - name: CATTLE_K8S_MANAGED
            value: "true"
          - name: CATTLE_CLUSTER_REGISTRY
            value: ""
          - name: CATTLE_SERVER_VERSION
            value: v2.8.1
          - name: CATTLE_INSTALL_UUID
            value: 26c8f3d4-ad4d-4412-87e6-2f4ecb3ce63c
          - name: CATTLE_INGRESS_IP_DOMAIN
            value: sslip.io
          image: rancher/rancher-agent:v2.8.1
          volumeMounts:
          - name: cattle-credentials
            mountPath: /cattle-credentials
            readOnly: true
      volumes:
      - name: cattle-credentials
        secret:
          secretName: cattle-credentials-535d46a
          defaultMode: 320
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 0
      maxSurge: 1

---
apiVersion: v1
kind: Service
metadata:
  name: cattle-cluster-agent
  namespace: cattle-system
spec:
  ports:
  - port: 80
    targetPort: 80
    protocol: TCP
    name: http
  - port: 443
    targetPort: 444
    protocol: TCP
    name: https-internal
  selector:
    app: cattle-cluster-agent
```

注册成功。如图：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/7b21cfeb2d6ac3e3de65070639e79973.png)

