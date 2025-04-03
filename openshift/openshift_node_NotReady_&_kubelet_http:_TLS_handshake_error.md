
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/fe19342061d94dad9c5b3e5d3d6c45c4.png)



# 问题现象
openshift 集群 node 节点 notready

```bash
$ oc get node
NAME                        STATUS     ROLES                  AGE     VERSION
master1.ocp4.demo.com   Ready      control-plane,master   4d14h   v1.29.7+6abe8a1
master2.ocp4.demo.com   Ready      control-plane,master   4d15h   v1.29.7+6abe8a1
master3.ocp4.demo.com   Ready      control-plane,master   4d14h   v1.29.7+6abe8a1
worker1.ocp4.demo.com   Ready      worker                 4d14h   v1.29.7+6abe8a1
worker2.ocp4.demo.com   NotReady   worker                 4d14h   v1.29.7+6abe8a1
worker3.ocp4.demo.com   Ready      worker                 4d14h   v1.29.7+6abe8a1
```

登陆worker2.ocp4.demo.com 检查 kubelet 日志
```bash
$ ssh core@worker2.ocp4.demo.com 
$ journalctl -b -f -u kubelet.service
```
输出：

```bash
Aug 27 06:51:56 worker2.ocp4.demo.com kubenswrapper[1562807]: I0827 06:51:56.776626 1562807 kubelet_node_status.go:402] "Setting node annotation to enable volume controller attach/detach"
Aug 27 06:51:56 worker2.ocp4.demo.com kubenswrapper[1562807]: I0827 06:51:56.777600 1562807 kubelet_node_status.go:729] "Recording event message for node" node="worker2.ocp4.demo.com" event="NodeHasSufficientMemory"
Aug 27 06:51:56 worker2.ocp4.demo.com kubenswrapper[1562807]: I0827 06:51:56.777630 1562807 kubelet_node_status.go:729] "Recording event message for node" node="worker2.ocp4.demo.com" event="NodeHasNoDiskPressure"
Aug 27 06:51:56 worker2.ocp4.demo.com kubenswrapper[1562807]: I0827 06:51:56.777640 1562807 kubelet_node_status.go:729] "Recording event message for node" node="worker2.ocp4.demo.com" event="NodeHasSufficientPID"
Aug 27 06:51:57 worker2.ocp4.demo.com kubenswrapper[1562807]: I0827 06:51:57.172065 1562807 log.go:245] http: TLS handshake error from 10.129.2.16:35748: no serving certificate available for the kubelet
Aug 27 06:51:57 worker2.ocp4.demo.com kubenswrapper[1562807]: E0827 06:51:57.734731 1562807 transport.go:123] "No valid client certificate is found but the server is not responsive. A restart may be necessary to retrieve new initial credentials." lastCertificateAvailabilityTime="2024-08-27 05:50:01.73440836 +0000 UTC m=+0.036890907" shutdownThreshold="5m0s"
Aug 27 06:51:57 worker2.ocp4.demo.com kubenswrapper[1562807]: I0827 06:51:57.766598 1562807 csi_plugin.go:880] Failed to contact API server when waiting for CSINode publishing: csinodes.storage.k8s.io "worker2.ocp4.demo.com" is forbidden: User "system:anonymous" cannot get resource "csinodes" in API group "storage.k8s.io" at the cluster scope
Aug 27 06:51:58 worker2.ocp4.demo.com kubenswrapper[1562807]: E0827 06:51:58.735454 1562807 transport.go:123] "No valid client certificate is found but the server is not responsive. A restart may be necessary to retrieve new initial credentials." lastCertificateAvailabilityTime="2024-08-27 05:50:01.73440836 +0000 UTC m=+0.036890907" shutdownThreshold="5m0s


$ oc login -u ocpadmin -p ocpadmin
Login successful.

You have access to 74 projects, the list has been suppressed. You can list all projects with 'oc projects'

Using project "default".

$ for i in `oc get nodes -o jsonpath=$'{range .items[*]}{.metadata.name}\n{end}'`; do oc get --raw /api/v1/nodes/$i/proxy/healthz; echo -e "\t$i"; done
ok      master1.ocp4.demo.com
ok      master2.ocp4.demo.com
ok      master3.ocp4.demo.com
ok      worker1.ocp4.demo.com
Error from server (ServiceUnavailable): error trying to reach service: remote error: tls: internal error
        worker2.ocp4.demo.com
ok      worker3.ocp4.demo.com

```

# 解决方法

执行证书批准多次，直到所有pending 全部消失。

```bash
$ oc get csr | grep -i pending
$ oc get csr -o go-template='{{range .items}}{{if not .status}}{{.metadata.name}}{{"\n"}}{{end}}{{end}}' | xargs oc adm certificate approve
```


参考：

- [https://access.redhat.com/solutions/3951691](https://access.redhat.com/solutions/3951691)
