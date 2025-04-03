
#  Gitlab 编排 .gitlab-ci.yaml


----------
##  1. 快速构建.gitlab-ci.yaml
创建一个项目
![.gitlab-ci.yaml](https://i-blog.csdnimg.cn/blog_migrate/4c628bed934245ac6aa58f63198b7582.png)
创建一个模板.gitlab-ci.yaml
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f15656b30d8ced4665b036ac461a84f8.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/5fa8ef927293ccf95be7eafa577f1955.png)
选择Getting-started模板
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/65bb7750897cc3e2db14f9a60a23ca6a.png)

```bash
stages:          # List of stages for jobs, and their order of execution
  - build
  - test
  - deploy

build-job:       # This job runs in the build stage, which runs first.
  stage: build
  script:
    - echo "Compiling the code..."
    - echo "Compile complete."

unit-test-job:   # This job runs in the test stage.
  stage: test    # It only starts when the job in the build stage completes successfully.
  script:
    - echo "Running unit tests... This will take about 60 seconds."
    - sleep 60
    - echo "Code coverage is 90%"

lint-test-job:   # This job also runs in the test stage.
  stage: test    # It can run at the same time as unit-test-job (in parallel).
  script:
    - echo "Linting code... This will take about 10 seconds."
    - sleep 10
    - echo "No lint issues found."

deploy-job:      # This job runs in the deploy stage.
  stage: deploy  # It only runs when *both* jobs in the test stage complete successfully.
  script:
    - echo "Deploying application..."
    - echo "Application successfully deployed."
```
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4b31aeef19377f7639c0c31a92d8eda9.png)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/2ca97b4d221528ada2525af1d46689d3.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/578fd32ad5b27655f1a1cccad1b55cdc.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/e5eb81166d71b470537f03715c0ac839.png)
全部正常。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/60f600362aa877f4391f3b9a48090a2d.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/82502938bc2f01eeae3bce57f4b4152e.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a1b660743f4048b6c5d5644ca2925417.png)
我们充分利用模板文件，可以为我们节省大量的时间。

## 2. python demo ci

 - [使用flask快速搭建website](https://ghostwritten.blog.csdn.net/article/details/122093464)

###  2.1 推送项目至gitlab
记录模块清单
```bash
pip freeze > requestments.txt
```

```bash
git init
git add .
git commit -m "add a new python demo"
git remote add origin root@gitlab.example.com:8081/root/flask_web1.git
git push
```
###  2.2 安装gitlab-runner

```bash
docker run -d --name gitlab-runner --restart always --net=host \
     -v /srv/gitlab-runner/config:/etc/gitlab-runner \
     -v /var/run/docker.sock:/var/run/docker.sock \
     gitlab/gitlab-runner:latest
```

###  2.3 选择一个本地部署的需求
我们就要为该项目注册shell运行的执行器的`runner`

```bash
$ docker run --rm --net=host -it -v /srv/gitlab-runner/config:/etc/gitlab-runner gitlab/gitlab-runner:alpine-v14.4.2 register
Runtime platform                                    arch=amd64 os=linux pid=7 revision=50fc80a6 version=14.4.2
Running in system-mode.                            
                                                   
Enter the GitLab instance URL (for example, https://gitlab.com/):
http://gitlab.example.com:8081
Enter the registration token:
8BxfhdBE2zf8NioR1UFE
Enter a description for the runner:
[yourdomain.com]: flask_demo
Enter tags for the runner (comma-separated):
python
Registering runner... succeeded                     runner=8BxfhdBE
Enter an executor: custom, parallels, docker+machine, kubernetes, docker, docker-ssh, shell, ssh, virtualbox, docker-ssh+machine:
shell
Runner registered successfully. Feel free to start it, but if it's running already the config should be automatically reloaded! 
```
查看配置

```bash
$ cat /srv/gitlab-runner/config/config.toml 
concurrent = 1
check_interval = 0

[session_server]
  session_timeout = 1800

[[runners]]
  name = "demo"
  url = "http://gitlab.example.com:8081/"
  token = "PF41kT9ZV_1DoT6VzcCu"
  executor = "docker"
  [runners.custom_build_dir]
  [runners.cache]
    [runners.cache.s3]
    [runners.cache.gcs]
    [runners.cache.azure]
  [runners.docker]
    tls_verify = false
    image = "ubuntu:20.04"
    dns = ["8.8.8.8", "1.1.1.1"]
    privileged = false
    disable_entrypoint_overwrite = false
    oom_kill_disable = false
    disable_cache = false
    volumes = ["/cache"]
    extra_hosts = ["gitlab.example.com:192.168.211.70"]
    shm_size = 0

..........

[[runners]]
  name = "flask_demo"
  url = "http://gitlab.example.com:8081"
  token = "tvCxfurs7EmRApH7un2a"
  executor = "shell"
  [runners.custom_build_dir]
  [runners.cache]
    [runners.cache.s3]
    [runners.cache.gcs]
    [runners.cache.azure]
```
为此项目注册runner，并不会影响其他项目的项目，因为配置是追加的。


选择一个最接近你的需求的、并修改量小的.gitlab-ci.yaml模板

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/1df37c5942b58406b12e7c5d1f17ae9c.png)
修改.gitlab-ci.yaml
我的本地环境是`ubuntu:8.04`,本地可能没有`flask`，所以要在`.gitlab-ci.yaml`,装完就可以部署，两步走，即两个`stage`。
.gitlab-ci.yaml
```bash
stages:
  - dep          # List of stages for jobs, and their order of execution
  - deploy

dep-job:
  stage: dep
  script:
    - apt -y install python3-pip
    - pip3 install flask

deploy-job:      # This job runs in the deploy stage.
  stage: deploy  # It only runs when *both* jobs in the test stage complete successfully.
  script:
    - echo "Deploying python flask demo application..."
    - python3 app.py"
```




创建一个Dockerfile模板
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/f7c0f7ed629fe06662515f9022532b46.png)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0885b6b66aed4379ab077d7d1fea3d51.png)
修改Dockerfile
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/49cdd6762cc93e9eb1692bc4d34671e1.png)

-----
更多阅读：

 - [部署gitlab ](https://ghostwritten.blog.csdn.net/article/details/121929582)
 - [gitlab-runner部署](https://ghostwritten.blog.csdn.net/article/details/107755143)
 - [Gitlab 基础配置](https://ghostwritten.blog.csdn.net/article/details/121962870)
 - [Create a Continuous Integration (CI) Pipeline in Gitlab](https://blog.csdn.net/xixihahalelehehe/article/details/121941628?spm=1001.2014.3001.5501)
 - [git与gitlab快速学习手册](https://ghostwritten.blog.csdn.net/article/details/121107739)

