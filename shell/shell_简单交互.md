#  shell 简单交互
interactive.sh
```bash
#! /bin/bash
echo "Hey what's Your First Name?"
read a
echo "welcome Mr./Mrs. $a, would you like to tell us, Your Last Name"
read b
echo "Thanks Mr./Mrs. $a $b for telling us your name"
echo "*******************"
echo "Mr./Mrs. $b, it's time to say you good bye"
```

执行：
```bash
$ bash interactive.sh
Hey what's Your First Name?
jack     
welcome Mr./Mrs. jack, would you like to tell us, Your Last Name
zong
Thanks Mr./Mrs. jack zong for telling us your name
*******************
Mr./Mrs. zong, it's time to say you good bye
```

