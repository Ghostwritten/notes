#  Gitlab 基础配置


----
## 1. 邮箱发送短信

```bash
$ docker exec -ti gitlab bash
$ vi /etc/gitlab/gitlab.rb
gitlab_rails['smtp_enable'] = true
gitlab_rails['smtp_address'] = "smtp-mail.outlook.com"
gitlab_rails['smtp_port'] = 25
gitlab_rails['smtp_user_name'] = "<xxxx>@outlook.com"
gitlab_rails['smtp_password'] = "<xxxxx>"
gitlab_rails['smtp_domain'] = "outlook.com"
gitlab_rails['smtp_authentication'] = "login"
gitlab_rails['smtp_enable_starttls_auto'] = true
gitlab_rails['smtp_tls'] = false

gitlab_rails['gitlab_email_enabled'] = true
gitlab_rails['gitlab_email_from'] = '<xxxx>@outlook.com'
gitlab_rails['gitlab_email_display_name'] = 'Gitlab'

```
保存退出修改，执行命令`gitlab-ctl reconfigure`重新配置gitlab

```bash
$ gitlab-ctl reconfigure
```

执行命令`gitlab-ctl console`测试发邮件，进入控制台之后执行命令


```bash
Notify.test_email('<xxxxxx>@139.com', '邮件标题', '邮件正文').deliver_now
```

```bash
irb(main):001:0> Notify.test_email('<xxxxxx>@139.com', '邮件标题', '邮件正文').deliver_now
Delivered mail 61b9f42db6726_4fc5a503772f@gitlab.example.com.mail (3732.4ms)
=> #<Mail::Message:154720, Multipart: false, Headers: <Date: Wed, 15 Dec 2021 13:57:01 +0000>, <From: Gitlab <zoxun@outlook.com>>, <Reply-To: Gitlab <noreply@gitlab.example.com>>, <To: 13522947651@139.com>, <Message-ID: <61b9f42db6726_4fc5a503772f@gitlab.example.com.mail>>, <Subject: 邮件标题>, <Mime-Version: 1.0>, <Content-Type: text/html; charset=UTF-8>, <Content-Transfer-Encoding: 7bit>, <Auto-Submitted: auto-generated>, <X-Auto-Response-Suppress: All>>
```
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c98a53ae11e03dd7e90d5090e89b418a.png)

## 2. 注册账号
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/906755db8946c31857eed61dc0992b6d.png)
填写自己的名字、邮箱、密码
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/84ef9dd555787daf94c25d517ccfdc67.png)
注册后等待管理员验证通过。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b517061080eb883fa6c82344136e04cf.png)

登陆管理员，找到Admin
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/7a76c686c870a991f6cab1f62bdd58eb.png)
找到管理user的界面
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/38490e4e16207b6a017c40a5cc5301f2.png)
发现新注册的用户
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/9cb07252bf6e7e2c462aef7a6bec0c35.png)
点击通过或拒绝。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/dc4cb90d3996e63f4b88ebc92cdca840.png)
通过后，你也可以对此用户进行限制访问或删除。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/7951281a4db61951372e9cd1b79c37d8.png)
通过后，租户登陆，选择用户角色
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d523657e7da1a82f0fbb120701e5660d.png)
创建项目或寻找项目
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/25dff059fb166adfe7a47014ddcba0fe.png)
与管理员界面的区别缺少admin的权限
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d925fcf2f4edf63b78b95c471d742938.png)
回到管理员的user用户管理界面，发现xiaoming已经被激活。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/db2a8d20041289ffc236bad7d9a6383c.png)
##  3. 创建项目
第一种
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d21a666f1af967ea1b7268bb1ee21b0d.png)
第二种
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e3b518c3766e39db4bdc6c35a059f93f.png)
第三种
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/5ddb10f4cf28f79b9b29f4314a8f6c8d.png)
三类项目
空白项目，导入项目，模板项目
###  3.1 创建空白项目
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e1d889d1d93ba617d20bc61bedecf4a4.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/fcf6e12d92476254cb68387974ac9ad0.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/811317f5e67274b7b0cffe42f451769a.png)
当我们在点击创建之前，如果点击了“`README`”，如下
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/cecdbee1b0a424b01a1d830523ff6377.png)
那创建出来的项目是这样的。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e6e2bd3984013b5c40755ee028f676fd.png)

### 3.2 创建模板项目
当我们选择模板项目时，会有需要各类开发的模板项目供我们选择。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/26676e76d4270fdd3d9d468d64cf6a3d.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c906be63580411c5810dad6980c8361b.png)
这个关于`kubernets`的`gitbook`项目
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a6748da145bbe637894e74163156c42b.png)
我们可以编写自己的`gitbook`了。
###  3.3 导入项目
导入项目，适合以下场景：

 - 项目迁移
 - 借用开源项目定制开发

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/78dfafa1105a8644caed5903308dc623.png)
我们需要一个`token`,这个`token`来自于你选择的平台，而不是来自自己的`gitlab`平台，我这选择了`github`平台。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/6dfade9d10aa0ce010cfc8ecda956f19.png)
我们现在去创建`token`，我们 登陆`github`，选择设置“`setting`”
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/7421140443efe0cac2a6c3ac1902c687.png)
找到“`developer settings`”
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8c90a283b4dd12579b2cf034c35bff1b.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/41c522fe8963e4ba23900c3ca71a65af.png)
根据兴趣随便取
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/127a9ac9a2fd502c3235fd2316b24682.png)
我们创建出来了
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/360f6fa0f1d25a2eda7759d203625ab8.png)

复制到gitlab的这里
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/fdcff2237f29554e65eda9e188defbeb.png)

获取到我的`github`账号下的项目列表，选择其中一个

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/2fe95fce6dad5ca71d0d6f5d9526e8a5.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/2fb7f8d250a1ed50420d50ad4fec0b35.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/07b6fe7ac7050b333ab718826a1cd5e0.png)
开始导入
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8c6a13c658f4af707130529b180eab6a.png)
导入完成。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/9897fc6a9882f148e0796069eccd4f75.png)
查看gitlab导入的项目
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ce48d6b33a4e84cf8c4bd5a92b6e3edb.png)
查看项目内容，然后根据自己的需求开发属于你自己的项目吧。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c5fa105170cff30610db94513ab43e71.png)

## 4. 删除项目
我们要删除这个项目
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/65a2a94cf08c16c8ce0d4d8dcd1b97b5.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/9baed6876f230e3fa0d60dae75bef56f.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c8b8f48072b73e543c5d962f0c465ba7.png)
再次确认
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4ff8b0c811c4af4f8c4a3d8f888934a4.png)
我们找不到了`gitlab-example-demo`了
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d28cc67c004fea2af32f9a609e2c0a31.png)
##  5. gitlab项目上传github
github创建一个空项目
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/bc0725edb86d9bc9983e26fea943a711.png)

```bash
$ git clone http://gitlab.example.com:8081/root/gitlab-example-demo.git
$ cd gitlab-example-demo
$ git remote -v
origin	http://gitlab.example.com:8081/root/gitlab-example-demo.git (fetch)
origin	http://gitlab.example.com:8081/root/gitlab-example-demo.git (push)
$ git remote add hello https://github.com/Ghostwritten/gitlab-example-demo.git
root@yourdomain:/data/gitlab/projects/gitlab-example-demo# git remote -v
hello	https://github.com/Ghostwritten/gitlab-example-demo.git (fetch)
hello	https://github.com/Ghostwritten/gitlab-example-demo.git (push)
origin	http://gitlab.example.com:8081/root/gitlab-example-demo.git (fetch)
origin	http://gitlab.example.com:8081/root/gitlab-example-demo.git (push)

$ git push -u hello
Username for 'https://github.com': ghostwritten
Password for 'https://ghostwritten@github.com': <access token>
Counting objects: 24, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (22/22), done.
Writing objects: 100% (24/24), 2.36 KiB | 201.00 KiB/s, done.
Total 24 (delta 3), reused 0 (delta 0)
remote: Resolving deltas: 100% (3/3), done.
To https://github.com/Ghostwritten/gitlab-example-demo.git
 * [new branch]      master -> master
Branch 'master' set up to track remote branch 'master' from 'hello'.

```
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f3b71c10c785614b541c0eee986e336f.png)

----
相关阅读：

 - [部署gitlab](https://ghostwritten.blog.csdn.net/article/details/121929582)
 - [gitlab-runner部署](https://ghostwritten.blog.csdn.net/article/details/107755143)
 - [Gitlab 基础配置](https://ghostwritten.blog.csdn.net/article/details/121962870)
 - [Create a Continuous Integration (CI) Pipeline in Gitlab](https://blog.csdn.net/xixihahalelehehe/article/details/121941628?spm=1001.2014.3001.5501)
 - [git与gitlab快速学习手册](https://ghostwritten.blog.csdn.net/article/details/121107739)
