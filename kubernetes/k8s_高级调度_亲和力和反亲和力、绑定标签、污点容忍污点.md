


--

## 1. 通过标签绑定 

```bash
spec:
  nodeSelector:
    bigdata-node: bigdata
  containers:
    - env:
```

pod只能运行在有bigdata-node: bigdata 标签的node节点

## 2. 通过node name绑定

```bash
spec:
  nodeName: test-oc08
  containers:
    - env:
```

pod只能运行在名为test-oc08节点上

 

## 3. node的亲和力

```bash
apiVersion: v1
kind: Pod
metadata:
  name: with-node-affinity
spec:
  affinity:
    nodeAffinity:  #节点选择
      requiredDuringSchedulingIgnoredDuringExecution: #定义必要规则
        nodeSelectorTerms:
        - matchExpressions:
          - key: e2e-az-NorthSouth  #必须匹配的键/值对（标签）
            operator: In 
            values:
            - e2e-az-North #必须匹配的键/值对（标签
            - e2e-az-South 
  containers:
  - name: with-node-affinity
    image: docker.io/ocpqe/hello-pod
```

该规则要求将pod放置在节点上，且节点的标签的关键字是`e2e-az-NorthSouth`，其值是e2e-az-North或者e2e-az-South

 - 您可以`In`在示例中看到正在使用的运算符。新的节点亲和力语法支持以下运算符：In，NotIn，Exists，DoesNotExist，Gt，Lt。您可以使用NotIn和DoesNotExist实现节点的反亲和行为，也可以使用节点异味来排斥特定节点的吊舱。
 - 如果同时指定`nodeSelector`和`nodeAffinity`，则必须同时满足两个条件，才能将Pod调度到候选节点上。
 - 如果您指定了多个`nodeSelectorTerms`与`nodeAffinity`类型相关联的类型，那么如果
   nodeSelectorTerms可以满足其中之一，则可以将Pod调度到一个节点上。
 - 如果您指定`matchExpressions`与关联的多个`nodeSelectorTerms`，则只有在
   matchExpressions满足所有条件后，才能将广告连播安排到一个节点上。
 - 如果您删除或更改计划了pod的节点的标签，则pod不会被删除。换句话说，亲和性选择仅在安排豆荚时有效。
 - `weight`在`preferredDuringSchedulingIgnoredDuringExecution`的范围是从1-100。对于满足所有调度要求（资源请求，RequiredDuringScheduling亲缘关系表达式等）的每个节点，调度程序将通过遍历此字段的元素并在该节点的匹配项中添加“权重”来计算总和MatchExpressions。然后，将该分数与该节点的其他优先级功能的分数合并。总得分最高的节点是最优选的。

```bash
apiVersion: v1
kind: Pod
metadata:
  name: with-node-affinity
spec:
  affinity:
    nodeAffinity: 
      preferredDuringSchedulingIgnoredDuringExecution: #定义首选规则
      - weight: 1  #权重
        preference:
          matchExpressions:
          - key: e2e-az-EastWest 
            operator: In 
            values:
            - e2e-az-East 
            - e2e-az-West 
  containers:
  - name: with-node-affinity
    image: docker.io/ocpqe/hello-pod
```

该节点具有标签的关键字为e2e-az-EastWest且其值为e2e-az-East或者e2e-az-West是该Pod的首选项的节点

## 4. pod间亲和性和反亲和性

```bash
apiVersion: v1
kind: Pod
metadata:
  name: with-pod-affinity
spec:
  affinity:
    podAffinity: #亲和性
      requiredDuringSchedulingIgnoredDuringExecution: 
      - labelSelector:
          matchExpressions:
          - key: security 
            operator: In 
            values:
            - S1 
        topologyKey: failure-domain.beta.kubernetes.io/zone
  containers:
  - name: with-pod-affinity
    image: docker.io/ocpqe/hello-pod
```

pod必须部署在一个节点上，这个节点上至少有一个正在运行的pod，这个pod中security=S1

```bash
apiVersion: v1
kind: Pod
metadata:
  name: with-pod-antiaffinity
spec:
  affinity:
    podAntiAffinity: #反亲和性
      preferredDuringSchedulingIgnoredDuringExecution: 
      - weight: 100 
        podAffinityTerm:
          labelSelector:
            matchExpressions:
            - key: security 
              operator: In 
              values:
              - S2 
          topologyKey: kubernetes.io/hostname
  containers:
  - name: with-pod-affinity
    image: docker.io/ocpqe/hello-pod
```

pod不太会部署在一个节点上，这个节点上正在运行的pod中有security=S2

## 5. node的污点

 - `NoSchedule`  #新pod不会被调到该节点，已经存在的pod不会影响
 - `PreferNoSchedule` #新pod尽量避免不会调度到该节点，已经存在的pod不会影响
 - `NoExecute`  #新pod不会被调到该节点，已经存在的pod将会被删除

```bash
spec:
  taints:
  - effect: NoSchedule
    key: key1
    timeAdded: null
    value: value1
```

给节点添加污点

```bash
spec:
  tolerations:
  - key: "key1"
    operator: "Equal"
    value: "value1"
    effect: "NoSchedule"
```

## 6. 综合

```bash
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: nginx
            operator: Exists
    podAntiAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchLabels:
            app.kubernetes.io/instance: nginx
            app.kubernetes.io/name: nginx
        topologyKey: kubernetes.io/hostname
```

pod调度对有nginx为key的节点保持亲和，对存在标签`app.kubernetes.io/instance: nginx`与`app.kubernetes.io/name: nginx`以及存在key为`kubernetes.io/hostname`的pod的节点表示反亲和
