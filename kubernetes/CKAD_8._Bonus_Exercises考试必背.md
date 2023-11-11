## Using an alias for kubectl

```bash
$ alias k=kubectl
$ k version
```
## Setting the namespace per context

```bash
$ kubectl config set-context <context-of-question> --namespace=<namespace-of-question>
```

## Deleting Kubernetes objects quickly

```bash
$ kubectl delete pod nginx --grace-period=0 --force
```

## Bash Commands
编写一个bash一线式文件，~/tmp/date.txt每5秒将当前日期写入文件中。创建目录（如果尚不存在）。新日期不会覆盖文件中的现有日期，而是将其追加。
```bash
if [ ! -d ~/tmp ]; then mkdir -p ~/tmp; fi; while true; do echo $(date) >> ~/tmp/date.txt; sleep 5; done;
```
编写一个bash一线式脚本，它定义一个counter初始值为0的变量。每秒递增一次该变量，然后将其值打印到控制台。
```bash
counter=0; while true; do counter=$((counter+1)); echo "$counter"; sleep 1; done;
```
编写一个bash一线式代码，该值定义了一个介于1到100之间的值的变量。在循环中，如果变量的值小于50，则打印该变量的值。如果该值大于或等于50，则中断该循环并打印出消息“结束：$值”。
```bash
while true; do random=$(((RANDOM % 100) + 1)); if [ $random -le 50 ]; then echo "$random"; else echo "END: $random"; break; fi; sleep 1; done;
```
## Kubernetes Object Information
### Finding specific annotations
为带有注释的任何现有Pod打印约10行Pod描述author=John Doe。
```bash
kubectl describe pods | grep -C 10 "author=John Doe"
```
### Finding all labels
打印所有Pod的标签并确定其Pod名称。以YAML格式呈现输出。
```bash
kubectl get pods -o yaml | grep -C 5 labels:
```

