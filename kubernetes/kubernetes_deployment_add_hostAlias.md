```bash
      hostAliases:
        - ip: 192.168.23.80
          hostnames:
            - rancher02.demo.com
```

```bash
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
        - ip: 192.168.23.80
          hostnames:
            - rancher02.demo.com
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
          secretName: cattle-credentials-fa3defb
          defaultMode: 320
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 0
      maxSurge: 1

```

