


## 1. 准备
- 三台物理机搭建 exsi
- 一台部署 vcenter 管理三台 exsi
- 每台物理机两张网卡，连接万兆交换机

登陆 vcenter 确认网卡配置
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b62051395aee509788debfb1d7dbd4f8.png)

## 2.  创建vsan网络

###  2.1 创建 vSphere Distributed Switch （vds）
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/7edaa04e4a5e8bc472af3a9e4c7e35fe.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0ad8e13039ceef7ab0bfbea7a30ac2ac.png)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c37bb4375f852a25cd87ea49bbd79fee.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4a4dec390112c831bbb8976b431c59f6.png)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f7d509a9d7534c25c57601a34f11f96a.png)

### 2.2  添加管理主机
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b2c457b6a612f88847b993f4dde02205.png)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/31de83f9c491d7c6b98d07d95ec3d6f6.png)


![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/24e261919f4ddec8619bc18ca752e67d.png)

不确定
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/26017c3b02766a5eddc9c69b39ae1a0e.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f4ad44167ff0fcf5822f29ed66d592bc.png)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/45563e57bd1446e73d74e53160f78668.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ba1ed9b73242589f3d7cdefef0f8a78e.png)


![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/85009e05f4b960be29c6b095693a8fc0.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/13fe35a39f1590c195136b7bb64572ee.png)


![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e425ad676008587a4ab4f6113c06d9fd.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/5a6161c20c8f54fe34b3ee64ac7d6a31.png)

### 2.3  添加 networking

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/84c070fab7630bf483f2be8c8a4438d6.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8065db6b6372cc8e011dd664f2004016.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/7668ce28265414e7859896d208265c38.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/daa540303f26f2798dc7667dfb7a558d.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/400d1dfe66d90579313e0270cdb45317.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a2d6adbdb7ca8d87e6c2210ed9e3ac7c.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/de61f50376095a7697459edc8b1b64fd.png)
确认状态
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d4219b150982754b45debc1937b641cf.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6510d8d9f7741feae08987507d37c98b.png)


## 3. 删除

###  3.1 删除 vmkernel adapter 
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3f012f9c45f1650be14606643c64a8a6.png)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/58d14eb0ba2b72661b0da085caf18493.png)


### 3.2  删除 hosts





![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/aa5036270bb6f3b9bb0bf8a4a1855942.png)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6948803f42ada0128b44e656c3bd0368.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/cbb75171000ed8b2bc6f9ad5f911d4b8.png)
###  3.3 删除 DSwitch
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/89dbbcc982b313bae8bf2aa001a16608.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/94abd141cac06140bd4b515b32e9e5f5.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/fbb1eeb0b9e69f9e3df72908d7d742f8.png)
参考：
- [Unable to Remove a Host from a vSphere Distributed Switch](https://docs.vmware.com/en/VMware-vSphere/6.5/com.vmware.vsphere.troubleshooting.doc/GUID-038AC93F-D710-48ED-8E3B-258A23FB2930.html)
