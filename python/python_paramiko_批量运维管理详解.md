#  python 模块 paramiko批量远程



-----
## 1. 简介
[paramiko](https://www.paramiko.org/) 模块是基于python实现了SSH2远程安全连接，支持认证和密钥方式，可以实现远程连接、命令执行、文件传输、中间SSH代理功能，相对于[pexpect](https://www.cnblogs.com/baishuchao/p/9339159.html)，封装层次更高。


## 2. 安装

```python
pip install paramiko
easy_install paramiko
paramiko依赖第三方的Crypto，Ecdsa和pyhton-devel,所以需要安装
```
## 3. paramiko核心组件
### 3.1 SSHClient类
SSHClient类是SSH服务会话的高级表示，该类实现了传输、通道、以及SFTP的校验、建立的方法

#### 3.1.1 connect 方法

connect方法实现了远程ssh连接并作校验

参数

```bash
hostname 连接的目标主机
port=SSH_PORT 指定端口
username=None 验证的用户名
password=None 验证的用户密码
pkey=None 私钥方式用于身份验证
key_filename=None 一个文件名或文件列表，指定私钥文件
timeout=None 可选的tcp连接超时时间
allow_agent=True, 是否允许连接到ssh代理，默认为True 允许
look_for_keys=True 是否在~/.ssh中搜索私钥文件，默认为True 允许
compress=False, 是否打开压缩
sock=None,
gss_auth=False,
gss_kex=False,
gss_deleg_creds=True,
gss_host=None,
banner_timeout=None
```

#### 3.1.2 exec_command方法

远程执行命令的方法，该命令的输入与输出流为标准输入、标出输出、标准错误输出

参数

```bash
command 执行的命令
bufsize=-1 文件缓冲区大小
timeout=None
get_pty=False
```


#### 3.1.3 load_system_host_key方法

夹在本地公钥文件，默认为`~/.ssh/known_hosts`

参数

```bash
filename=None 指定本地公钥文件
```

#### 3.1.4 set_missing_host_key_policy方法
设置连接的远程主机没有本地主机密钥或HostKeys对象时的策略，目前支持三种：

 - `AutoAddPolicy`
 自动添加主机名及主机密钥到本地HostKeys对象，不依赖load_system_host_key的配置。即新建立ssh连接时不需要再输入yes或no进行确认
 - `WarningPolicy`
   用于记录一个未知的主机密钥的python警告。并接受，功能上和AutoAddPolicy类似，但是会提示是新连接
 - `RejectPolicy` 
自动拒绝未知的主机名和密钥，依赖load_system_host_key的配置。此为默认选项

用法：

```bash
ssh.paramiko.SSHClient()
set_missing_host_key_policy(paramiko.AutoAddPolicy())
```
### 3.2 SFTPClient类
SFTPCLient作为一个sftp的客户端对象，根据ssh传输协议的sftp会话，实现远程文件操作，如上传、下载、权限、状态

#### 3.2.1 from_transport
  创建一个已连通的SFTP客户端通道。
  定义：

```python
  from_transport(cls,t)
```

  示例：

```python
import paramiko

t = paramiko.Transport(('192.168.56.132',22))
t.connect(username='root',password='1234567')
sftp = paramiko.SFTPClient.from_transport(t)
```
#### 3.2.2 put
将本地文件上传到服务器，参数confirm：是否调用stat()方法检查文件状态，返回ls -l的结果。
定义：

```python
put(localpath, remotepath, callback=None, confirm=True) 
```
参数说明：

 - localpath(str类型)，需上传的本地文件(源)；
 - remotepath(str类型),远程路径(目标)；
 - callback(funcation(int,int)),获取已接收的字节数及总传输字节数，以便回调函数调用，默认为None;
 - confirm(bool类型)，文件上传完毕后是否调用stat()方法，以便确认文件的大小。

示例：

```python
localpath='/home/access.log'
remotepath='/data/logs/access.log'
sftp.put(localpath,remotepath)
```
#### 3.2.3 get
从服务器下载文件到本地
定义：

```python
get(remotepath, localpath, callback=None)
```
参数说明：

 - remotepath(str类型)，需要下载的远程文件(源)；
 - callback(funcation(int,int)),获取已接收的字节数及总和传输字节数，以便回调函数调用，默认为None.
 示例说明：

```python
remotepath = '/data/logs/access.log'
localpath = '/home/access.log'
sftp.get(remotepath,localpath)
```
#### 3.2.4 其他
 - remove() 在服务器上删除目录
 - rename() 在服务器上重命名目录
 - stat() 查看服务器文件状态
 - listdir() 列出服务器目录下的文件

### 3.3 远程连接并执行命令
实现远程连接主机，并执行命令，同时记录日志
####  3.3.1 直接验证方式

```python
import paramiko

host = '172.16.200.45'
port = 22
user = 'root'
passwd = '123123'
# 创建SSH对象
ssh = paramiko.SSHClient()
# 允许连接不在know_hosts文件中的主机
ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())
# 连接服务器
ssh.connect(hostname=host, port=port, username=user, password=passwd)
paramiko.util.log_to_file('syslogin.log')  #将登录信息记录日志
 # 执行命令
stdin, stdout, stderr = ssh.exec_command('df')
# 获取命令结果
result = stdout.read()
print(result)
# 关闭连接
ssh.close()
```
#### 3.3.2 SSHClient 封装 Transport

```python
import paramiko


host = '172.16.200.45'
port = 22
user = 'root'
passwd = '123123'

transport = paramiko.Transport((host, port))
transport.connect(username=user, password=passwd)

ssh = paramiko.SSHClient()
ssh._transport = transport

stdin, stdout, stderr = ssh.exec_command('df')
print(stdout.read())
```
#### 3.3.3 基于公钥密钥连接

```python
import paramiko
private_key = paramiko.RSAKey.from_private_key_file('/home/fuzengjie/.ssh/id_rsa')
 
# 创建SSH对象
ssh = paramiko.SSHClient()
# 允许连接不在know_hosts文件中的主机
ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())
# 连接服务器
ssh.connect(hostname='172.16.200.45', port=22, username='fuzengjie', key=private_key)
 
# 执行命令
stdin, stdout, stderr = ssh.exec_command('df')
# 获取命令结果
result = stdout.read()
 
# 关闭连接
ssh.close()
```
#### 3.3.4 SSHClient 封装 Transport 使用公钥方式

```python
import paramiko

private_key = paramiko.RSAKey.from_private_key_file('/home/fuzengjie/.ssh/id_rsa')

transport = paramiko.Transport(('172.16.200.45', 22))
transport.connect(username='fuzengjie', pkey=private_key)

ssh = paramiko.SSHClient()
ssh._transport = transport

stdin, stdout, stderr = ssh.exec_command('df')

transport.close()
```
### 3.4 远程连接实现文件上传下载
#### 3.4.1 基于用户名密码上传下载

```python
import paramiko


host = '172.16.200.45'
port = 22
user = 'root'
passwd = '123123'
transport = paramiko.Transport((host,port))
transport.connect(username=user,password=passwd)

sftp = paramiko.SFTPClient.from_transport(transport)
# 将location.py 上传至服务器 /tmp/test.py
a = sftp.put('/Users/fuzengjie/1', '/tmp/fuzj123',confirm=True)
print(a)  #打印上传到服务器上的文件状态
# 将remove_path 下载到本地 local_path
sftp.get('/root/tesst.py', '/Users/fuzengjie/test.py')

transport.close()
```
#### 3.4.2 基于公钥密钥上传下载

```python
import paramiko
 
private_key = paramiko.RSAKey.from_private_key_file('/home/fuzengjie/.ssh/id_rsa')
 
transport = paramiko.Transport(('172.16.200.45', 22))
transport.connect(username='fuzengjie', pkey=private_key )
 
sftp = paramiko.SFTPClient.from_transport(transport)
# 将location.py 上传至服务器 /tmp/test.py
sftp.put('/tmp/location.py', '/tmp/test.py')
# 将remove_path 下载到本地 local_path
sftp.get('/root/123.txt', '/tmp/123')
 
transport.close()
```
参考：

 - [python远程连接paramiko 模块和堡垒机实现 ](https://www.cnblogs.com/pycode/p/paramiko.html)
 - [Module paramiko](https://pyneng.readthedocs.io/en/latest/book/18_ssh_telnet/paramiko.html)
 - [Paramiko- How to SSH and transfer files with python](https://medium.com/@keagileageek/paramiko-how-to-ssh-and-file-transfers-with-python-75766179de73)

