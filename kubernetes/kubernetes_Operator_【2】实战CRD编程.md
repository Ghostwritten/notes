
## 1. 内建资源
k8s 的pod、service、 deployment
## 2. CRD
custom resource definition 自定义的k8s资源类型
k8s通过Apisever，在etcd中注册一种新的资源类型，通过实现custom controler来监听资源对象的变化

## 3. operator目的
operator是一个感知应用状态的控制器
coreos推出的简化复杂的有状态的应用管理框架。
通过扩展kubernets API来自动创建、管理和配置应用实例

## 4. operator原理
operator基于third Party Resources 扩展新的应用资源
通过控制器来保证应用于预期状态
operator 就是通过CRD实现定制化的Controller，它与k8s内建的controller遵循同样的运行模式。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/834cec88f5074bea4c3d3f76cdd4bbea.png#pic_center)

## 5. etcd operator
通过kubernetes API观察etcd集群的当前状态
分析当前状态与预期状态
调用集群etcd API或kubernets 

## 6. docker registry
[部署安装](https://blog.csdn.net/xixihahalelehehe/article/details/107406198)

## 7. operator开发流程
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/663d225a7be8eb309fb1b724899a1651.png#pic_center)
开发准备

```bash
$ mkdir -p $GOPATH/src/github.com/operator-framework
$ cd $GOPATH/src/github.com/operator-framework
$ git clone https://github.com/operator-framework/operator-sdk
$ cd operator-sdk
$ git checkout master
$ make dep
$ make install
```
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3a0a7d14c48264be631c3a9a41239c7a.png#pic_center)
k8s.io依赖包放到GOPATH/src目录下
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/de63e47b4b9aaba705389ca92dfdae4e.png#pic_center)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/435f857640d6f6922311d800087270a1.png#pic_center)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a8de7f4ce80829a248265d9d269de043.png#pic_center)

