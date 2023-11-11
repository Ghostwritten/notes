排序/var/lib/docker/containers目录以显示哪些容器目录具有最大的日志文件：

```bash
$ du -d1 -h /var/lib/docker/containers | sort -h
```

清除您选择的容器日志文件的内容：

```bash
$ cat /dev/null > /var/lib/docker/containers/container_id/container_log_name
```

