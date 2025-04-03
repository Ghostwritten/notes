#  Linux Command vgextend 扩展卷组
tags: 设备

![](https://i-blog.csdnimg.cn/blog_migrate/056c54ad2b45ae7f60892d62b1ba06eb.png)




##  1. 简介
在 Linux 中使用逻辑卷管理 (LVM) 为用户提供了创建和使用分区的灵活性。您可以轻松地创建、修改、调整大小和删除各种存储卷。
您可以使用vgextend命令通过物理卷扩展卷组来轻松调整其大小。该命令很简单，只需在使用 vgextend 命令时添加物理卷作为参数即可。

## 2. 什么是 LVM？
LVM 是一个 Linux 系统，负责管理 Linux 系统中的文件系统和逻辑卷。尽管 Linux 中还有其他卷管理工具，但还是推荐使用 LVM 的高级功能。正如我们将在本指南中看到的那样，您可以使用此命令行工具实现很多目标。

为了更好地理解如何使用vgextend命令，我们将创建两个物理卷和一个卷组。完成后，我们将使用 vgextend 将一个物理卷添加到另一个物理卷的卷组中。
## 3. Creating Physical Volumes
![](https://i-blog.csdnimg.cn/blog_migrate/76afbb85d54e61b0c3c5d17af51ada70.png)

我们目前没有物理卷。我们需要一个块设备来初始化物理卷。我们可以使用以下命令列出块设备：
```bash
$ pvs
$ sudo lvmdiskscan
```
![](https://i-blog.csdnimg.cn/blog_migrate/8d16dce0a9710db405e7d8baa0b779c3.png)

由于我们需要创建两个物理卷，我们将使用`/dev/sda1`和`/dev/sdb1`。但在此之前，我们必须卸载块设备。

要卸载块设备，请使用以下命令并替换块设备以匹配您的情况：
```bash
$ sudo umount /dev/sda1

$ sudo umount /dev/sdb1
```
卸载块设备后，我们可以继续使用`pvcreate`命令来初始化物理卷。

要创建两个物理卷，以下命令将是：
```bash
$ sudo pvcreate /dev/sda1
$ sudo pvcreate /dev/sdb1
```
![](https://i-blog.csdnimg.cn/blog_migrate/a111d4b14da94d44c97ae3335d632773.png)
我们成功创建了物理卷，可以使用pvs命令进行确认。
![](https://i-blog.csdnimg.cn/blog_migrate/03ea33f74b1b3d2587c99fa3a60a1187.png)


## 4. Creating Volume Groups
我们需要一个物理卷来创建卷组。让我们首先使用以下命令验证两个物理卷中没有卷组：
```bash
$ sudo vgs
```
![](https://i-blog.csdnimg.cn/blog_migrate/52c5909319ac0b4409e96e9986a8270a.png)
我们现在可以继续为其中一个物理卷创建一个卷组，我们将通过向其中添加另一个物理卷来扩展其大小。因此，要为`/dev/sdb1`创建卷组，以下命令将是：
```bash
$ sudo vgcreate volgroup1 /dev/sdb1
```
![](https://i-blog.csdnimg.cn/blog_migrate/f527b7c059583ca2f2f889441a5fca88.png)
我们将卷组命名为`volgroup`。我们可以使用 `vgs` 命令验证它。
![](https://i-blog.csdnimg.cn/blog_migrate/ada8b3192498fccf71bbfb524277f1de.png)
要获取有关创建的卷组的更多详细信息，请使用以下`vgdisplay`命令：
```bash
$ vgdisplay volgroup1
```
![](https://i-blog.csdnimg.cn/blog_migrate/3ef610df07f0a069e1eff7615d8b64d7.png)

## 5. extend Volume Groups
我们需要关注的是卷组的`Free PE`大小。我们目前有`1919` 免费 PE。要扩展这个大小，我们可以使用`vgextend`命令并添加我们其他物理卷的名称/dev/sda1作为参数。

以下命令将是：
```bash
$ vgextend volgroup1 /dev/sda1
```
![](https://i-blog.csdnimg.cn/blog_migrate/fdffeda864f7910b82d2f0489ac48e32.png)
您应该会收到与上图中类似的成功消息，确认卷组已成功扩展。我们可以验证新的大小，如下所示：
![](https://i-blog.csdnimg.cn/blog_migrate/53493f4c975babf80740c30a96c3958f.png)
您可以注意到我们新的免费 PE 大小从`1919`扩展到`2046`。这就是您可以轻松使用 vgextend Linux 命令通过添加物理卷来扩展卷组大小的方式。


参考：
- [Working With Vgextend Linux Command](https://linuxhint.com/vgextend-linux-command/)
