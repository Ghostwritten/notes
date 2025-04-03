


@[TOC]
## yaml

```bash
$ vim redis-deploy.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "1"
    meta.helm.sh/release-name: infini-redis
    meta.helm.sh/release-namespace: redis
  generation: 1
  labels:
    app.kubernetes.io/instance: infini-redis
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/name: infini-redis-redis
    helm.sh/chart: redis-2.0.1
  name: infini-redis-redis
  namespace: redis
spec:
  progressDeadlineSeconds: 600
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app.kubernetes.io/instance: infini-redis
      app.kubernetes.io/name: infini-redis-redis
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      creationTimestamp: null
      labels:
        app.kubernetes.io/instance: infini-redis
        app.kubernetes.io/managed-by: Helm
        app.kubernetes.io/name: infini-redis-redis
        helm.sh/chart: redis-2.0.1
    spec:
      affinity:
        nodeAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - preference:
              matchExpressions:
              - key: redis
                operator: In
                values:
                - ""
            weight: 1
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - podAffinityTerm:
              labelSelector:
                matchLabels:
                  app.kubernetes.io/instance: redis
                  app.kubernetes.io/name: redis
              topologyKey: kubernetes.io/hostname
            weight: 1
      containers:
      - command:
        - redis-server
        - /etc/redis.conf
        env:
        - name: PASS
          valueFrom:
            secretKeyRef:
              key: PASS
              name: redis
        - name: PORT
          value: "6379"
        image: redis:6.0
        imagePullPolicy: IfNotPresent
        livenessProbe:
          exec:
            command:
            - /bin/bash
            - -ec
            - |
              redis-cli -p "${PORT}" -a "${PASS}" ping 2> /dev/null | grep PONG
          failureThreshold: 3
          initialDelaySeconds: 5
          periodSeconds: 10
          successThreshold: 1
          timeoutSeconds: 1
        name: redis
        ports:
        - containerPort: 6379
          name: redis
          protocol: TCP
        readinessProbe:
          exec:
            command:
            - /bin/bash
            - -ec
            - |
              redis-cli -p "${PORT}" -a "${PASS}" ping 2> /dev/null | grep PONG
          failureThreshold: 3
          initialDelaySeconds: 5
          periodSeconds: 10
          successThreshold: 1
          timeoutSeconds: 1
        resources:
          limits:
            cpu: "2"
            memory: 2G
          requests:
            cpu: "2"
            memory: 2G
        startupProbe:
          exec:
            command:
            - /bin/bash
            - -ec
            - |
              redis-cli -p "${PORT}" -a "${PASS}" ping 2> /dev/null | grep PONG
          failureThreshold: 15
          initialDelaySeconds: 15
          periodSeconds: 10
          successThreshold: 1
          timeoutSeconds: 1
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
        volumeMounts:
        - mountPath: /etc/redis.conf
          name: config
          subPath: redis.conf
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
      volumes:
      - configMap:
          defaultMode: 420
          name: redis-config
        name: config
---
apiVersion: v1
kind: Service
metadata:
  annotations:
    meta.helm.sh/release-name: infini-redis
    meta.helm.sh/release-namespace: redis
  labels:
    app.kubernetes.io/instance: redis
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/name: redis
  name: redis
  namespace: upm-system
spec:
  ipFamilies:
  - IPv6
  ipFamilyPolicy: SingleStack
  ports:
  - name: redis
    port: 6379
    protocol: TCP
    targetPort: redis
  publishNotReadyAddresses: true
  selector:
    app.kubernetes.io/instance: redis
    app.kubernetes.io/name: redis
  sessionAffinity: None
  type: ClusterIP
---
apiVersion: v1
data:
  redis.conf: |
    protected-mode "no"
    port 6379
    tcp-backlog 511
    unixsocket "/data/redis.sock"
    timeout 60
    tcp-keepalive 0
    daemonize no
    supervised no
    pidfile "/data/redis.pid"
    loglevel notice
    logfile "/data/redis.log"
    databases "16"
    always-show-logo yes
    stop-writes-on-bgsave-error yes
    rdbcompression yes
    rdbchecksum yes
    dbfilename "dump.rdb"
    dir "/data/"
    requirepass uhi3fan_SA84+8UKD=UAW23
    maxclients 10000
    maxmemory 1024mb
    maxmemory-policy volatile-lru
    maxmemory-samples 5
    lazyfree-lazy-eviction no
    lazyfree-lazy-expire no
    lazyfree-lazy-server-del no
    replica-lazy-flush no
    appendonly no
    appendfilename appendonly.aof
    appendfsync no
    no-appendfsync-on-rewrite yes
    auto-aof-rewrite-percentage 100
    auto-aof-rewrite-min-size 512mb
    aof-load-truncated yes
    aof-use-rdb-preamble yes
    lua-time-limit 5000
    slowlog-log-slower-than 5000
    slowlog-max-len 10000
    latency-monitor-threshold 0
    hash-max-ziplist-entries 512
    hash-max-ziplist-value 64
    list-max-ziplist-size -2
    list-compress-depth 0
    set-max-intset-entries 512
    zset-max-ziplist-entries 128
    zset-max-ziplist-value 64
    hll-sparse-max-bytes 3000
    stream-node-max-bytes 4096
    stream-node-max-entries 100
    activerehashing yes
    client-output-buffer-limit normal 0 0 0
    client-output-buffer-limit replica 256mb 64mb 60
    client-output-buffer-limit pubsub 32mb 8mb 60
    hz 10
    dynamic-hz yes
kind: ConfigMap
metadata:
  labels:
    app.kubernetes.io/managed-by: Helm
  name: infini-redis-redis-config
  namespace: upm-system
---
apiVersion: v1
data:
  PASS: dWhpM2Zhbl9TQTg0KzhVS0Q9VUFXMjM=
kind: Secret
metadata:
  labels:
    app.kubernetes.io/managed-by: Helm
  name: infini-redis-redis
  namespace: upm-system
type: Opaque
```
执行：

```bash
kubectl apply -f redis-deploy.yaml
```


## helm 
 



- [https://www.learnitguide.net/2023/04/how-to-deploy-redis-cluster-on.html](https://www.learnitguide.net/2023/04/how-to-deploy-redis-cluster-on.html)
- [https://bitnami.com/stack/redis/helm](https://bitnami.com/stack/redis/helm)
