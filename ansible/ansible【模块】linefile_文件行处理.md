

--

 - [ansible 快速学习手册](https://ghostwritten.blog.csdn.net/article/details/104344960)

---
##  1. 简介
lineinfile:文件内容修改、在某行前面添加一行、在某行后面添加一行、删除某一行、末尾加入一行、替换或添加某一行

##  2. 示例
### 2.1 文件内容修改
其中regexp为要修改的源内容的正则匹配，line为修改后的内容
bbb修改为bbbbbb
```bash
ansible all -m lineinfile -a "dest=/root/test.txt regexp='bbb' line='bbbbbbb'"
```
###  2.2 在某一行前面插入一行

```bash
ansible all -m lineinfile -a "dest=/root/test.txt insertbefore='aa(.*)' line='eeee'"
```
###  2.3 在某一行后面插入一行

```bash
ansible all -m lineinfile -a "dest=/root/test.txt insertafter='aa(.*)' line='eeee'"
```
###  2.4 删除某一行

```bash
ansible all -m lineinfile -a "dest=/root/test.txt regexp='aa(.*)' state=absent"
```
###  2.5 末尾加入一行

```bash
ansible all -m lineinfile -a "dest=/root/test.txt line='hehe'"
```
###  2.6 替换或添加某一行

```bash
ansible all -m lineinfile -a "dest=/root/test.txt regexp='he(.*)' line='lllll' state=present"
```

