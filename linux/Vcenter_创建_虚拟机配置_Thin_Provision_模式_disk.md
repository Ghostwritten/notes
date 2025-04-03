## 介绍

在vCenter中选择虚拟磁盘格式通常也取决于您的需求和使用情况。

vSphere支持多种虚拟磁盘格式，以下是一些常见的格式：

- `Thick Provision Lazy Zeroed`：这是vSphere中的默认格式。它会预分配虚拟磁盘所需的存储空间，但只有在虚拟机写入数据时才填充磁盘空间。这种格式可用于性能要求较高的应用程序。

- `Thick Provision Eager Zeroed`：与“懒惰填充”相比，这种格式会在虚拟磁盘初始化时完全填充存储空间。这种格式可用于需要对虚拟磁盘进行强制格式化的应用程序，如Oracle RAC。

- `Thin Provision`：这种格式只分配虚拟磁盘实际写入的存储空间，而不是预分配整个磁盘空间。这种格式可以节省存储空间，但可能会对性能产生一定影响。

- `SeSparse`：这种格式是一种“按需压缩”格式，它可以在虚拟磁盘上对非常规工作负载（如大型数据集）进行高效的存储和检索。

如果您需要更高的性能，可以选择“懒惰填充”或“急切填充”格式。如果您需要节省存储空间，则可以选择“Thin Provisioning”格式。而如果您需要对非常规工作负载进行高效存储和检索，则可以选择“SeSparse”格式。

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/749432c5a16fb5a99fb9fda0e3ef5928.png)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/cf273abaf1215d62ec753cfd696f362c.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/48e7f39ab6fd08ccde0a099b15582bba.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ed218047ceac8bc85ce99df0bee7bc7e.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e40e54672f6d84eea1a3dd7d4020f111.png)

