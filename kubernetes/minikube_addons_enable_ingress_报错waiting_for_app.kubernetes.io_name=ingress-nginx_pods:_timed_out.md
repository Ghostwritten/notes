```bash
[root@control-plane ~]# more  /tmp/minikube_addons_5e63cb8ceb2e7e06e33b8dd7a9dcbebc0ad5ec5a_0.log
Log file created at: 2022/03/28 23:36:49
Running on machine: localhost
Binary: Built with gc go1.17.7 for linux/amd64
Log line format: [IWEF]mmdd hh:mm:ss.uuuuuu threadid file:line] msg
I0328 23:36:49.391439   13343 out.go:297] Setting OutFile to fd 1 ...
I0328 23:36:49.392626   13343 out.go:344] TERM=xterm,COLORTERM=, which probably does not support color
I0328 23:36:49.392631   13343 out.go:310] Setting ErrFile to fd 2...
I0328 23:36:49.392635   13343 out.go:344] TERM=xterm,COLORTERM=, which probably does not support color
I0328 23:36:49.393005   13343 root.go:315] Updating PATH: /root/.minikube/bin
W0328 23:36:49.394055   13343 root.go:293] Error reading config file at /root/.minikube/config/config.json: open /root/.minikube/config/config.json: no such file or directory
I0328 23:36:49.397015   13343 config.go:176] Loaded profile config "minikube": Driver=none, ContainerRuntime=docker, KubernetesVersion=v1.23.3
I0328 23:36:49.397073   13343 addons.go:65] Setting ingress=true in profile "minikube"
I0328 23:36:49.397085   13343 addons.go:153] Setting addon ingress=true in "minikube"
I0328 23:36:49.397725   13343 host.go:66] Checking if "minikube" exists ...
I0328 23:36:49.418696   13343 exec_runner.go:51] Run: systemctl --version
I0328 23:36:49.475823   13343 kubeconfig.go:92] found "minikube" server: "https://192.168.211.51:8443"
I0328 23:36:49.475923   13343 api_server.go:165] Checking apiserver status ...
I0328 23:36:49.476838   13343 exec_runner.go:51] Run: sudo pgrep -xnf kube-apiserver.*minikube.*
I0328 23:36:49.694120   13343 exec_runner.go:51] Run: sudo egrep ^[0-9]+:freezer: /proc/83479/cgroup
I0328 23:36:49.741903   13343 api_server.go:181] apiserver freezer: "5:freezer:/kubepods/burstable/pod9ee7293ecf157cb8f91b037a0cafe583/8679611813a265b3737224ba291e7597a554ccfe32700a2a48887dd558c3b0d5"
I0328 23:36:49.741986   13343 exec_runner.go:51] Run: sudo cat /sys/fs/cgroup/freezer/kubepods/burstable/pod9ee7293ecf157cb8f91b037a0cafe583/8679611813a265b3737224ba291e7597a554ccfe32700a2a48887dd558c3b0d5/freezer.state
I0328 23:36:49.765475   13343 api_server.go:203] freezer state: "THAWED"
I0328 23:36:49.765588   13343 api_server.go:240] Checking apiserver healthz at https://192.168.211.51:8443/healthz ...
I0328 23:36:49.780980   13343 api_server.go:266] https://192.168.211.51:8443/healthz returned 200:
ok
I0328 23:36:49.782866   13343 out.go:176]   - Using image registry.cn-hangzhou.aliyuncs.com/google_containers/nginx-ingress-controller:v1.1.1
I0328 23:36:49.783560   13343 out.go:176]   - Using image registry.cn-hangzhou.aliyuncs.com/google_containers/kube-webhook-certgen:v1.1.1
I0328 23:36:49.784602   13343 out.go:176]   - Using image registry.cn-hangzhou.aliyuncs.com/google_containers/kube-webhook-certgen:v1.1.1
I0328 23:36:49.829362   13343 addons.go:348] installing /etc/kubernetes/addons/ingress-deploy.yaml
I0328 23:36:49.830895   13343 exec_runner.go:151] cp: memory --> /etc/kubernetes/addons/ingress-deploy.yaml (15662 bytes)
I0328 23:36:49.831565   13343 exec_runner.go:51] Run: sudo cp -a /tmp/minikube3766028856 /etc/kubernetes/addons/ingress-deploy.yaml
I0328 23:36:49.880237   13343 exec_runner.go:51] Run: sudo KUBECONFIG=/var/lib/minikube/kubeconfig /var/lib/minikube/binaries/v1.23.3/kubectl apply -f /etc/kubernetes/addons/ingress-deploy.yaml
I0328 23:36:51.210368   13343 exec_runner.go:84] Completed: sudo KUBECONFIG=/var/lib/minikube/kubeconfig /var/lib/minikube/binaries/v1.23.3/kubectl apply -f /etc/kubernetes/addons/ingress-deploy.yaml: (1.330100311s)
I0328 23:36:51.210445   13343 addons.go:386] Verifying addon ingress=true in "minikube"
I0328 23:36:51.214311   13343 out.go:176] * Verifying ingress addon...
I0328 23:36:51.222278   13343 kapi.go:75] Waiting for pod with label "app.kubernetes.io/name=ingress-nginx" in ns "ingress-nginx" ...
I0328 23:36:51.318181   13343 kapi.go:86] Found 3 Pods for label selector app.kubernetes.io/name=ingress-nginx
I0328 23:36:51.318196   13343 kapi.go:96] waiting for pod "app.kubernetes.io/name=ingress-nginx", current state: Pending: [<nil>]
I0328 23:36:51.835420   13343 kapi.go:96] waiting for pod "app.kubernetes.io/name=ingress-nginx", current state: Pending: [<nil>]
I0328 23:36:52.326237   13343 kapi.go:96] waiting for pod "app.kubernetes.io/name=ingress-nginx", current state: Pending: [<nil>]
I0328 23:36:52.828832   13343 kapi.go:96] waiting for pod "app.kubernetes.io/name=ingress-nginx", current state: Pending: [<nil>]
I0328 23:36:53.361794   13343 kapi.go:96] waiting for pod "app.kubernetes.io/name=ingress-nginx", current state: Pending: [<nil>]
I0328 23:36:53.824038   13343 kapi.go:96] waiting for pod "app.kubernetes.io/name=ingress-nginx", current state: Pending: [<nil>]
I0328 23:36:54.336770   13343 kapi.go:96] waiting for pod "app.kubernetes.io/name=ingress-nginx", current state: Pending: [<nil>]
I0328 23:36:54.858569   13343 kapi.go:96] waiting for pod "app.kubernetes.io/name=ingress-nginx", current state: Pending: [<nil>]
I0328 23:36:55.324919   13343 kapi.go:96] waiting for pod "app.kubernetes.io/name=ingress-nginx", current state: Pending: [<nil>]
I0328 23:36:55.848421   13343 kapi.go:96] waiting for pod "app.kubernetes.io/name=ingress-nginx", current state: Pending: [<nil>]
I0328 23:36:56.338131   13343 kapi.go:96] waiting for pod "app.kubernetes.io/name=ingress-nginx", current state: Pending: [<nil>]
I0328 23:36:56.827384   13343 kapi.go:96] waiting for pod "app.kubernetes.io/name=ingress-nginx", current state: Pending: [<nil>]
I0328 23:36:57.330343   13343 kapi.go:96] waiting for pod "app.kubernetes.io/name=ingress-nginx", current state: Pending: [<nil>]
I0328 23:36:57.844633   13343 kapi.go:96] waiting for pod "app.kubernetes.io/name=ingress-nginx", current state: Pending: [<nil>]
I0328 23:36:58.323505   13343 kapi.go:96] waiting for pod "app.kubernetes.io/name=ingress-nginx", current state: Pending: [<nil>]
I0328 23:36:58.826469   13343 kapi.go:96] waiting for pod "app.kubernetes.io/name=ingress-nginx", current state: Pending: [<nil>]
I0328 23:36:59.338902   13343 kapi.go:96] waiting for pod "app.kubernetes.io/name=ingress-nginx", current state: Pending: [<nil>]
I0328 23:36:59.832431   13343 kapi.go:96] waiting for pod "app.kubernetes.io/name=ingress-nginx", current state: Pending: [<nil>]
I0328 23:37:00.325000   13343 kapi.go:96] waiting for pod "app.kubernetes.io/name=ingress-nginx", current state: Pending: [<nil>]
I0328 23:37:00.828614   13343 kapi.go:96] waiting for pod "app.kubernetes.io/name=ingress-nginx", current state: Pending: [<nil>]
I0328 23:37:01.325896   13343 kapi.go:96] waiting for pod "app.kubernetes.io/name=ingress-nginx", current state: Pending: [<nil>]
I0328 23:37:01.824734   13343 kapi.go:96] waiting for pod "app.kubernetes.io/name=ingress-nginx", current I0328 23:37:39.354590   13343 kapi.go:96] waiting for pod "app.kubernetes.io/name=ingress-nginx", current state: Pending: [<nil>]
I0328 23:37:39.828198   13343 kapi.go:96] waiting for pod "app.kubernetes.io/name=ingress-nginx", current state: Pending: [<nil>]
I0328 23:37:40.324084   13343 kapi.go:96] waiting for pod "app.kubernetes.io/name=ingress-nginx", current state: Pending: [<nil>]
I0328 23:37:40.828390   13343 kapi.go:96] waiting for pod "app.kubernetes.io/name=ingress-nginx", current state: Pending: [<nil>]
I0328 23:37:41.341050   13343 kapi.go:96] waiting for pod "app.kubernetes.io/name=ingress-nginx", current state: Pending: [<nil>]
I0328 23:37:41.824371   13343 kapi.go:96] waiting for pod "app.kubernetes.io/name=ingress-nginx", current state: Pending: [<nil>]
I0328 23:37:42.325310   13343 kapi.go:96] waiting for pod "app.kubernetes.io/name=ingress-nginx", current state: Pending: [<nil>]
I0328 23:37:42.873478   13343 kapi.go:108] duration metric: took 51.622054465s to wait for app.kubernetes.io/name=ingress-nginx ...
I0328 23:37:42.873513   13343 addons.go:116] Writing out "minikube" config to set ingress=true...
I0328 23:37:42.906323   13343 out.go:176] * 启动 'ingress' 插件
Log file created at: 2022/03/28 23:38:16
Running on machine: localhost
Binary: Built with gc go1.17.7 for linux/amd64
Log line format: [IWEF]mmdd hh:mm:ss.uuuuuu threadid file:line] msg
I0328 23:38:16.292389   14564 out.go:297] Setting OutFile to fd 1 ...
I0328 23:38:16.292753   14564 out.go:344] TERM=xterm,COLORTERM=, which probably does not support color
I0328 23:38:16.292757   14564 out.go:310] Setting ErrFile to fd 2...
I0328 23:38:16.292760   14564 out.go:344] TERM=xterm,COLORTERM=, which probably does not support color
I0328 23:38:16.292976   14564 root.go:315] Updating PATH: /root/.minikube/bin
W0328 23:38:16.293109   14564 root.go:293] Error reading config file at /root/.minikube/config/config.json: open /root/.minikube/config/config.json: no such file or directory
I0328 23:38:16.293863   14564 config.go:176] Loaded profile config "minikube": Driver=none, ContainerRuntime=docker, KubernetesVersion=v1.23.3
I0328 23:38:16.293886   14564 addons.go:65] Setting ingress=true in profile "minikube"
I0328 23:38:16.293895   14564 addons.go:153] Setting addon ingress=true in "minikube"
W0328 23:38:16.293899   14564 addons.go:165] addon ingress should already be in state true
I0328 23:38:16.294242   14564 host.go:66] Checking if "minikube" exists ...
I0328 23:38:16.294850   14564 exec_runner.go:51] Run: systemctl --version
I0328 23:38:16.303337   14564 kubeconfig.go:92] found "minikube" server: "https://192.168.211.51:8443"
I0328 23:38:16.303373   14564 api_server.go:165] Checking apiserver status ...
I0328 23:38:16.303743   14564 exec_runner.go:51] Run: sudo pgrep -xnf kube-apiserver.*minikube.*
I0328 23:38:16.400994   14564 exec_runner.go:51] Run: sudo egrep ^[0-9]+:freezer: /proc/83479/cgroup
I0328 23:38:16.441352   14564 api_server.go:181] apiserver freezer: "5:freezer:/kubepods/burstable/pod9ee7293ecf157cb8f91b037a0cafe583/8679611813a265b3737224ba291e7597a554ccfe32700a2a48887dd558c3b0d5"
I0328 23:38:16.441431   14564 exec_runner.go:51] Run: sudo cat /sys/fs/cgroup/freezer/kubepods/burstable/pod9ee7293ecf157cb8f91b037a0cafe583/8679611813a265b3737224ba291e7597a554ccfe32700a2a48887dd558c3b0d5/freezer.state
I0328 23:38:16.467837   14564 api_server.go:203] freezer state: "THAWED"
I0328 23:38:16.468092   14564 api_server.go:240] Checking apiserver healthz at https://192.168.211.51:8443/healthz ...
I0328 23:38:16.477061   14564 api_server.go:266] https://192.168.211.51:8443/healthz returned 200:
ok
I0328 23:38:16.478690   14564 out.go:176]   - Using image registry.cn-hangzhou.aliyuncs.com/google_containers/nginx-ingress-controller:v1.1.1
I0328 23:38:16.479419   14564 out.go:176]   - Using image registry.cn-hangzhou.aliyuncs.com/google_containers/kube-webhook-certgen:v1.1.1
Log file created at: 2022/03/30 17:21:14
Running on machine: control-plane
Binary: Built with gc go1.17.7 for linux/amd64
Log line format: [IWEF]mmdd hh:mm:ss.uuuuuu threadid file:line] msg
I0330 17:21:14.007183   24027 out.go:297] Setting OutFile to fd 1 ...
I0330 17:21:14.026334   24027 out.go:344] TERM=xterm,COLORTERM=, which probably does not support color
I0330 17:21:14.026345   24027 out.go:310] Setting ErrFile to fd 2...
I0330 17:21:14.026348   24027 out.go:344] TERM=xterm,COLORTERM=, which probably does not support color
I0330 17:21:14.026555   24027 root.go:315] Updating PATH: /root/.minikube/bin
W0330 17:21:14.026755   24027 root.go:293] Error reading config file at /root/.minikube/config/config.json: open /root/.minikube/config/config.json: no such file or directory
I0330 17:21:14.027379   24027 config.go:176] Loaded profile config "minikube": Driver=none, ContainerRuntime=docker, KubernetesVersion=v1.23.3
I0330 17:21:14.027396   24027 addons.go:65] Setting ingress=true in profile "minikube"
I0330 17:21:14.027405   24027 addons.go:153] Setting addon ingress=true in "minikube"
I0330 17:21:14.027824   24027 host.go:66] Checking if "minikube" exists ...
I0330 17:21:14.028117   24027 exec_runner.go:51] Run: systemctl --version
I0330 17:21:14.032820   24027 kubeconfig.go:92] found "minikube" server: "https://192.168.211.51:8443"
I0330 17:21:14.032843   24027 api_server.go:165] Checking apiserver status ...
I0330 17:21:14.033092   24027 exec_runner.go:51] Run: sudo pgrep -xnf kube-apiserver.*minikube.*
I0330 17:21:14.072651   24027 exec_runner.go:51] Run: sudo egrep ^[0-9]+:freezer: /proc/9595/cgroup
I0330 17:21:14.101394   24027 api_server.go:181] apiserver freezer: "9:freezer:/kubepods/burstable/pod2d693b0650b5509a417109e3d2778dd8/0e16b27c856a4c9ae1889569e02cb1c99c06a742419664061969341147da0f64"
I0330 17:21:14.101444   24027 exec_runner.go:51] Run: sudo cat /sys/fs/cgroup/freezer/kubepods/burstable/pod2d693b0650b5509a417109e3d2778dd8/0e16b27c856a4c9ae1889569e02cb1c99c06a742419664061969341147da0f64/freezer.state
I0330 17:21:14.127085   24027 api_server.go:203] freezer state: "THAWED"
I0330 17:21:14.127118   24027 api_server.go:240] Checking apiserver healthz at https://192.168.211.51:8443/healthz ...
I0330 17:21:14.134567   24027 api_server.go:266] https://192.168.211.51:8443/healthz returned 200:
ok
I0330 17:21:14.136056   24027 out.go:176]   - Using image registry.cn-hangzhou.aliyuncs.com/google_containers/kube-webhook-certgen:v1.1.1
I0330 17:21:14.136760   24027 out.go:176]   - Using image registry.cn-hangzhou.aliyuncs.com/google_containers/nginx-ingress-controller:v1.1.1
I0330 17:21:14.138378   24027 out.go:176]   - Using image registry.cn-hangzhou.aliyuncs.com/google_containers/kube-webhook-certgen:v1.1.1
I0330 17:21:14.138741   24027 addons.go:348] installing /etc/kubernetes/addons/ingress-deploy.yaml
I0330 17:21:14.139200   24027 exec_runner.go:151] cp: memory --> /etc/kubernetes/addons/ingress-deploy.yaml (15662 bytes)
I0330 17:21:14.139893   24027 exec_runner.go:51] Run: sudo cp -a /tmp/minikube1921582905 /etc/kubernetes/addons/ingress-deploy.yaml
I0330 17:21:14.188086   24027 exec_runner.go:51] Run: sudo KUBECONFIG=/var/lib/minikube/kubeconfig /var/lib/minikube/binaries/v1.23.3/kubectl apply -f /etc/kubernetes/addons/ingress-deploy.yaml
I0330 17:21:14.731447   24027 addons.go:386] Verifying addon ingress=true in "minikube"
I0330 17:21:14.732227   24027 out.go:176] * Verifying ingress addon...
I0330 17:21:14.734594   24027 kapi.go:75] Waiting for pod with label "app.kubernetes.io/name=ingress-nginx" in ns "ingress-nginx" ...
I0330 17:21:14.762959   24027 kapi.go:86] Found 3 Pods for label selector app.kubernetes.io/name=ingress-nginx
I0330 17:21:14.762967   24027 kapi.go:96] waiting for pod "app.kubernetes.io/name=ingress-nginx", current state: Pending: [<nil>]
I0330 17:21:15.269563   24027 kapi.go:96] waiting for pod "app.kubernetes.io/name=ingress-nginx", current state: Pending: [<nil>]
I0330 17:21:15.770147   24027 kapi.go:96] waiting for pod "app.kubernetes.io/name=ingress-nginx", current state: Pending: [<nil>]
I0330 17:21:16.270511   24027 kapi.go:96] waiting for pod "app.kubernetes.io/name=ingress-nginx", current state: Pending: [<nil>]
I0330 17:21:16.767920   24027 kapi.go:96] waiting for pod "app.kubernetes.io/name=ingress-nginx", current state: Pending: [<nil>]
I0330 20:11:15.621422    5773 kapi.go:96] waiting for pod "app.kubernetes.io/name=ingress-nginx", current state: Pending: [<nil>]
I0330 20:11:15.625001    5773 kapi.go:96] waiting for pod "app.kubernetes.io/name=ingress-nginx", current state: Pending: [<nil>]
I0330 20:11:15.625013    5773 kapi.go:108] duration metric: took 6m0.056801384s to wait for app.kubernetes.io/name=ingress-nginx ...
I0330 20:11:15.626046    5773 out.go:176] 
W0330 20:11:15.626279    5773 out.go:241] X Exiting due to MK_ADDON_ENABLE: run callbacks: running callbacks: [waiting for app.kubernetes.io/name=ingress-nginx pods: timed out waiting for the condition]
W0330 20:11:15.626322    5773 out.go:241] * 
W0330 20:11:15.629059    5773 out.go:241] ╭─────────────────────────────────────────────────────────────────────────────────────────────╮
│                                                                                             │
│    * If the above advice does not help, please let us know:                                 │
│      https://github.com/kubernetes/minikube/issues/new/choose                               │
│                                                                                             │
│    * Please run `minikube logs --file=logs.txt` and attach logs.txt to the GitHub issue.    │
│    * Please also attach the following file to the GitHub issue:                             │
│    * - /tmp/minikube_addons_5e63cb8ceb2e7e06e33b8dd7a9dcbebc0ad5ec5a_0.log                  │
│                                                                                             │
╰─────────────────────────────────────────────────────────────────────────────────────────────╯
```

