

----
## 1. 介绍
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3fd9dd793e22eb23d40ac4840fb6fc3c.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/fefdb7fe77bcff1df02d2fb7a5b94f3d.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f9cf5cbbdf8fb3d5c88d8a5b3ca15384.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8b0395b5aba53538ae6b04cf40bdc1ea.png)
technical overview : container and system calls
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ca359799bbbcee8b03fed457eb5b91cf.png)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6a4c2e4fa9acfe8a1e3a1fad84fea589.png)
## 2. Container calls Linux Kernel
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3808110d405f7ded5bd35dc2c3d09172.png)

```c
root@master:~# k run pod --image=nginx
pod/pod created
root@master:~# k get pod
NAME   READY   STATUS    RESTARTS   AGE
pod    1/1     Running   0          9s
root@master:~# k exec pod -ti -- bash
root@pod:/# 
root@pod:/# uname -r
4.4.0-198-generic
root@pod:/# exit
exit
root@master:~# uname -r
4.4.0-142-generic


root@master:~# strace uname -r
execve("/bin/uname", ["uname", "-r"], [/* 26 vars */]) = 0
brk(NULL)                               = 0x163a000
access("/etc/ld.so.nohwcap", F_OK)      = -1 ENOENT (No such file or directory)
access("/etc/ld.so.preload", R_OK)      = -1 ENOENT (No such file or directory)
openat(AT_FDCWD, "/etc/ld.so.cache", O_RDONLY|O_CLOEXEC) = 3
fstat(3, {st_mode=S_IFREG|0644, st_size=47425, ...}) = 0
mmap(NULL, 47425, PROT_READ, MAP_PRIVATE, 3, 0) = 0x7fbc1c8b3000
close(3)                                = 0
access("/etc/ld.so.nohwcap", F_OK)      = -1 ENOENT (No such file or directory)
openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libc.so.6", O_RDONLY|O_CLOEXEC) = 3
read(3, "\177ELF\2\1\1\3\0\0\0\0\0\0\0\0\3\0>\0\1\0\0\0\20\35\2\0\0\0\0\0"..., 832) = 832
fstat(3, {st_mode=S_IFREG|0755, st_size=2030928, ...}) = 0
mmap(NULL, 8192, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7fbc1c8b1000
mmap(NULL, 4131552, PROT_READ|PROT_EXEC, MAP_PRIVATE|MAP_DENYWRITE, 3, 0) = 0x7fbc1c2a5000
mprotect(0x7fbc1c48c000, 2097152, PROT_NONE) = 0
mmap(0x7fbc1c68c000, 24576, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x1e7000) = 0x7fbc1c68c000
mmap(0x7fbc1c692000, 15072, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_ANONYMOUS, -1, 0) = 0x7fbc1c692000
close(3)                                = 0
arch_prctl(ARCH_SET_FS, 0x7fbc1c8b2540) = 0
mprotect(0x7fbc1c68c000, 16384, PROT_READ) = 0
mprotect(0x606000, 4096, PROT_READ)     = 0
mprotect(0x7fbc1c8bf000, 4096, PROT_READ) = 0
munmap(0x7fbc1c8b3000, 47425)           = 0
brk(NULL)                               = 0x163a000
brk(0x165b000)                          = 0x165b000
openat(AT_FDCWD, "/usr/lib/locale/locale-archive", O_RDONLY|O_CLOEXEC) = 3
fstat(3, {st_mode=S_IFREG|0644, st_size=2999664, ...}) = 0
mmap(NULL, 2999664, PROT_READ, MAP_PRIVATE, 3, 0) = 0x7fbc1bfc8000
close(3)                                = 0
uname({sysname="Linux", nodename="master", ...}) = 0
fstat(1, {st_mode=S_IFCHR|0600, st_rdev=makedev(136, 0), ...}) = 0
write(1, "4.4.0-142-generic\n", 184.4.0-142-generic
)     = 18
close(1)                                = 0
close(2)                                = 0
exit_group(0)                           = ?
+++ exited with 0 +++

```


## 3. Open Container Initiative OCI
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3963252af916bd8e1b6e95ad1effeedf.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3f14a8242a9cbd41947a732e988c2cf1.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/5e2f51340be9624fdd4a68509771dd4a.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/62d94d5a96722695e240f2cbf8f6c550.png)
## 4. Crictl
参考链接：
[https://kubernetes.io/zh/docs/tasks/debug-application-cluster/crictl/](https://kubernetes.io/zh/docs/tasks/debug-application-cluster/crictl/)
[Kubernetes crictl](https://ghostwritten.blog.csdn.net/article/details/116591151)


```c
root@master:~# docker ps
CONTAINER ID        IMAGE                  COMMAND                  CREATED             STATUS              PORTS               NAMES
daa4fcda3062        calico/node            "start_runit"            56 minutes ago      Up 56 minutes                           k8s_calico-node_calico-node-ngbm8_kube-system_837fbf7e-0060-4f5c-bd62-fdecf5f7e334_0
47f0bd50cfb9        10cc881966cf           "/usr/local/bin/kube…"   2 days ago          Up 2 days                               k8s_kube-proxy_kube-proxy-lfkn9_kube-system_08f4f57e-d10b-4efe-99d7-33509c6492b0_0
bc34cf5b8061        k8s.gcr.io/pause:3.2   "/pause"                 2 days ago          Up 2 days                               k8s_POD_calico-node-ngbm8_kube-system_837fbf7e-0060-4f5c-bd62-fdecf5f7e334_0
e687794da368        k8s.gcr.io/pause:3.2   "/pause"                 2 days ago          Up 2 days                               k8s_POD_kube-proxy-lfkn9_kube-system_08f4f57e-d10b-4efe-99d7-33509c6492b0_0
8c953da2a0f3        3138b6e3d471           "kube-scheduler --au…"   2 days ago          Up 2 days                               k8s_kube-scheduler_kube-scheduler-master_kube-system_81d2d21449d64d5e6d5e9069a7ca99ed_0
2597fa68a300        b9fa1895dcaa           "kube-controller-man…"   2 days ago          Up 2 days                               k8s_kube-controller-manager_kube-controller-manager-master_kube-system_360cd07520ba8dce55b5d403c66acf83_0
27b65081b111        ca9843d3b545           "kube-apiserver --ad…"   2 days ago          Up 2 days                               k8s_kube-apiserver_kube-apiserver-master_kube-system_ee31a01764366141f7c85e23f94828f8_0
ad3166e92a06        0369cf4303ff           "etcd --advertise-cl…"   2 days ago          Up 2 days                               k8s_etcd_etcd-master_kube-system_77699ae6105937dbb48c0a720843ce8e_0
794740e27507        k8s.gcr.io/pause:3.2   "/pause"                 2 days ago          Up 2 days                               k8s_POD_kube-scheduler-master_kube-system_81d2d21449d64d5e6d5e9069a7ca99ed_0
c666f003a0ad        k8s.gcr.io/pause:3.2   "/pause"                 2 days ago          Up 2 days                               k8s_POD_kube-controller-manager-master_kube-system_360cd07520ba8dce55b5d403c66acf83_0
1fb5d789dd68        k8s.gcr.io/pause:3.2   "/pause"                 2 days ago          Up 2 days                               k8s_POD_kube-apiserver-master_kube-system_ee31a01764366141f7c85e23f94828f8_0
22a1ebc8d2d5        k8s.gcr.io/pause:3.2   "/pause"                 2 days ago          Up 2 days                               k8s_POD_etcd-master_kube-system_77699ae6105937dbb48c0a720843ce8e_0



root@master:~# crictl ps
CONTAINER ID        IMAGE                                                                                 CREATED             STATE               NAME                      ATTEMPT             POD ID
daa4fcda30622       calico/node@sha256:04b8a7be6a277000ea4ae12f32692b2f5532cd095fe5d6b6e3ff876d750c336a   About an hour ago   Running             calico-node               0                   bc34cf5b8061f
47f0bd50cfb95       10cc881966cfd                                                                         2 days ago          Running             kube-proxy                0                   e687794da3683
8c953da2a0f34       3138b6e3d4712                                                                         2 days ago          Running             kube-scheduler            0                   794740e275074
2597fa68a300d       b9fa1895dcaa6                                                                         2 days ago          Running             kube-controller-manager   0                   c666f003a0ad6
27b65081b1114       ca9843d3b5454                                                                         2 days ago          Running             kube-apiserver            0                   1fb5d789dd685
ad3166e92a063       0369cf4303ffd                                                                         2 days ago          Running             etcd                      0                   22a1ebc8d2d50
root@master:~# 


root@master:~# crictl pods
POD ID              CREATED             STATE               NAME                             NAMESPACE           ATTEMPT
bc34cf5b8061f       2 days ago          Ready               calico-node-ngbm8                kube-system         0
e687794da3683       2 days ago          Ready               kube-proxy-lfkn9                 kube-system         0
794740e275074       2 days ago          Ready               kube-scheduler-master            kube-system         0
c666f003a0ad6       2 days ago          Ready               kube-controller-manager-master   kube-system         0
1fb5d789dd685       2 days ago          Ready               kube-apiserver-master            kube-system         0
22a1ebc8d2d50       2 days ago          Ready               etcd-master                      kube-system         0
root@master:~# 

```
## 5. Sandbox Runtime Kata containers
参考链接：
[https://katacontainers.io/](https://katacontainers.io/)
[https://github.com/kata-containers/kata-containers](https://github.com/kata-containers/kata-containers)
[https://www.hi-linux.com/posts/23259.html](https://www.hi-linux.com/posts/23259.html)
[Kubernetes kata-container 介绍](https://ghostwritten.blog.csdn.net/article/details/116596343)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/69c8e49966723f117ca4daec0623d8fc.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/730e01c4b832117db1c7fd8c1cd2ac6c.png)
## 6. Sandbox Runtime gVisor
参考链接：
[https://github.com/google/gvisor](https://github.com/google/gvisor)
[https://gvisor.dev/](https://gvisor.dev/)
[https://gvisor.dev/docs/](https://gvisor.dev/docs/)![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/bdc1378294bad0bf297431e8c4c9f395.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e8e695000681a791772dbbe864540b84.png)
## 7. Create and use RuntimeClasses
参考链接：
[https://kubernetes.io/docs/concepts/containers/runtime-class/](https://kubernetes.io/docs/concepts/containers/runtime-class/)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/fdb32424c66f3b688a17ba719b9fd101.png)

```c
root@master:~/cks/runtimeclass# vim rc.yaml
apiVersion: node.k8s.io/v1beta1
kind: RuntimeClass
metadata:
  name: gvisor
handler: runsc



root@master:~/cks/runtimeclass# k -f rc.yaml  create
runtimeclass.node.k8s.io/gvisor created

root@master:~/cks/runtimeclass# k get runtimeclass
NAME     HANDLER   AGE
gvisor   runsc     8s


root@master:~/cks/runtimeclass#  k run gvisor --image=nginx -oyaml --dry-run=client > pod.yaml
root@master:~/cks/runtimeclass# vim pod.yaml


root@master:~/cks/runtimeclass# k get pods
NAME     READY   STATUS              RESTARTS   AGE
gvisor   1/1     ContainerCreating   0          19h


root@master:~# k describe pods gvisor
..........
Events:
  Type     Reason                  Age                     From     Message
  ----     ------                  ----                    ----     -------
  Warning  FailedCreatePodSandBox  47h (x1778 over 2d19h)  kubelet  Failed to create pod sandbox: rpc error: code = Unknown desc = RuntimeHandler "runsc" not supported

```
需要接下来的步骤
## 8. Install and use gVisor
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e89df52ce19bd5ea5c4e72f8d4242fe0.png)

```c
root@master:~# bash <(curl -s https://raw.githubusercontent.com/killer-sh/cks-course-environment/master/course-content/microservice-vulnerabilities/container-runtimes/gvisor/install_gvisor.sh)
```

或者执行
```c
root@master:~/cks/runtimeclass# vim  install_gvisor.sh 
#!/usr/bin/env bash
# IF THIS FAILS then you can try to change the URL= further down from latest to a specific release
# https://gvisor.dev/docs/user_guide/install

# gvisor
sudo apt-get update && \
sudo apt-get install -y \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common


# install from web
{
  set -e
  URL=https://storage.googleapis.com/gvisor/releases/release/latest
#  URL=https://storage.googleapis.com/gvisor/releases/release/20201130.0 # try this version instead if latest doesn't work for you
  wget ${URL}/runsc ${URL}/runsc.sha512 \
    ${URL}/gvisor-containerd-shim ${URL}/gvisor-containerd-shim.sha512 \
    ${URL}/containerd-shim-runsc-v1 ${URL}/containerd-shim-runsc-v1.sha512
  sha512sum -c runsc.sha512 \
    -c gvisor-containerd-shim.sha512 \
    -c containerd-shim-runsc-v1.sha512
  rm -f *.sha512
  chmod a+rx runsc gvisor-containerd-shim containerd-shim-runsc-v1
  sudo mv runsc gvisor-containerd-shim containerd-shim-runsc-v1 /usr/local/bin
}

# containerd enable runsc
mkdir -p /etc/containerd
cat > /etc/containerd/config.toml <<EOF
disabled_plugins = ["restart"]
[plugins.linux]
  shim_debug = true
[plugins.cri.containerd.runtimes.runsc]
  runtime_type = "io.containerd.runsc.v1"
EOF

# crictl should use containerd as default
{
cat <<EOF | sudo tee /etc/crictl.yaml
runtime-endpoint: unix:///run/containerd/containerd.sock
EOF
}

systemctl restart containerd

# kubelet should use containerd
{
cat <<EOF | sudo tee /etc/default/kubelet
KUBELET_EXTRA_ARGS="--container-runtime remote --container-runtime-endpoint unix:///run/containerd/containerd.sock"
EOF
}
systemctl daemon-reload
systemctl restart kubelet

```

```c
root@master:~/cks/runtimeclass# bash install_gvisor.sh
root@master:~/cks/runtimeclass# systemctl status kubelet
● kubelet.service - kubelet: The Kubernetes Node Agent
   Loaded: loaded (/lib/systemd/system/kubelet.service; enabled; vendor preset: enabled)
  Drop-In: /etc/systemd/system/kubelet.service.d
           └─10-kubeadm.conf
   Active: active (running) since Thu 2021-05-13 20:30:54 PDT; 4min 42s ago
     Docs: https://kubernetes.io/docs/home/
 Main PID: 24673 (kubelet)
    Tasks: 14
   Memory: 93.2M
      CPU: 7.489s
   CGroup: /system.slice/kubelet.service
           └─24673 /usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet


```
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ad1701d096d1381ed2cab9c7b1f8cc0a.png)

