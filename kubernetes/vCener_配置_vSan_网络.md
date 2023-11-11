


## 1. 准备
- 三台物理机搭建 exsi
- 一台部署 vcenter 管理三台 exsi
- 每台物理机两张网卡，连接万兆交换机

登陆 vcenter 确认网卡配置
![在这里插入图片描述](https://img-blog.csdnimg.cn/79864608232b4771ac5336c843677b84.png)

## 2.  创建vsan网络

###  2.1 创建 vSphere Distributed Switch （vds）
![在这里插入图片描述](https://img-blog.csdnimg.cn/fa678555165e4ed89cc726c79bebb55c.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/b9f32ebbe7094f8eb8f4c6cffb53c553.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/89e856f6c9484414ae6f360d16e616d7.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/29b80ac0265146198b1241eb4b36b252.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/28c03ca189e64dffa9d66822d0270808.png)

### 2.2  添加管理主机
![在这里插入图片描述](https://img-blog.csdnimg.cn/eb267953807742ea96b8172d9dc18bbf.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/31be3831ed81419f8094704d2eec8ebe.png)


![在这里插入图片描述](https://img-blog.csdnimg.cn/456ab131b08c4c2e87bc23816dc99445.png)

不确定
![在这里插入图片描述](https://img-blog.csdnimg.cn/493c2e90ef2e473bad50f2d23657dc66.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/1b14143c6fa042dd83f1146a4e540ac0.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/9ade02e2af4e4677acca633eccb03540.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/0f494a525309442aa31bbe917d6c778d.png)


![在这里插入图片描述](https://img-blog.csdnimg.cn/8435f0c3cc414aac884eaf2713440598.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/86ca7d15e8264421a31fc7ac2ca8a636.png)


![在这里插入图片描述](https://img-blog.csdnimg.cn/b003951cf27345948485fbf8976b18f3.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/aef6acca82a94faa9d7b276c6107597a.png)

### 2.3  添加 networking

![在这里插入图片描述](https://img-blog.csdnimg.cn/d3dc91ba8ab94a3c83c3e4467cf87223.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/81b54cacb600450e9e0dc7e0abf1ea31.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/9e2b8df4d7bd4ed5bc39cdfe48dcb472.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/88cb0d50807b445394a19619ef5a405c.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/027e707b2d224fb1b841df2a19695829.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/abf6eab7fc6847f0a8add1d6b0a644b1.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/28bcd69af23043e3b4ffdf68ba7170d2.png)
确认状态
![在这里插入图片描述](https://img-blog.csdnimg.cn/8d1eb86ecd58498aa8def3cf987adda5.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/9bdbd3443adf4e739690890ea59cbdc6.png)


## 3. 删除

###  3.1 删除 vmkernel adapter 
![在这里插入图片描述](https://img-blog.csdnimg.cn/c1ae50c11bb1443bad2e795e4b7afc1c.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/ffbf7311d2404d43a9c10dfbe4ef38b5.png)


### 3.2  删除 hosts





![在这里插入图片描述](https://img-blog.csdnimg.cn/51dd718b20b7494c9763cf2d07d84b48.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/08db328ba715463f9845b167fa4765bf.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/b10fd3a5cef148dd8b068c7221e4ef3c.png)
###  3.3 删除 DSwitch
![在这里插入图片描述](https://img-blog.csdnimg.cn/008b1e91a821452f8bdfef61f2d4deb1.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/1426efa4ce6346be9b25aab8d6a1d738.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/3dc826eefaa845c5b1e3a06f136bc5c4.png)
参考：
- [Unable to Remove a Host from a vSphere Distributed Switch](https://docs.vmware.com/en/VMware-vSphere/6.5/com.vmware.vsphere.troubleshooting.doc/GUID-038AC93F-D710-48ED-8E3B-258A23FB2930.html)
