

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/34dfeea4abc84da684b0ec7444cc055f.jpeg)






## 1. 前言
在 VMware 虚拟化环境中，vCenter Server 负责管理多个 ESXi 主机，实现集群管理、资源调度、HA/DRS 等功能。将 ESXi 主机加入 vCenter，可以提升管理效率，实现自动化运维。本教程介绍如何将 **ESXi 加入 vCenter Server**。

## 2. 先决条件
在进行操作前，请确保具备以下条件：
- 一台已安装 **VMware ESXi** 的物理或虚拟服务器
- 一个 **vCenter Server（VCSA 或 Windows 版本）**
- 具有 **vCenter 管理员权限**
- ESXi 和 vCenter 之间网络通信正常（检查端口 443、902）
- 已获取 vCenter 访问地址和管理员账号

## 3. 操作步骤
### 步骤 1：登录 vSphere Client
1. 打开浏览器，输入 vCenter Server 地址，例如：
   ```bash
   https://<vcenter-ip>/ui
   ```
2. 使用 vCenter 管理员账户（如 `administrator@vsphere.local`）登录。

### 步骤 2：创建数据中心（可选）
如果 vCenter 还未创建数据中心，可按以下步骤操作：
1. 在左侧导航栏，右键点击 **vCenter 服务器**，选择 **新建数据中心**。
2. 输入数据中心名称（如 `DC01`），点击 **确定**。

### 步骤 3：创建集群（可选）
如果要使用 DRS、HA 等功能，需要先创建集群：
1. 右键点击 **数据中心**，选择 **新建集群**。
2. 输入集群名称（如 `Cluster01`）。
3. 选择启用或禁用 **DRS（动态资源调度）** 和 **HA（高可用）**，然后点击 **创建**。

### 步骤 4：将 ESXi 主机加入 vCenter
1. 右键点击 **数据中心或集群**，选择 **添加主机**。
2. 输入 **ESXi 主机 IP 地址**（如 `192.168.21.81`）。
3. 输入 ESXi **root 用户名和密码**，点击 **下一步**。
4. **确认证书**（如果出现 SSL 证书警告，点击 "是" 或 "接受"）。
5. 选择是否将主机置于维护模式（默认 **否**，除非有特殊需求）。
6. 选择要分配给 ESXi 主机的 **资源池或集群**，点击 **下一步**。
7. **确认摘要信息**，点击 **完成**。

ESXi 加入成功后，你可以在 vSphere Client 左侧树状视图中看到该主机。



![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/1d48697ff33e888a7d9299863d081114.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ad9069041a8aa2520b507e42b968bb42.png)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/00fd1e1546f02ff1643f22130fe22d92.png)


![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/06c8971070f9da4190429df67318d61c.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/06ab4df9e2fdfe40919207fce46b55ee.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/cf27d33a54ead2346a1974570050f9fb.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0df92e05b1fc8f984b13c0d0f1d27e57.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/62902de17d02c9a3eafed564b9934d02.png)

## 4. 常见问题及解决方案
### 问题 1：无法连接到 ESXi 主机
**原因分析：**
- 网络通信不通，检查 ESXi 和 vCenter 服务器是否能互相 ping 通。
- ESXi SSH 未启用，可尝试登录 ESXi 管理页面（https://<esxi-ip>），启用 SSH 并尝试手动连接。

**解决方案：**
- 确保 vCenter 和 ESXi 之间 **防火墙未阻挡** 443、902 端口。
- 在 ESXi 上运行以下命令，查看管理服务状态：
  ```bash
  /etc/init.d/hostd restart
  /etc/init.d/vpxa restart
  ```

### 问题 2：ESXi 版本与 vCenter 版本不兼容
**解决方案：**
- 确保 vCenter 版本 **高于或等于** ESXi 版本。
- 使用 `VMware Compatibility Guide` 检查兼容性。

### 问题 3：添加 ESXi 主机时提示 SSL 证书错误
**解决方案：**
- 如果是自签名证书，直接选择 **接受**。
- 如果是企业环境，建议在 vCenter 配置受信任的 SSL 证书。

## 5. 结论
成功将 ESXi 添加到 vCenter 后，可以通过 vCenter 进行 **虚拟机管理、资源调度、创建集群** 等操作。希望本教程能帮助你顺利完成 ESXi 加入 vCenter 的过程！





