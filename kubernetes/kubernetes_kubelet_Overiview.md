#  kubernetes kubelet Overiview
tags: kubelet



[![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/fdb3fd7aa4b3636b730980e68f4c1454.png)](https://www.rottentomatoes.com/m/schindlers_list)
*《辛德勒的名单》*

{% youtube %}
https://www.youtube.com/watch?v=7o7pvoT0S0A
{% endyoutube %}

##  简介
[kubelet](https://github.com/kubernetes/kubelet) 是在每个节点上运行的主要“节点代理”。它可以使用以下之一向 apiserver 注册节点：主机名；覆盖主机名的标志；或云提供商的特定逻辑。

kubelet 根据 `PodSpec` 工作。PodSpec 是描述 pod 的 YAML 或 JSON 对象。kubelet 采用一组通过各种机制（主要通过 apiserver）提供的 PodSpec，并确保这些 PodSpec 中描述的容器运行且健康。kubelet 不管理不是由 Kubernetes 创建的容器。它是一种负责向控制平面服务（API Server）传递信息的服务。它与 ETCD 交互以读取配置详细信息。它与主节点通信以获取命令并有效地工作。

除了来自 apiserver 的 PodSpec 之外，还可以通过三种方式将容器清单提供给 Kubelet。

 - File：在命令行上作为标志传递的路径。将定期监视此路径下的文件是否有更新。监控周期默认为 20 秒，可通过标志配置。
 - HTTP 端点：在命令行上作为参数传递的 HTTP 端点。该端点每 20 秒检查一次（也可以使用标志进行配置）。
 - HTTP 服务器：kubelet 还可以侦听 HTTP 并响应一个简单的 API（当前未规范）以提交新的清单。

##  架构
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8f03062492d13f1abc8bcb2181af3342.png)
##  参数

 - [kubelet Options](https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet/)

