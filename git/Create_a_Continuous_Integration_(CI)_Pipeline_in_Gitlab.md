#  Gitlab 创建持续集成 (CI) Pipeline



---
## 1.简介
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4a5364b4494240321c135a5805ec8a21.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/24015e47738a87ddf3efb42afccad790.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/16260e6871e351e8bf802b8af8c684ff.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f001ee70be5f4e3a3927315e096c3d4b.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/9d7cfa586640ae4730731ee381e6e746.png)
## 2.CI demo
###  2.1 python demo
vscode创建一下目录和文件
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/2ccd47dcd2e4e3c0bd4115e803e1f851.png)
内容：
test_functions.py

```bash
from app.functions import sum

def test_sum():
    assert add(1, 10) == 11
```
functions.py

```bash
def add(a, b):
    return a + b
```

安装测试工具
```bash
$ pip3 install pipenv
Installing collected packages: virtualenv
  WARNING: Failed to write executable - trying to use .deleteme logic
ERROR: Could not install packages due to an OSError: [WinError 2] 系统找不到指定的文件。: 'C:\\Python310\\Scripts\\virtualenv.exe' -> 'C:\\Python310\\Scripts\\virtualenv.exe.deleteme'

#尝试 
pip3 install pipenv --user
Requirement already satisfied: pipenv in c:\python310\lib\site-packages (2021.11.23)
Requirement already satisfied: setuptools>=36.2.1 in c:\python310\lib\site-packages (from pipenv) (57.4.0)
Requirement already satisfied: pip>=18.0 in c:\users\xh\appdata\roaming\python\python310\site-packages (from pipenv) (21.3.1)
Requirement already satisfied: virtualenv-clone>=0.2.5 in c:\python310\lib\site-packages (from pipenv) (0.5.7)  
Requirement already satisfied: certifi in c:\users\xh\appdata\roaming\python\python310\site-packages (from pipenv) (2021.10.8)
Requirement already satisfied: virtualenv in c:\python310\lib\site-packages (from pipenv) (20.10.0)
Requirement already satisfied: six<2,>=1.9.0 in c:\python310\lib\site-packages (from virtualenv->pipenv) (1.16.0)
Requirement already satisfied: distlib<1,>=0.3.1 in c:\python310\lib\site-packages (from virtualenv->pipenv) (0.3.4)
Requirement already satisfied: backports.entry-points-selectable>=1.0.4 in c:\python310\lib\site-packages (from 
virtualenv->pipenv) (1.1.1)
Requirement already satisfied: filelock<4,>=3.2 in c:\python310\lib\site-packages (from virtualenv->pipenv) (3.4.0)
Requirement already satisfied: platformdirs<3,>=2 in c:\python310\lib\site-packages (from virtualenv->pipenv) (2.4.0)


$ pipenv
bash: pipenv: command not found

#尝试
$ python -m pipenv

#安装pytest
$ python -m pipenv install pytest
Creating a virtualenv for this project...
Pipfile: D:\gitlab\Pipfile
Using C:/Python310/python.exe (3.10.0) to create virtualenv...
[   =] Creating virtual environment...created virtual environment CPython3.10.0.final.0-64 in 6580ms
  creator CPython3Windows(dest=C:\Users\XH\.virtualenvs\gitlab-DBb610So, clear=False, no_vcs_ignore=False, global=False)
  seeder FromAppData(download=False, pip=bundle, setuptools=bundle, wheel=bundle, via=copy, app_data_dir=C:\Users\XH\AppData\Local\pypa\virtualenv)
    added seed packages: pip==21.3.1, setuptools==58.3.0, wheel==0.37.0
  activators BashActivator,BatchActivator,FishActivator,NushellActivator,PowerShellActivator,PythonActivator    

Successfully created virtual environment!
Virtualenv location: C:\Users\XH\.virtualenvs\gitlab-DBb610So
Creating a Pipfile for this project...
Installing pytest...
Adding pytest to Pipfile's [packages]...
Installation Succeeded
Pipfile.lock not found, creating...
Locking [dev-packages] dependencies...
Locking [packages] dependencies...
           Building requirements...
Resolving dependencies...
Success!
Updated Pipfile.lock (99a583)!
Installing dependencies from Pipfile.lock (99a583)...
  ================================ 0/0 - 00:00:00
To activate this project's virtualenv, run pipenv shell.
Alternatively, run a command inside the virtualenv with pipenv run.


$ python -m pipenv shell
Launching subshell in virtual environment...

clear
```
创建结构文件`__init__.py`
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/628ed2d78dde6b24b973f8683fddf98c.png)
测试
```bash
$ python -m pytest
============================================= test session starts =============================================
platform win32 -- Python 3.10.0, pytest-6.2.5, py-1.11.0, pluggy-1.0.0
rootdir: D:\gitlab\gitlab_example_en
collected 0 items / 1 error

=================================================== ERRORS ====================================================
__________________________________ ERROR collecting tests/test_functions.py ___________________________________ 
ImportError while importing test module 'D:\gitlab\gitlab_example_en\tests\test_functions.py'.
Hint: make sure your test modules/packages have valid Python names.
Traceback:
C:\Python310\lib\importlib\__init__.py:126: in import_module
    return _bootstrap._gcd_import(name[level:], package, level)
tests\test_functions.py:1: in <module>
    from app.functions import sum
E   ImportError: cannot import name 'sum' from 'app.functions' (D:\gitlab\gitlab_example_en\app\functions.py)   
=========================================== short test summary info =========================================== 
ERROR tests/test_functions.py
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!! Interrupted: 1 error during collection !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!! 
============================================== 1 error in 0.14s =============================================== 
```
没通过

修改内容
test_functions.py

```bash
from app.functions import add

def test_add():
    assert add(1, 10) == 11
```
测试

```bash
$ python -m pytest
============================================= test session starts =============================================
platform win32 -- Python 3.10.0, pytest-6.2.5, py-1.11.0, pluggy-1.0.0
rootdir: D:\gitlab\gitlab_example_en
collected 1 item                                                                                                

tests\test_functions.py .                                                                                [100%]

============================================== 1 passed in 0.03s ==============================================
```
测试通过

### 2.2 部署gitlab

 - [docker部署gitlab](https://ghostwritten.blog.csdn.net/article/details/121929582)
 - [配置本地与gitlab互信](https://blog.csdn.net/xixihahalelehehe/article/details/105240194?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522163958422116780271511591%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fblog.%2522%257D&request_id=163958422116780271511591&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_v2~rank_v29-2-105240194.pc_v2_rank_blog_default&utm_term=gitlab&spm=1018.2226.3001.4450)

登陆gitlab，创建一个空项目`gitlab-example-demo`
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/7a50269b05ec73bddc492d8f5b81bc13.png)

在vscode中的终端`gitlab-example-en`目录执行：
```bash
$ git config --global user.name "Administrator"
$ git config --global user.email "admin@example.com"
$ ssh-keygen -t rsa -C "admin@example.com"
$ cat /c/Users/XH/.ssh/id_rsa.pub
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCtv5pEPO+Esp14+bztwrq9dM4hD8rUF/Du/1tM6l2fKAgpmFoqM3OhMv9D/x7+94AL2+GUPJ5HPqbCDwzGRfUnLi02RnAHXij3+H/ZRzceCEBlCMvqx//+g/fmtYHdDxKlOcjfxlT4aAytKmRtpYPKIzzHd/lwMWtxpgHvnr+gOM6s67eVabOPlH6iyi7UyIoCy5Xg/wg5IDXFCSGfw3FSZS9EaDdWTTfntwGX7jnX3cEiY/kphKC7dvai3B/YUyx6ioZBgTeBN1aakMkaSiyRMEeQ4HmDI4QogiqHMgTNJCiUq5oiDf0JMwrW/m/IJnZemq4W1cheegaxvJKraJFWoIBp6/AOjisjZrMAbZbrpFDLzvsMJcqDgHSLjQd1hXMaLvR1K9JfYKwsjGzR8XaoKRW1742BbtqLq46qmzqW0pHpShGMmbeALAJMvjRqOG7MuTKcVe2CWyfX7QrIFxVucZ0tijlLMjuqZquUnVjsYm+SUujevm7h+IW09esL7j8= admin@example.com

```
赋值`/c/Users/XH/.ssh/id_rsa.pub`到gitlab的ssh_key,如图
![/c/Users/XH/.ssh/id_rsa.pub](https://i-blog.csdnimg.cn/blog_migrate/e760ebb82bf29676dc277f2fe6fdaaed.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a4cf2be7d392cdb32b484a2b7ba3b27a.png)
### 2.3 项目上传gitlab

初始化本地项目，开始上传gitlab仓库
```bash
$ git init
$ git commit -m "add a new demo"
git remote add origin http://192.168.211.70:8081/root/gitlab-example-demo.git
$ git push -u origin master
Username for 'http://192.168.211.70:8081': root
Password for 'http://root@192.168.211.70:8081': 
Enumerating objects: 7, done.
Counting objects: 100% (7/7), done.
Delta compression using up to 4 threads
Compressing objects: 100% (5/5), done.
Writing objects: 100% (7/7), 525 bytes | 262.00 KiB/s, done.
Total 7 (delta 0), reused 0 (delta 0)
To http://192.168.211.70:8081/root/gitlab-example-demo.git
 * [new branch]      master -> master
Branch 'master' set up to track remote branch 'master' from 'origin'.
```
上传成功
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f512cd4125841f36169fdd06ffbc2039.png)

###  2.4 编排.gitlab-ci.yaml
如果在gitlab运行自动化测试，需要用到`.gitlab-ci.yaml`，下面我们开始编写

```bash
$ cat .gitlab-ci.yaml
stages:
  - test

python_tests:
  image: python:3.9
  stage: test
  script:
    - pip3 install pipenv
```

```bash
$ git add .
$ git commit -m "add .gitlab-ci.yaml"
$ git push
```
当我们推送到`gitlab`发现项目无法运行自动测试。我们查明原因
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c2ebd2601f60258899ce0adffda79620.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/970382c7f87247aafdb721402371c222.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/1408867eceaf4ea6c29e745ead4b6234.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/93b6bd2444d72d878e4f6fc8e8d3372e.png)

###  2.5. 部署gitlab-runner
我们原来缺少一个 action runners，也就是`gitlab-runner`
[官方安装gitlab-runner](https://docs.gitlab.com/runner/install/docker.html)

我们可以用一个小的镜像`gitlab/gitlab-runner:alpine-v14.4.2`
```bash
 docker run -d --name gitlab-runner --restart always --net=host \
     -v /srv/gitlab-runner/config:/etc/gitlab-runner \
     -v /var/run/docker.sock:/var/run/docker.sock \
     gitlab/gitlab-runner:alpine-v14.4.2
```

[注册runner](https://docs.gitlab.com/runner/register/index.html#docker)

```bash
$ docker run --rm -it -v /srv/gitlab-runner/config:/etc/gitlab-runner gitlab/gitlab-runner:alpine-v14.4.2 register
Runtime platform                                    arch=amd64 os=linux pid=7 revision=50fc80a6 version=14.4.2
Running in system-mode.                            
                                                   
Enter the GitLab instance URL (for example, https://gitlab.com/):
http://192.168.211.70:8081
Enter the registration token:
6D5mo8iWCLBaVdqcaqjN
Enter a description for the runner:
[329b671ffa00]: gitlab-example
Enter tags for the runner (comma-separated):

Registering runner... succeeded                     runner=6D5mo8iW
Enter an executor: ssh, virtualbox, docker+machine, shell, docker-ssh+machine, kubernetes, custom, docker, docker-ssh, parallels:
docker
Enter the default Docker image (for example, ruby:2.6):
ubuntu:20.04
Runner registered successfully. Feel free to start it, but if it's running already the config should be automatically reloaded! 
```
注册完成后，gitlab界面检查`gitlab-runner`是否注册成功
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a67e0aee4677d4aab09895419f5a6df6.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a6aadc54f1554bddecc0e34ccd059e89.png)
绿色代表成功，回到项目界面，已经开始在跑了。

### 2.6 测试跑起来
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/89a83f6e60ee8d9aed1021932e66a4b8.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3b4fb7e91546e093696621416e7f884b.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d6d5fb55e63c63d9c95ca591adaf678e.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/036a0f03d11b9c43cd985cf12f562547.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/98b0b5477874fe3aab3e87f15a7dd5be.png)

### 2.7 CI变得更安全

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/db76110a39dc53378ac85eb312511ed4.png)

创建普通用户

```bash
apt-get update && apt-get upgrade
useradd -m -s /bin/bash youtube
passwd youtube
usermod -aG sudo youtube
```
禁用关于root用户ssh的根访问

```bash
$ su - youtube
$ vim /etc/ssh/sshd_config
PermitRootLogin no


$ systemctl restart ssh
$ ssh root@192.168.211.70（拒绝）
```
安装Docker
```c
ssh-copy-id youtube@server-IP
apt-get install -y docker.io
#当前用户授权
usermod -aG docker $USER
docker ps
docker run hello-world
```
在youtube用户下重新运行部署新的`gitlab-runner`并注册，继续测试。

-----


<font color=	#3CB371 size=4>视频</font>：[https://mp.weixin.qq.com/s/NKgpZ1CCybkrNakZIlN-Ng](https://mp.weixin.qq.com/s/NKgpZ1CCybkrNakZIlN-Ng)

<font color=	#FF4500 size=4>原创</font>：[https://www.youtube.com/watch?v=6QtJDaycUwA](https://www.youtube.com/watch?v=6QtJDaycUwA)

<font color=	#000000 size=4>github</font>：[https://github.com/Ghostwritten/gitlab-example-demo.git](https://github.com/Ghostwritten/gitlab-example-demo.git)

<font color=	#1E90FF size=4>更多阅读</font>：

 - [部署gitlab ](https://ghostwritten.blog.csdn.net/article/details/121929582)
 - [gitlab-runner部署](https://ghostwritten.blog.csdn.net/article/details/107755143)
 - [Gitlab 基础配置](https://ghostwritten.blog.csdn.net/article/details/121962870)
 - [Create a Continuous Integration (CI) Pipeline in Gitlab](https://blog.csdn.net/xixihahalelehehe/article/details/121941628?spm=1001.2014.3001.5501)
 - [git与gitlab快速学习手册](https://ghostwritten.blog.csdn.net/article/details/121107739)
