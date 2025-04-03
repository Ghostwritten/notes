
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/30d0542998134fb5b4ba854d6f5031d7.png)



### Linux setfacl 命令详解

在 Linux 系统中，文件权限控制通常通过传统的所有者（Owner）、所属组（Group）、其他用户（Others）模式进行管理。然而，这种传统模式有时难以满足复杂的权限需求，例如需要为多个特定用户设置不同的权限。这时，`setfacl` 命令和 ACL（访问控制列表）便成为了解决方案。

---

#### 一、ACL 和 setfacl 简介

**ACL（Access Control List）** 是一种细粒度的权限管理机制，允许为文件或目录指定多个用户或组的访问权限，从而突破传统权限模型的限制。

**`setfacl` 命令** 是用于设置文件和目录 ACL 的工具。

---

#### 二、基本语法

```bash
setfacl [选项] 权限 文件或目录
```

**常用选项**：
- `-m`：修改或添加 ACL 条目。
- `-x`：移除指定的 ACL 条目。
- `-b`：移除所有 ACL 条目（恢复为默认权限）。
- `-k`：移除默认 ACL 条目。
- `-R`：递归设置 ACL（对子目录和文件生效）。
- `-d`：设置默认 ACL，仅对目录有效。

---

#### 三、常用操作

##### 1. 查看 ACL
使用 `getfacl` 命令查看文件或目录的 ACL：
```bash
getfacl 文件或目录
```
示例：
```bash
getfacl example.txt
```
输出：
```
# file: example.txt
# owner: user1
# group: group1
user::rw-
group::r--
other::r--
```

##### 2. 为用户设置权限
为特定用户（如 `user2`）添加读写权限：
```bash
setfacl -m u:user2:rw example.txt
```
解释：
- `u:user2:rw` 表示为用户 `user2` 设置读写权限。

##### 3. 为组设置权限
为特定组（如 `group2`）添加读权限：
```bash
setfacl -m g:group2:r example.txt
```

##### 4. 删除 ACL 条目
移除用户 `user2` 的 ACL：
```bash
setfacl -x u:user2 example.txt
```

##### 5. 设置默认 ACL
默认 ACL 适用于目录，所有新创建的文件或子目录将继承该 ACL：
```bash
setfacl -d -m u:user2:rw example_dir
```

##### 6. 递归设置 ACL
对目录及其所有子文件和子目录设置 ACL：
```bash
setfacl -R -m u:user2:rw example_dir
```

---

#### 四、示例操作

假设有一个目录 `project`，需要实现以下权限：
1. 用户 `user1` 拥有所有权限。
2. 用户 `user2` 只能读取和写入文件。
3. 组 `developers` 只能读取文件。

##### 1. 创建示例目录和文件
```bash
mkdir project
touch project/file1 project/file2
```

##### 2. 设置 ACL
```bash
setfacl -m u:user1:rwx project/file1
setfacl -m u:user2:rw project/file1
setfacl -m g:developers:r project/file1
```

##### 3. 验证 ACL
```bash
getfacl project/file1
```
输出：
```
# file: project/file1
# owner: root
# group: root
user::rw-
user:user1:rwx
user:user2:rw-
group::r--
group:developers:r--
mask::rwx
other::---
```

---

#### 五、注意事项

1. **ACL 支持的文件系统**：
   - ACL 需要文件系统支持，例如 ext3、ext4、xfs 等。
   - 确保挂载分区时使用了 `acl` 参数，可以通过以下命令确认：
     ```bash
     mount | grep acl
     ```

2. **权限覆盖顺序**：
   - ACL 优先级高于传统的文件权限。
   - 如果用户在 ACL 中有特定权限，则传统权限不会生效。

3. **清理 ACL**：
   - 使用 `setfacl -b` 可以清除所有 ACL，恢复为传统权限模型。

---

#### 六、总结

`setfacl` 是管理文件和目录权限的强大工具，适用于多用户、多权限需求的场景。通过灵活使用 ACL，可以大大简化权限管理，同时满足复杂的访问控制需求。

