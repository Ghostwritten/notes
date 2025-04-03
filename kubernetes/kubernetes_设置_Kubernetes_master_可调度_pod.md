
kubernetes 设置节点可调度

```bash
kubectl taint node node01 node-role.kubernetes.io/master-
```

kubernetes 设置节点不可调度

```bash
kubectl taint node node01 node-role.kubernetes.io/master="":NoSchedule
```

