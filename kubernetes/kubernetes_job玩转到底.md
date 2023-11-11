#  kubernetes job
tags: job,cronjob

[![在这里插入图片描述](https://img-blog.csdnimg.cn/be4a0543d7684afbbe08be998faedfb1.png)](https://www.rottentomatoes.com/m/the_help)

*《相助》一部关于黑人种族主题电影*


## 1. 简介
Job对象通常用于运行那些仅需要执行一次的任务（例如数据库迁移，批处理脚本等等）。通过Job对象创建运行的Pod具有高可靠性，因为Job Controller会自动重启运行失败的Pod（例如Pod所在Node重启或宕机）。
Job的本质是确保一个或多个Pod健康地运行直至运行完毕。

## 2. 参数

 - `.spec.completions`   #需要成功执行的次数
 - `.spec.parallelism`     #并发的数量
 - `.spec.template.spec.restartPolicy`     
   .spec.template.spec.restartPolicy属性拥有三个候选值：OnFailure，Never和Always。默认值为Always。它主要用于描述Pod内容器的重启策略。在Job中只能将此属性设置为OnFailure或Never。如果`.spec.template.spec.restartPolicy = OnFailure`，如果Pod内某个容器的exit code不为0，那么Pod就会在其内部重启这个容器。`.spec.template.spec.restartPolicy =  Never`，那么Pod内某个容器exit code不为0时，就不会触发容器的重启
 - `.spec.backoffLimit`
   .spec.backoffLimit用于设置Job的容错次数，默认值为6。当Job运行的Pod失败次数到达.spec.backoffLimit次时，Job Controller不再新建Pod，直接停止运行这个Job，将其运行结果标记为Failure。另外，Pod运行失败后再次运行的时间间隔呈递增状态，例如10s，20s，40s。。。
 - `.spec.activeDeadlineSeconds`
   .spec.activeDeadlineSeconds属性用于设置Job运行的超时时间。如果Job运行的时间超过了设定的秒数，那么此Job就自动停止运行所有的Pod，并将Job退出状态标记为reason:DeadlineExceeded。
  - `ttlSecondsAfterFinished` 
   1.12版本之后，k8s提出了通过TTL自动删除Job的特性，当前仅对job生效，对 Complete 和 Failed 状态的Job都会自动删除，以后会逐步对所有的其他资源对象生效。
   Job pi-with-ttl 的 ttlSecondsAfterFinished 值为 `100`，则，在其结束 100 秒之后，将可以被自动删除
如果 ttlSecondsAfterFinished 被设置为 `0`，则 TTL 控制器在 Job 执行结束后，立刻就可以清理该 Job 及其 Pod
如果 ttlSecondsAfterFinished 值未设置，则 TTL 控制器不会清理该 Job

截止日期
该`.spec.startingDeadlineSeconds`字段是可选的。如果它由于任何原因错过了计划的时间，则表示开始工作的最后期限（以秒为单位）。截止日期之后，cron作业不会开始作业。未能按时完成任务的作业将计为失败的作业。如果未指定此字段，则作业没有截止日期。
CronJob控制器计算出cron作业错过了多少时间表。如果错过了100个以上的计划，则不再计划cron作业。如果.spec.startingDeadlineSeconds未设置，则CronJob控制器将从status.lastScheduleTime现在开始计数错过的日程表。
并发策略
该`.spec.concurrencyPolicy`字段也是可选的。它指定如何处理由该cron作业创建的作业的并发执行。该规范可能仅指定以下并发策略之一：

 - `Allow` （默认）：cron作业允许同时运行的作业
 - `Forbid`：cron作业不允许并发运行；如果是时候开始新的作业并且之前的作业尚未完成，则cron作业会跳过新的作业
 - `Replace`：如果是时候开始新的作业并且之前的作业尚未完成，则cron作业将用新的作业替换当前正在运行的作业





## 3. job创建过程细讲


```bash
apiVersion: batch/v1
kind: Job
metadata:
  name: pi
spec:
  template:
    spec:
      containers:
      - name: pi
        image: resouer/ubuntu-bc 
        command: ["sh", "-c", "echo 'scale=10000; 4*a(1)' | bc -l "]
      restartPolicy: Never
  backoffLimit: 4 #容错4次


$ kubectl create -f job.yaml
```
bc 命令是 Linux 里的“计算器”；-l 表示，我现在要使用标准数学库；而 a(1)，则是调用数学库中的 arctangent 函数，计算 `atan(1)`。这是什么意思呢？中学知识告诉我们：`tan(π/4) = 1`。所以，4*atan(1)正好就是π，也就是 3.1415926…。这其实就是一个计算π值的容器。而通过 `scale=10000`，我指定了输出的小数点后的位数是 10000。在我的计算机上，这个计算大概用时 1 分 54 秒。

**但是，跟其他控制器不同的是，Job 对象并不要求你定义一个 spec.selector 来描述要控制哪些 Pod**


```bash
$ kubectl describe jobs/pi
Name:             pi
Namespace:        default
Selector:         controller-uid=c2db599a-2c9d-11e6-b324-0209dc45a495
Labels:           controller-uid=c2db599a-2c9d-11e6-b324-0209dc45a495
                  job-name=pi
Annotations:      <none>
Parallelism:      1
Completions:      1
..
Pods Statuses:    0 Running / 1 Succeeded / 0 Failed
Pod Template:
  Labels:       controller-uid=c2db599a-2c9d-11e6-b324-0209dc45a495
                job-name=pi
  Containers:
   ...
  Volumes:              <none>
Events:
  FirstSeen    LastSeen    Count    From            SubobjectPath    Type        Reason            Message
  ---------    --------    -----    ----            -------------    --------    ------            -------
  1m           1m          1        {job-controller }                Normal      SuccessfulCreate  Created pod: pi-rq5rl
```
可以看到，这个 Job 对象在创建后，它的 Pod 模板，**被自动加上了一个 controller-uid=< 一个随机字符串 > 这样的 Label。而这个 Job 对象本身，则被自动加上了这个 Label 对应的 Selector，从而 保证了 Job 与它所管理的 Pod 之间的匹配关系。**

而 Job Controller 之所以要使用这种携带了 UID 的 Label，就是为了避免不同 Job 对象所管理的 Pod 发生重合。需要注意的是，**这种自动生成的 Label 对用户来说并不友好，所以不太适合推广到 Deployment 等长作业编排对象上。**

Pod 进入了 Running 状态
```bash
$ kubectl get pods
NAME                                READY     STATUS    RESTARTS   AGE
pi-rq5rl                            1/1       Running   0          10s
```
而几分钟后计算结束，这个 Pod 就会进入 Completed 状态：

```bash
$ kubectl get pods
NAME                                READY     STATUS      RESTARTS   AGE
pi-rq5rl                            0/1       Completed   0          4m
```
这也是我们需要在 Pod 模板中定义 `restartPolicy=Never` 的原因：离线计算的 Pod 永远都不应该被重启，否则它们会再重新计算一遍。**事实上，restartPolicy 在 Job 对象里只允许被设置为 Never 和 OnFailure；而在 Deployment 对象里，restartPolicy 则只允许被设置为 Always。**

此时，我们通过 kubectl logs 查看一下这个 Pod 的日志，就可以看到计算得到的 Pi 值已经被打印了出来：

```bash
$ kubectl logs pi-rq5rl
3.141592653589793238462643383279...
```
这时候，你一定会想到这样一个问题，如果这个离线作业失败了要怎么办？比如，我们在这个例子中定义了 `restartPolicy=Never`，那么离线作业失败后 Job Controller 就会不断地尝试创建一个新 Pod，如下所示：

```bash
$ kubectl get pods
NAME                                READY     STATUS              RESTARTS   AGE
pi-55h89                            0/1       ContainerCreating   0          2s
pi-tqbcz                            0/1       Error               0          5s
```
Job 对象的 spec.backoffLimit 字段里定义了重试次数为 4（即，backoffLimit=4），而**这个字段的默认值是 6**。

需要注意的是，Job Controller 重新创建 Pod 的间隔是呈指数增加的，即下一次重新创建 Pod 的动作会分别发生在 10 s、20 s、40 s …后，而如果你定义的 `restartPolicy=OnFailure`，那么离线作业失败后，**Job Controller 就不会去尝试创建新的 Pod。但是，它会不断地尝试重启 Pod 里的容器**。这也正好对应了 restartPolicy 的含义。

如前所述，当一个 Job 的 Pod 运行结束后，它会进入 Completed 状态。但是，如果这个 Pod 因为某种原因一直不肯结束呢？在 Job 的 API 对象里，有一个 `spec.activeDeadlineSeconds` 字段可以设置最长运行时间，比如：


```bash
spec:
 backoffLimit: 5
 activeDeadlineSeconds: 100
```
一旦运行超过了 100 s，这个 Job 的所有 Pod 都会被终止。并且，你可以在 Pod 的状态里看到终止的原因是 `reason: DeadlineExceeded`。以上，就是一个 Job API 对象最主要的概念和用法了。不过，离线业务之所以被称为 Batch Job，当然是因为它们可以以“Batch”，也就是并行的方式去运行。


##  4. Job Controller 对并行作业的控制方法

在 Job 对象中，负责并行控制的参数有两个：

 - `spec.parallelism`，它定义的是一个 Job 在任意时间最多可以启动多少个 Pod 同时运行；
 - `spec.completions`，它定义的是 Job 至少要完成的 Pod 数目，即 Job 的最小完成数。



```bash
apiVersion: batch/v1
kind: Job
metadata:
  name: pi
spec:
#这个 Job 最大的并行数是 2，而最小的完成数是 4
  parallelism: 2
  completions: 4
  template:
    spec:
      containers:
      - name: pi
        image: resouer/ubuntu-bc
        command: ["sh", "-c", "echo 'scale=5000; 4*a(1)' | bc -l "]
      restartPolicy: Never
  backoffLimit: 4



$ kubectl create -f job.yaml

#两个状态字段，即 DESIRED 和 SUCCESSFUL
#DESIRED 的值，正是 completions 定义的最小完成数。
$ kubectl get job
NAME      DESIRED   SUCCESSFUL   AGE
pi        4         0            3s



$ kubectl get pods
NAME       READY     STATUS    RESTARTS   AGE
pi-5mt88   1/1       Running   0          6s
pi-gmcq5   1/1       Running   0          6s
```
40 s 后，这两个 Pod 相继完成计算。这时我们可以看到，每当有一个 Pod 完成计算进入 Completed 状态时，就会有一个新的 Pod 被自动创建出来，并且快速地从 Pending 状态进入到 `ContainerCreating` 状态：

```bash
$ kubectl get pods
NAME       READY     STATUS    RESTARTS   AGE
pi-gmcq5   0/1       Completed   0         40s
pi-84ww8   0/1       Pending   0         0s
pi-5mt88   0/1       Completed   0         41s
pi-62rbt   0/1       Pending   0         0s

$ kubectl get pods
NAME       READY     STATUS    RESTARTS   AGE
pi-gmcq5   0/1       Completed   0         40s
pi-84ww8   0/1       ContainerCreating   0         0s
pi-5mt88   0/1       Completed   0         41s
pi-62rbt   0/1       ContainerCreating   0         0s
```
紧接着，Job Controller 第二次创建出来的两个并行的 Pod 也进入了 Running 状态：

```bash
$ kubectl get pods 
NAME       READY     STATUS      RESTARTS   AGE
pi-5mt88   0/1       Completed   0          54s
pi-62rbt   1/1       Running     0          13s
pi-84ww8   1/1       Running     0          14s
pi-gmcq5   0/1       Completed   0          54s
```
终，后面创建的这两个 Pod 也完成了计算，进入了 Completed 状态。这时，由于所有的 Pod 均已经成功退出，这个 Job 也就执行完了，所以你会看到它的 SUCCESSFUL 字段的值变成了 4：

```bash
$ kubectl get pods 
NAME       READY     STATUS      RESTARTS   AGE
pi-5mt88   0/1       Completed   0          5m
pi-62rbt   0/1       Completed   0          4m
pi-84ww8   0/1       Completed   0          4m
pi-gmcq5   0/1       Completed   0          5m

$ kubectl get job
NAME      DESIRED   SUCCESSFUL   AGE
pi        4         4            5m
```

## 5.  Job Controller 的工作原理

 - 首先，Job Controller 控制的对象，直接就是 Pod。
 - 其次，Job Controller 在控制循环中进行的调谐（Reconcile）操作，是根据实际在 Running 状态 Pod 的数目、已经成功退出的 Pod 的数目，以及 `parallelism`、`completions`参数的值共同计算出在这个周期里，应该创建或者删除的 Pod 数目，然后调用 Kubernetes API 来执行这个操作。


以创建 Pod 为例。在上面计算 Pi 值的这个例子中，当 Job 一开始创建出来时，实际处于 Running 状态的 Pod 数目 =0，已经成功退出的 Pod 数目 =0，而用户定义的 completions，也就是最终用户需要的 Pod 数目 =4。

所以，在这个时刻，**需要创建的 Pod 数目 = 最终需要的 Pod 数目 - 实际在 Running 状态 Pod 数目 - 已经成功退出的 Pod 数目 = 4 - 0 - 0= 4**。也就是说，Job Controller 需要创建 4 个 Pod 来纠正这个不一致状态。

可是，我们又定义了这个 Job 的 `parallelism=2`。也就是说，我们规定了每次并发创建的 Pod 个数不能超过 2 个。所以，Job Controller 会对前面的计算结果做一个修正，修正后的期望创建的 Pod 数目应该是：2 个。

这时候，Job Controller 就会并发地向 kube-apiserver 发起两个创建 Pod 的请求。类似地，如果在这次调谐周期里，Job Controller 发现实际在 Running 状态的 Pod 数目，比 parallelism 还大，那么它就会删除一些 Pod，使两者相等。综上所述，**Job Controller 实际上控制了，作业执行的并行度，以及总共需要完成的任务数这两个重要参数。而在实际使用时，你需要根据作业的特性，来决定并行度（parallelism）和任务数（completions）的合理取值**。

##  6. 三种job方法
### 6.1 外部管理器 +Job 模板
也是最简单粗暴的用法，这种模式的特定用法是：把 Job 的 YAML 文件定义为一个“模板”，然后用一个外部工具控制这些“模板”来生成 Job。这时，Job 的定义方式如下所示：


```bash
apiVersion: batch/v1
kind: Job
metadata:
  name: process-item-$ITEM
  labels:
    jobgroup: jobexample
spec:
  template:
    metadata:
      name: jobexample
      labels:
        jobgroup: jobexample
    spec:
      containers:
      - name: c
        image: busybox
        command: ["sh", "-c", "echo Processing item $ITEM && sleep 5"]
      restartPolicy: Never
```
以看到，我们在这个 Job 的 YAML 里，定义了 $ITEM 这样的“变量”。所以，在控制这种 Job 时，我们只要注意如下两个方面即可：

 1. 创建 Job 时，替换掉 `$ITEM` 这样的变量；
 2. 所有来自于同一个模板的 Job，都有一个 `jobgroup: jobexample` 标签，也就是说这一组 Job
    使用这样一个相同的标识。
    
而做到第一点非常简单。比如，你可以通过这样一句 shell 把 $ITEM 替换掉：

```bash
$ mkdir ./jobs
$ for i in apple banana cherry
do
  cat job-tmpl.yaml | sed "s/\$ITEM/$i/" > ./jobs/job-$i.yaml
done
```
这样，一组来自于同一个模板的不同 Job 的 yaml 就生成了。接下来，你就可以通过一句 kubectl create 指令创建这些 Job 了：

```bash
$ kubectl create -f ./jobs
$ kubectl get pods -l jobgroup=jobexample
NAME                        READY     STATUS      RESTARTS   AGE
process-item-apple-kixwv    0/1       Completed   0          4m
process-item-banana-wrsf7   0/1       Completed   0          4m
process-item-cherry-dnfu9   0/1       Completed   0          4m
```
###  6.2 拥有固定任务数目的并行 Job
这种模式下，我只关心最后是否有指定数目（spec.completions）个任务成功退出。至于执行时的并行度是多少，我并不关心。比如，我们这个计算 Pi 值的例子，就是这样一个典型的、拥有固定任务数目（completions=4）的应用场景。 它的 `parallelism` 值是 2；或者，你可以干脆不指定 parallelism，直接使用默认的并行度（即：1）。此外，你还可以使用一个工作队列（Work Queue）进行任务分发。这时，Job 的 YAML 文件定义如下所示：

```bash
apiVersion: batch/v1
kind: Job
metadata:
  name: job-wq-1
spec:
  completions: 8
  parallelism: 2
  template:
    metadata:
      name: job-wq-1
    spec:
      containers:
      - name: c
        image: myrepo/job-wq-1
        env:
        - name: BROKER_URL
          value: amqp://guest:guest@rabbitmq-service:5672
        - name: QUEUE
          value: job1
      restartPolicy: OnFailure
```
我们可以看到，它的 completions 的值是：8，这意味着我们总共要处理的任务数目是 8 个。也就是说，总共会有 8 个任务会被逐一放入工作队列里（你可以运行一个外部小程序作为生产者，来提交任务）。所以，一旦你用 kubectl create 创建了这个 Job，它就会以并发度为 2 的方式，每两个 Pod 一组，创建出 8 个 Pod。每个 Pod 都会去连接 BROKER_URL，从 RabbitMQ 里读取任务，然后各自进行处理。这个 Pod 里的执行逻辑，我们可以用这样一段伪代码来表示：

```bash
/* job-wq-1的伪代码 */
queue := newQueue($BROKER_URL, $QUEUE)
task := queue.Pop()
process(task)
exit
```
###  6.3 指定并行度（parallelism），但不设置固定的 completions 的值
此时，你就必须自己想办法，来决定什么时候启动新 Pod，什么时候 Job 才算执行完成。在这种情况下，任务的总数是未知的，所以你不仅需要一个工作队列来负责任务分发，还需要能够判断工作队列已经为空（即：所有的工作已经结束了）。这时候，Job 的定义基本上没变化，只不过是不再需要定义 completions 的值了而已

```bash
apiVersion: batch/v1
kind: Job
metadata:
  name: job-wq-2
spec:
  parallelism: 2
  template:
    metadata:
      name: job-wq-2
    spec:
      containers:
      - name: c
        image: gcr.io/myproject/job-wq-2
        env:
        - name: BROKER_URL
          value: amqp://guest:guest@rabbitmq-service:5672
        - name: QUEUE
          value: job2
      restartPolicy: OnFailure
```
而对应的 Pod 的逻辑会稍微复杂一些，我可以用这样一段伪代码来描述：

```bash
/* job-wq-2的伪代码 */
for !queue.IsEmpty($BROKER_URL, $QUEUE) {
  task := queue.Pop()
  process(task)
}
print("Queue empty, exiting")
exit
```
##  7. CronJob定时

```bash
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: hello
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hello
            image: busybox
            args:
            - /bin/sh
            - -c
            - date; echo Hello from the Kubernetes cluster
          restartPolicy: OnFailure
```
CronJob 是一个 Job 对象的控制器（Controller）
CronJob 与 Job 的关系，正如同 Deployment 与 ReplicaSet 的关系一样。CronJob 是一个专门用来管理 Job 对象的控制器。只不过，它创建和删除 Job 的依据，是 schedule 字段定义的、一个标准的Unix Cron格式的表达式。比如，`"*/1 * * * *"`。这个 Cron 表达式里 */1 中的 * 表示从 0 开始，/ 表示“每”，1 表示偏移量。所以，它的意思就是：从 0 开始，每 1 个时间单位执行一次。

 - Cron 表达式中的五个部分分别代表：分钟、小时、日、月、星期



```bash
$ kubectl create -f ./cronjob.yaml
cronjob "hello" created

# 一分钟后
$ kubectl get jobs
NAME               DESIRED   SUCCESSFUL   AGE
hello-4111706356   1         1         2s


$ kubectl get cronjob hello
NAME      SCHEDULE      SUSPEND   ACTIVE    LAST-SCHEDULE
hello     */1 * * * *   False     0         Thu, 6 Sep 2018 14:34:00 -070
```
需要注意的是，由于定时任务的特殊性，很可能某个 Job 还没有执行完，另外一个新 Job 就产生了。这时候，你可以通过 `spec.concurrencyPolicy` 字段来定义具体的处理策略。比如：

 1. `concurrencyPolicy=Allow`，这也是默认情况，这意味着这些 Job 可以同时存在；
 2. `concurrencyPolicy=Forbid`，这意味着不会创建新的 Pod，该创建周期被跳过；
 3. `concurrencyPolicy=Replace`，这意味着新产生的 Job 会替换旧的、没有执行完的 Job。


而如果某一次 Job 创建失败，这次创建就会被标记为“miss”。当在指定的时间窗口内，miss 的数目达到 100 时，那么 CronJob 会停止再创建这个 Job。这个时间窗口，可以由 `spec.startingDeadlineSeconds` 字段指定。比如 `startingDeadlineSeconds=200`，意味着在过去 200 s 里，如果 miss 的数目达到了 100 次，那么这个 Job 就不会被创建执行了。


## 8. 实战

### 8.1 非并发Job
非并发Job的含义是，Job启动后，只运行一个pod，pod运行结束后整个Job也就立刻结束。
以下是简单的Job配置文件，只包含一个pod，输出圆周率小数点后2000位，运行时间大概为10s：

```bash
apiVersion: batch/v1
kind: Job
metadata:
  name: pi
spec:
  template:
    spec:
      containers:
      - name: pi
        image: perl
        command: ["perl",  "-Mbignum=bpi", "-wle", "print bpi(2000)"]
      restartPolicy: Never
  backoffLimit: 4
```
以上示例无需设置选择器、pod标签。无需设置`.spec.completions`、`.spec.parallelism`，这两个字段的默认值都是1。backoffLimit=4，表示允许pod失败的次数。将以上内容保存成文件并创建Job：

```bash
$ kubectl create -f https://k8s.io/examples/controllers/job.yaml
job "pi" created
```
### 8.2 粗并发Job
本例创建一个Job，但Job要创建多个pod。了解完示例后就明白为什么叫“粗并发”。
本示例需要一个消息队列服务的配合，不详细描述如何部署、填充消息队列服务。假设我们有一个RabbitMQ服务，集群内访问地址为：amqp://guest:guest@rabbitmq-service:5672。其有一个名为job1的队列，队列内有apple banana cherry date fig grape lemon melon共8个成员。
另外假设我们有一个名为gcr.io/<project>/job-wq-1的image，其功能是从队列中读取出一个元素并打印到标准输出，然后结束。注意，它只处理一个元素就结束了。接下来创建如下Job：

```bash
apiVersion: batch/v1
kind: Job
metadata:
  name: job-wq-1
spec:
  completions: 8
  parallelism: 2
  template:
    metadata:
      name: job-wq-1
    spec:
      containers:
      - name: c
        image: gcr.io/<project>/job-wq-1
        env:
        - name: BROKER_URL
          value: amqp://guest:guest@rabbitmq-service:5672
        - name: QUEUE
          value: job1
      restartPolicy: OnFailure
```
上例中，`completions`的值为8，等于job1队列中元素的个数。因为每个成功的pod处理一个元素，所以需要成功8次，job1中的所有成员就会被处理完成。在粗并发模式下，completions的值必需指定，否则其默认值为1，整个Job只处理一个成员就结束了。

上例中，`parallelism`的值是2。虽然需要pod成功8次，但在同一时间，只允许有两个pod**并发**。一个成功结束后，再启动另一个。这个参数的主要目的是控制并发pod的个数，可根据实际情况调整。当然可以不指定，那么默认的并发个数就是1。
env中的内容告诉image如何访问队列。

### 8.3 CronJob 

```bash
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: hello
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hello
            image: busybox
            args:
            - /bin/sh
            - -c
            - date; echo Hello from the Kubernetes cluster
          restartPolicy: OnFailure
```

```bash
kubectl create -f https://k8s.io/examples/application/job/cronjob.yaml
kubectl get cronjob hello
kubectl logs $pods
kubectl delete cronjob hello
```

命令行执行cronjob

```bash
kubectl run hello --schedule="*/1 * * * *" --restart=OnFailure --image=busybox -- /bin/sh -c "date; echo Hello from the Kubernetes cluster"
```


### 8.4. Job的自动清理

```bash
apiVersion: batch/v1
kind: Job
metadata:
  name: pi-with-ttl
spec:
  ttlSecondsAfterFinished: 100
  template:
    spec:
      containers:
      - name: pi
        image: perl
        command: ["perl",  "-Mbignum=bpi", "-wle", "print bpi(2000)"]
      restartPolicy: Never
```



参考：

 - [kubernetes job](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/job/)
 - [Understanding Jobs in Kubernetes](https://levelup.gitconnected.com/understanding-jobs-in-kubernetes-68ac21b272d8)
 - [google cloud running Job](https://cloud.google.com/kubernetes-engine/docs/how-to/jobs?hl=zh_cn)
 - [Kubernetes - Jobs](https://www.tutorialspoint.com/kubernetes/kubernetes_jobs.htm)

