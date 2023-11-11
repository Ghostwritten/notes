#  Linux Command vgextend 扩展卷组
tags: 设备

![](https://img-blog.csdnimg.cn/7467b9e33fb94337b378a539febc9f99.png)




##  1. 简介
在 Linux 中使用逻辑卷管理 (LVM) 为用户提供了创建和使用分区的灵活性。您可以轻松地创建、修改、调整大小和删除各种存储卷。
您可以使用vgextend命令通过物理卷扩展卷组来轻松调整其大小。该命令很简单，只需在使用 vgextend 命令时添加物理卷作为参数即可。

## 2. 什么是 LVM？
LVM 是一个 Linux 系统，负责管理 Linux 系统中的文件系统和逻辑卷。尽管 Linux 中还有其他卷管理工具，但还是推荐使用 LVM 的高级功能。正如我们将在本指南中看到的那样，您可以使用此命令行工具实现很多目标。

为了更好地理解如何使用vgextend命令，我们将创建两个物理卷和一个卷组。完成后，我们将使用 vgextend 将一个物理卷添加到另一个物理卷的卷组中。
## 3. Creating Physical Volumes
![](https://img-blog.csdnimg.cn/2d1be94f5b414a349bc6936f2675c9ad.png)

我们目前没有物理卷。我们需要一个块设备来初始化物理卷。我们可以使用以下命令列出块设备：
```bash
$ pvs
$ sudo lvmdiskscan
```
![](https://img-blog.csdnimg.cn/b4c7b8d5ee62422990fa1ef6ad19c960.png)

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
![](https://img-blog.csdnimg.cn/30acc3d17c034667bdb9e905ebd310c7.png)
我们成功创建了物理卷，可以使用pvs命令进行确认。
![](https://img-blog.csdnimg.cn/770eb781f35642c0a5221b73d54cb86c.png)


## 4. Creating Volume Groups
我们需要一个物理卷来创建卷组。让我们首先使用以下命令验证两个物理卷中没有卷组：
```bash
$ sudo vgs
```
![](https://img-blog.csdnimg.cn/52e771f4d28140f198108bad55ba0d11.png)
我们现在可以继续为其中一个物理卷创建一个卷组，我们将通过向其中添加另一个物理卷来扩展其大小。因此，要为`/dev/sdb1`创建卷组，以下命令将是：
```bash
$ sudo vgcreate volgroup1 /dev/sdb1
```
![](https://img-blog.csdnimg.cn/bac656209ab84c80b8df0345dcf9540a.png)
我们将卷组命名为`volgroup`。我们可以使用 `vgs` 命令验证它。
![](https://img-blog.csdnimg.cn/af42f0fe831e4d92a51276d86c4568f9.png)
要获取有关创建的卷组的更多详细信息，请使用以下`vgdisplay`命令：
```bash
$ vgdisplay volgroup1
```
![](https://img-blog.csdnimg.cn/d2a2fe0fd5034c509e9164457d9625c7.png)

## 5. extend Volume Groups
我们需要关注的是卷组的`Free PE`大小。我们目前有`1919` 免费 PE。要扩展这个大小，我们可以使用`vgextend`命令并添加我们其他物理卷的名称/dev/sda1作为参数。

以下命令将是：
```bash
$ vgextend volgroup1 /dev/sda1
```
![](https://img-blog.csdnimg.cn/0bdd433eb82d4e359fd546a10effd002.png)
您应该会收到与上图中类似的成功消息，确认卷组已成功扩展。我们可以验证新的大小，如下所示：
![](https://img-blog.csdnimg.cn/ed823de54c8948e98ed25c8447e74037.png)
您可以注意到我们新的免费 PE 大小从`1919`扩展到`2046`。这就是您可以轻松使用 vgextend Linux 命令通过添加物理卷来扩展卷组大小的方式。


参考：
- [Working With Vgextend Linux Command](https://linuxhint.com/vgextend-linux-command/)
