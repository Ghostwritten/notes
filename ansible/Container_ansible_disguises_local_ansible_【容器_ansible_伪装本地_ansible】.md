
![](https://i-blog.csdnimg.cn/blog_migrate/b9aa22d0be908b76eaf46faaff56c095.png)
预备条件：

- [ctr & crictl $ nerdctl & containerd install](https://blog.csdn.net/xixihahalelehehe/article/details/134264754)
- [了解 kubespray 是什么](https://github.com/kubernetes-sigs/kubespray)

 kubespray 包含 ansible、ansible-playbook命令以及通过kubespray项目安装kubernetes集群的介质。

```bash
nerdctl pull quay.io/kubespray/kubespray:
nerdctl save -o quay.io_kubespray_kubespray_v2.24.1.tar quay.io/kubespray/kubespray:v2.24.1
nerdctl load -i quay.io_kubespray_kubespray_v2.24.1.tar 

默认每个镜像版本对应有kubespray的版本仓库。
nerdctl  run --name ansible --network=host  -itd  -v "${HOME}"/.ssh:/root/.ssh  -v /etc/hosts:/etc/hosts quay.io/kubespray/kubespray:v2.24.1  bash

如果你有一些ansilbe-playbook的代码可以挂载到里面，
nerdctl  run --name ansible --network=host  -itd -v "$(pwd)"/ansible-playbook:/ansible-playbook -v "${HOME}"/.ssh:/root/.ssh  -v /etc/hosts:/etc/hosts quay.io/kubespray/kubespray:v2.24.1  bash
```
编排 `inventory.ini`

```bash
$ nerdctl exec -ti ansible vim /ansible-playbook/inventory.ini
[all]
kube-master01 ansible_host=10.70.0.71
kube-master02 ansible_host=10.70.0.72
kube-master03 ansible_host=10.70.0.73
kube-node01 ansible_host=10.70.0.74
kube-node02 ansible_host=10.70.0.75
kube-node03 ansible_host=10.70.0.76

[bastion]
bastion01 ansible_host=10.70.0.78 ansible_user=root

[kube_control_plane]
kube-master01
kube-master02
kube-master03

[etcd]
kube-master01
kube-master02
kube-master03

[kube_node]
kube-node01
kube-node02
kube-node03

[calico_rr]

[k8s_cluster:children]
kube_control_plane
kube_node
calico_rr


$ nerdctl exec -ti ansible ansible -i /ansible-playbook/inventory.ini all -m ping

$ vim /root/.bashrc
alias ansible='nerdctl exec -ti ansible ansible -i /ansible-playbook/inventory.ini'
...

$ source  /root/.bashrc

$ ansible  all -m ping
```

