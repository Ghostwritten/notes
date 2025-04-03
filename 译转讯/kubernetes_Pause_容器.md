

---

##  1. 简介

Pause 容器，又叫 Infra 容器，本文将探究该容器的作用与原理。

我们知道在 kubelet 的配置中有这样一个参数：

```bash
KUBELET_POD_INFRA_CONTAINER=--pod-infra-container-image=registry.access.redhat.com/rhel7/pod-infrastructure:latest
```
上面是 openshift 中的配置参数，kubernetes 中默认的配置参数是：

```bash
KUBELET_POD_INFRA_CONTAINER=--pod-infra-container-image=gcr.io/google_containers/pause-amd64:3.0
```
Pause 容器，是可以自己来定义，官方使用的 gcr.io/google_containers/pause-amd64:3.0 容器的代码见 [Github](https://github.com/kubernetes/kubernetes/tree/master/build/pause)，使用 C 语言编写。

## 2. Pause 容器特点
镜像非常小，目前在 700KB 左右
永远处于 Pause (暂停) 状态

##  3. Pause 容器背景
像 Pod 这样一个东西，本身是一个逻辑概念。那在机器上，它究竟是怎么实现的呢？这就是我们要解释的一个问题。

既然说 Pod 要解决这个问题，核心就在于如何让一个 Pod 里的多个容器之间最高效的共享某些资源和数据。

因为容器之间原本是被 Linux Namespace 和 cgroups 隔开的，所以现在实际要解决的是怎么去打破这个隔离，然后共享某些事情和某些信息。这就是 Pod 的设计要解决的核心问题所在。

所以说具体的解法分为两个部分：**网络和存储**。

Pause 容器就是为解决 Pod 中的网络问题而生的。

##  4. Pause 容器实现
Pod 里的多个容器怎么去共享网络？下面是个例子：

比如说现在有一个 Pod，其中包含了一个容器 A 和一个容器 B，它们两个就要共享 Network Namespace。在 Kubernetes 里的解法是这样的：它会在每个 Pod 里，额外起一个 Infra container 小容器来共享整个 Pod 的 Network Namespace。

Infra container 是一个非常小的镜像，大概 700KB 左右，是一个 C 语言写的、永远处于 “暂停” 状态的容器。由于有了这样一个 Infra container 之后，其他所有容器都会通过 Join Namespace 的方式加入到 Infra container 的 Network Namespace 中。

所以说一个 Pod 里面的所有容器，它们看到的网络视图是完全一样的。即：它们看到的网络设备、IP 地址、Mac 地址等等，跟网络相关的信息，其实全是一份，这一份都来自于 Pod 第一次创建的这个 Infra container。这就是 Pod 解决网络共享的一个解法。

在 Pod 里面，一定有一个 IP 地址，是这个 Pod 的 Network Namespace 对应的地址，也是这个 Infra container 的 IP 地址。所以大家看到的都是一份，而其他所有网络资源，都是一个 Pod 一份，并且被 Pod 中的所有容器共享。这就是 Pod 的网络实现方式。

由于需要有一个相当于说中间的容器存在，所以整个 Pod 里面，必然是 Infra container 第一个启动。并且整个 Pod 的生命周期是等同于 Infra container 的生命周期的，与容器 A 和 B 是无关的。这也是为什么在 Kubernetes 里面，它是允许去单独更新 Pod 里的某一个镜像的，即：做这个操作，整个 Pod 不会重建，也不会重启，这是非常重要的一个设计。

## 5. Pause 容器的作用
我们检查 node 节点的时候会发现每个 node 上都运行了很多的 pause 容器，例如如下。

```bash
$ docker ps
CONTAINER ID        IMAGE                           COMMAND ...
...
3b45e983c859        gcr.io/google_containers/pause-amd64:3.0    "/pause" ...
...
dbfc35b00062        gcr.io/google_containers/pause-amd64:3.0    "/pause" ...
...
c4e998ec4d5d        gcr.io/google_containers/pause-amd64:3.0    "/pause" ...
...
508102acf1e7        gcr.io/google_containers/pause-amd64:3.0    "/pause" ...    
```
kubernetes 中的 pause 容器主要为每个业务容器提供以下功能：

 - 在 pod 中担任 Linux 命名空间共享的基础；
 - 启用 pid 命名空间，开启 init 进程。

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/9f9f4c99cba6827e6e4531b5641e0791.png)

##  6. 共享命名空间
在 Linux 中，当您运行一个新进程时，该进程会从父进程继承其命名空间。在新命名空间中运行进程的方式是与父进程“取消共享”命名空间，从而创建一个新的命名空间。这是一个使用该`unshare`工具在新的 PID、UTS、IPC 和挂载命名空间中运行 shell 的示例。

```bash
sudo unshare --pid --uts --ipc --mount -f chroot rootfs /bin/sh
```
进程运行后，您可以将其他进程添加到进程的命名空间以形成 pod。`setns`可以使用系统调用将新进程添加到现有命名空间。

pod 中的容器在它们之间共享命名空间。Docker 让您可以稍微自动化该过程，因此让我们看一个如何使用pause容器和共享命名空间从头开始创建 pod 的示例。首先，我们需要使用 Docker 启动 pause 容器，以便我们可以将容器添加到 pod。

```bash
docker run -d --name pause -p 8080:80 gcr.io/google_containers/pause-amd64:3.0
```
然后我们可以为我们的 pod 运行容器。首先，我们将运行 nginx。这将设置 nginx 将请求代理到端口 2368 上的 localhost。

> 请注意，我们还将主机端口 8080 映射到 pause 容器而不是 nginx 容器上的端口 80，因为 pause 容器设置了 nginx将加入的初始网络命名空间。

```bash
$ cat <<EOF >> nginx.conf
error_log stderr;
events { worker_connections  1024; }
http {
    access_log /dev/stdout combined;
    server {
        listen 80 default_server;
        server_name example.com www.example.com;
        location / {
            proxy_pass http://127.0.0.1:2368;
        }
    }
}
EOF
$ docker run -d --name nginx -v `pwd`/nginx.conf:/etc/nginx/nginx.conf --net=container:pause --ipc=container:pause --pid=container:pause nginx
```
然后再为 [ghost](https://github.com/TryGhost/Ghost) 创建一个应用容器，这是一款博客软件。

```bash
$ docker run -d --name ghost --net=container:pause --ipc=container:pause --pid=container:pause ghost
```
现在访问 `http://localhost:8880/` 就可以看到 ghost 博客的界面了。
解析
`pause` 容器将内部的 `80` 端口映射到宿主机的 `8880` 端口，`pause` 容器在宿主机上设置好了网络 `namespace` 后，nginx 容器加入到该网络 namespace 中，我们看到 nginx 容器启动的时候指定了 `--net=container:pause`，ghost 容器同样加入到了该网络 namespace 中，这样三个容器就共享了网络，互相之间就可以使用 localhost 直接通信，`--ipc=contianer:pause --pid=container:pause` 就是三个容器处于同一个 `namespace` 中，`init` 进程为 `pause`，这时我们进入到 ghost 容器中查看进程情况。

```c
# ps aux
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.0   1024     4 ?        Ss   13:49   0:00 /pause
root         5  0.0  0.1  32432  5736 ?        Ss   13:51   0:00 nginx: master p
systemd+     9  0.0  0.0  32980  3304 ?        S    13:51   0:00 nginx: worker p
node        10  0.3  2.0 1254200 83788 ?       Ssl  13:53   0:03 node current/in
root        79  0.1  0.0   4336   812 pts/0    Ss   14:09   0:00 sh
root        87  0.0  0.0  17500  2080 pts/0    R+   14:10   0:00 ps aux
```
在 ghost 容器中同时可以看到 pause 和 nginx 容器的进程，并且 pause 容器的 PID 是 1。而在 Kubernetes 中容器的 `PID=1` 的进程即为容器本身的业务进程。

##  7. 回收僵尸
在 Linux 中，PID 命名空间中的进程形成一棵树，每个进程都有一个父进程。树的根部只有一个进程实际上没有父进程。这是“`init`”进程，它的 PID 为 `1`。

进程可以使用`fork`和`exec`系统调用启动其他进程。当他们这样做时，新进程的父进程就是调用fork系统调用的进程。`fork`用于启动正在运行的进程的另一个副本，并`exec`用于用新进程替换当前进程，保持相同的 PID（为了运行完全独立的应用程序，您需要运行fork 和 exec系统调用。进程创建一个新副本本身作为具有新 PID 的子进程使用fork，然后当子进程运行时，它会检查它是否是子进程并运行`exec`用你真正想要运行的替换自己。大多数语言提供了一种通过单个函数执行此操作的方法）。每个进程在 OS 进程表中都有一个条目。这记录了有关进程状态和退出代码的信息。当一个子进程完成运行时，它的进程表条目会一直保留到父进程使用wait系统调用检索到它的退出代码。这被称为“回收”僵尸进程。

僵尸进程是已经停止运行但它们的进程表条目仍然存在的进程，因为父进程尚未通过wait系统调用检索它。从技术上讲，每个终止的进程在很短的时间内都是僵尸，但它们可以存活更长时间。

wait当父进程在子进程完成后不调用系统调用时，就会出现更长寿的僵尸进程。发生这种情况的一种情况是父进程编写得不好并且简单地省略了wait调用，或者当父进程在子进程之前死亡并且新的父进程没有调用wait它。当进程的父进程在子进程之前死亡时，操作系统将子进程分配给“init”进程或 PID 1。即，init 进程“采用”子进程并成为其父进程。这意味着现在当子进程退出时，新的父进程 (init) 必须调用wait以获取其退出代码，否则它的进程表条目将永远保留并变成僵尸。

在容器中，一个进程必须是每个 PID 命名空间的 `init` 进程。使用 Docker，每个容器通常都有自己的 PID 命名空间，而 `ENTRYPOINT` 进程是 init 进程。但是，正如我在上一篇关于 [Kubernetes pod 的文章](https://www.ianlewis.org/en/what-are-kubernetes-pods-anyway)中所指出的，可以使容器在另一个容器的命名空间中运行。在这种情况下，**一个容器必须承担 init 进程的角色，而其他容器则作为 init 进程的子进程添加到命名空间中**。

在关于 Kubernetes pods 的帖子中，我在一个容器中运行了 nginx，并将 ghost 添加到 nginx 容器的 PID 命名空间中。

```bash
$ docker run -d --name nginx -v `pwd`/nginx.conf:/etc/nginx/nginx.conf -p 8080:80 nginx
$ docker run -d --name ghost --net=container:nginx --ipc=container:nginx --pid=container:nginx ghost
```
在这种情况下，**nginx 承担 PID 1 的角色，并添加 ghost 作为 nginx 的子进程**。这大部分都很好，但从技术上讲，nginx 现在负责任何幽灵孤儿的孩子。例如，如果 `ghost fork` 自己或使用 运行子进程`exec`，并在子进程完成之前崩溃，那么这些子进程将被 nginx 采用。但是，Nginx 的设计并不是为了能够作为 init 进程运行并收获僵尸。这意味着我们可能会拥有很多它们，并且它们将在该容器的整个生命周期内持续存在。

在 Kubernetes pod 中，容器的运行方式与上面大致相同，但是为每个 pod 创建了一个特殊的pause容器。这个pause容器运行一个非常简单的进程，它不执行任何功能，但基本上永远处于休眠状态。它是如此简单，以至于我可以在此处包含撰写本文时的完整源代码：

```bash
/*
Copyright 2016 The Kubernetes Authors.
Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at
    http://www.apache.org/licenses/LICENSE-2.0
Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
*/

#include <signal.h>
#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>
#include <sys/wait.h>
#include <unistd.h>

static void sigdown(int signo) {
  psignal(signo, "Shutting down, got signal");
  exit(0);
}

static void sigreap(int signo) {
  while (waitpid(-1, NULL, WNOHANG) > 0);
}

int main() {
  if (getpid() != 1)
    /* Not an error because pause sees use outside of infra containers. */
    fprintf(stderr, "Warning: pause should be the first process\n");

  if (sigaction(SIGINT, &(struct sigaction){.sa_handler = sigdown}, NULL) < 0)
    return 1;
  if (sigaction(SIGTERM, &(struct sigaction){.sa_handler = sigdown}, NULL) < 0)
    return 2;
  if (sigaction(SIGCHLD, &(struct sigaction){.sa_handler = sigreap,
                                             .sa_flags = SA_NOCLDSTOP},
                NULL) < 0)
    return 3;

  for (;;)
    pause();
  fprintf(stderr, "Error: infinite loop terminated\n");
  return 42;
}
```

如您所见，它不只是睡觉。它执行另一重要功能。它承担 PID 1 的角色，并在僵尸wait进程被父进程孤立时通过调用它们来获取僵尸进程。这样我们就不会在 Kubernetes pod 的 PID 命名空间中堆积僵尸。

值得注意的是，在 PID 命名空间共享方面有很多反复。如果您启用了 PID 命名空间共享，则仅由 pause 容器完成收割僵尸，目前仅在 Kubernetes 1.7+ 中可用。如果使用 Docker 1.13.1+ 运行 Kubernetes 1.7，则默认启用它，除非使用[kubelet 标志](https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet/)( `--docker-disable-shared-pid=true`) 禁用。这在 Kubernetes 1.8 中已[恢复](https://github.com/kubernetes/kubernetes/pull/51634)，现在默认情况下禁用，除非由 kubelet 标志 ( `--docker-disable-shared-pid=false`) 启用。请参阅此 [GitHub 问题](https://github.com/kubernetes/kubernetes/issues/1615)中关于添加对 PID 命名空间共享的支持的讨论。

**如果未启用 PID 命名空间共享，则 Kubernetes pod 中的每个容器都将拥有自己的 PID 1，并且每个容器都需要自己获取僵尸进程**。很多时候这不是问题，因为应用程序不会产生其他进程，但是**僵尸进程耗尽内存**是一个经常被忽视的问题。正因为如此，并且因为 PID 命名空间共享使您能够在同一个 pod 中的容器之间发送信号，所以我真的希望 PID 命名空间共享成为 Kubernetes 中的默认设置。

---

 - 感谢[Ian Matthew Lewis](https://www.ianlewis.org/en/almighty-pause-container)、[jimmysong](https://jimmysong.io/kubernetes-handbook/concepts/pause-container.html) 的文章

参考链接：

 - [The Almighty Pause Container](https://www.ianlewis.org/en/almighty-pause-container)
 - [https://jimmysong.io/kubernetes-handbook/concepts/pause-container.html](https://jimmysong.io/kubernetes-handbook/concepts/pause-container.html)
 - [What are Kubernetes Pods Anyway?](https://www.ianlewis.org/en/what-are-kubernetes-pods-anyway)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/646c500e367037b76a8e6e7713b5b847.gif#pic_center)

