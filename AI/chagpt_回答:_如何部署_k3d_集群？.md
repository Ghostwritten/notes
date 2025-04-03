![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/87904dcb3cd9c330c79e2dc2c371aeda.png)
写出k3d部署命令步骤
----------
K3d是一个快速部署Kubernetes的工具，部署步骤如下：
1. 下载安装K3d：`curl -s https://raw.githubusercontent.com/rancher/k3d/master/install.sh | bash`
2. 创建K3d集群：`k3d create --name my-cluster --api-port 6551 --publish 8443:8443`
3. 启动K3d集群：`k3d start --name my-cluster`
4. 配置Kubectl：`export KUBECONFIG="$(k3d get-kubeconfig --name='my-cluster')"`
5. 验证K3d集群：`kubectl get nodes`
