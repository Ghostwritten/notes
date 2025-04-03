#  shell wait 等待命令
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0bbbbc8e2dc5aafcf4306114e24340fa.png)




## 1. 简介
bash wait 命令是一个 Shell 命令，它等待后台运行的进程完成并返回退出状态。与等待指定时间的sleep 命令不同，该wait命令等待所有或特定后台任务完成。

## 2. 语法
在 bash 脚本中使用wait命令有不同的方法。下表解释了每个用例。

|命令|	解释|
|--|--|
|wait	|如果没有任何参数，该wait命令会等待所有后台进程完成，然后再继续执行脚本。
|wait <process or job ID>	|添加的 PID 或作业 ID 会等待特定进程结束，然后再继续执行脚本。
|wait -n	|仅等待以下后台进程完成并返回退出状态。
|wait -f	|终止程序首先等待后台任务完成后再退出。

##  3. 示例
### 3.1 等待命令
在 bash 脚本中使用wait时需要了解三个附加参数：

 - 1.`&`命令后的和号 `( )` 表示后台作业。
 - 2.`$!`获取最后一个后台进程的PID。使用多个后台进程时，将先前的 PID 存储在一个变量中。
 - 3.`$?`打印上一个进程的退出状态。

要查看这三个参数如何协同工作，请打开终端窗口并运行：

```bash
sleep 10 &
echo $!
echo $?
```
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/0a625e74daa9cd7480efe038f3581c34.png)
该`$!`参数存储后台进程PID，同时`$?`存储退出状态。退出状态0表示命令成功完成。

### 3.2 单进程等待
1. 首先打开终端并创建一个简单的后台进程：

```bash
sleep 10 &
```
2. 确认作业在后台运行：

```bash
jobs -l
```
3. 使用wait不带任何参数的命令暂停直到进程完成：

```bash
wait
```
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/07b4f19a746f62c8b06614805a919c01.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b6f1afb7614ac416a3e65a0450f01348.png)
10 秒后（由于sleep 10），控制台打印完成消息。

### 3.3 单进程 bash 等待
使用该wait命令指示后台进程必须在脚本内执行的时间点。

1. 例如，在文本编辑器中添加以下代码：

```bash
#!/bin/bash
echo Background process &
echo First message
echo Second message
wait
echo Third message
```
如果后台进程没有完成第一个和第二个进程，则该wait命令调用暂停以等待第二个进程之后后台进程完成，然后再继续执行第三个进程。

2. 将脚本另存为`single_process.sh`。在终端中，更改权限以使脚本可执行：

```bash
sudo chmod +x single_process.sh
```
3. 运行脚本：

```bash
./single_process.sh
```
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/7a631781d057097c9ba9b8e88f78b4ed.png)
后台进程在命令之前的任何时间完成`wait`，并且脚本继续。

### 3.4 多个进程等待
1.打开文本编辑器，添加以下多进程脚本：

```bash
#!/bin/bash
sleep 10 &
sleep 15 &
sleep 5 &
echo $(date +%T)
wait 
echo $(date +%T)
```
wait该脚本在命令之前和之后打印当前时间。没有任何参数，程序会等待所有进程完成。

2. 将脚本另存为test.sh并关闭文件。接下来，使脚本可执行：

```bash
sudo chmod +x test.sh
```

3. 最后，运行程序：

```bash
./test.sh
```
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/5d29595324d4ffd84691c44a511ef6bf.png)
由于进程在后台运行，所有三个进程都在十五秒内完成。

4. 使用相同的脚本测试以下用例：

- 将`-n`参数添加到`<strong>wait</strong>`. 只有最快的过程完成，脚本在十秒后结束。
- 添加作业 ID 以指示脚本应等待哪个作业。例如，`wait %1`暂停以等待进程 1 ( sleep 10) 完成。

### 3.5 多个进程 bash 等待 PID 
1. 与多个进程一起工作时，使用 `PID` 来标识一个进程。下面的示例脚本显示了一个用例：

```bash
#!/bin/bash
echo "Process 1 lasts for 2s" && sleep 2 &
PID=$!
echo "Process 2 lasts for 3s" && sleep 3 &
echo "Current time $(date +%T)"
wait $PID
echo "Process 1 ended at time $(date +%T) with exit status $?"
wait $!
echo "Process 2 ended at time $(date +%T) with exit status $?"
```
2. 将脚本另存为multi_wait.sh。使脚本可执行：

```bash
sudo chmod +x multi_wait.sh
```

3. 运行脚本查看输出：

```bash
./multi_wait.sh
```
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/50a7fdd15f10615c7534f7adb6bb5095.png)
该脚本需要两秒钟来完成第一个过程（由于sleep2）和三秒钟来完成第二个过程。这两个过程同时执行，都在三秒内完成。

参考：
- [Bash wait Command with Examples](https://phoenixnap.com/kb/bash-wait-command)
