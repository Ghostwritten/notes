pause
在playbook执行的过程中暂停一定时间或者提示用户进行某些操作
常用参数：

 - minutes：暂停多少分钟
 - seconds：暂停多少秒
 - prompt：打印一串信息提示用户操作

示例：

```bash
- name: wait on user input
   pause: prompt="Warning! Detected slight issue. ENTER to continue CTRL-C a to quit" 
- name: timed wait
  pause: seconds=30
```

