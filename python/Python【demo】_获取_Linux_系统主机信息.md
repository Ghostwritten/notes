

 - 该脚本适用于`centos`、`redhat`
 - 该脚本适用于在多网卡的条件下准确获取ip地址信息，比如在存在：lo、eth0、ens99、br、docker0
 - 该脚本仅适用于获取本机主机信息，无法远程批量。

使用 python 代码收集主机的系统信息，主要：

 - 主机名称
 - IP
 - 系统版本
 - 服务器厂商
 - 型号
 - 序列号
 - CPU 信息
 - 内存等系统信息。

```python
#!/usr/bin/env python
#encoding: utf-8

'''
收集主机的信息：
主机名称、IP、系统版本、服务器厂商、型号、序列号、CPU信息、内存信息
'''

from subprocess import Popen, PIPE
import os,sys

''' 获取 ifconfig 命令的输出 '''
def getIfconfig():
    p = Popen(['ifconfig'], stdout = PIPE)
    data = p.stdout.read()

    return data

''' 获取 dmidecode 命令的输出 '''
def getDmi():
    p = Popen(['dmidecode'], stdout = PIPE)
    data = p.stdout.read()
    return data

''' 根据空行分段落 返回段落列表'''
def parseData(data):
    parsed_data = []
    new_line = ''
    data = [i for i in data.split('\n') if i]
    for line in data:
        if line[0].strip():
            parsed_data.append(new_line)
            new_line = line + '\n'
        else:
            new_line += line + '\n'
    return [i for i in parsed_data if i]

''' 根据输入的段落数据分析出ifconfig的每个网卡ip信息 '''
def parseIfconfig(parsed_data):
    dic = {}
    parsed_data = [i for i in parsed_data if i.startswith('e')]
    for lines in parsed_data:
        line_list = lines.split('\n')
        devname = line_list[0].split()[0]
        macaddr = line_list[0].split()[-1]
        ipaddr  = line_list[1].split()[1].split(':')[0]
       
        break
    dic['ip'] = ipaddr
    return dic

''' 根据输入的dmi段落数据 分析出指定参数 '''
def parseDmi(parsed_data):
    dic = {}
    parsed_data = [i for i in parsed_data if i.startswith('System Information')]
    parsed_data = [i for i in parsed_data[0].split('\n')[1:] if i]
    dmi_dic = dict([i.strip().split(':') for i in parsed_data])
    dic['vender'] = dmi_dic['Manufacturer'].strip()
    dic['product'] = dmi_dic['Product Name'].strip()
    dic['sn'] = dmi_dic['Serial Number'].strip()
    return dic

''' 获取Linux系统主机名称 '''
def getHostname():
    import socket
    hostname = socket.gethostname()
#    with open('/etc/sysconfig/network') as fd:
#        for line in fd:
#            if line.startswith('HOSTNAME'):
#                hostname = line.split('=')[1].strip()
#                break
#    print hostname
    return {'hostname':hostname}

''' 获取Linux系统的版本信息 '''
def getOsVersion():
    with open('/etc/issue') as fd:
        for line in fd:
            osver = line.strip()
            break
    return {'osver':osver}

''' 获取CPU的型号和CPU的核心数 '''
def getCpu():
    num = 0
    with open('/proc/cpuinfo') as fd:
        for line in fd:
            if line.startswith('processor'):
                num += 1
            if line.startswith('model name'):
                cpu_model = line.split(':')[1].strip().split()
                cpu_model = cpu_model[0] + ' ' + cpu_model[2]  + ' ' + cpu_model[-1]
    return {'cpu_num':num, 'cpu_model':cpu_model}

''' 获取Linux系统的总物理内存 '''
def getMemory():
    with open('/proc/meminfo') as fd:
        for line in fd:
            if line.startswith('MemTotal'):
                mem = int(line.split()[1].strip())
                break
    mem = '%.f' % (mem / 1024.0) + ' MB'
    return {'Memory':mem}

if __name__ == '__main__':
    dic = {}
    data_ip = getIfconfig()
    parsed_data_ip = parseData(data_ip)
    ip = parseIfconfig(parsed_data_ip)
    
    data_dmi = getDmi()
    parsed_data_dmi = parseData(data_dmi)
    dmi = parseDmi(parsed_data_dmi)

    hostname = getHostname()
    osver = getOsVersion()
    cpu = getCpu()
    mem = getMemory()
    
    dic.update(ip)
    dic.update(dmi)
    dic.update(hostname)
    dic.update(osver)
    dic.update(cpu)
    dic.update(mem)

    ''' 将获取到的所有数据信息并按简单格式对齐显示 '''
    for k,v in dic.items():
        print '%-10s:%s' % (k, v)
```
执行:

```perl
$ python host_info.py
product   :VMware Virtual Platform
osver     :\S
sn        :VMware-56 4d b1 93 b5 b5 d4 72-5f b2 e6 1d ea cd 41 a0
Memory    :7803 MB
cpu_num   :4
ip        :192.168.211.100
vender    :VMware, Inc.
hostname  :ghostwritten.com
cpu_model :Intel(R) i7-6600U 2.60GHz
```
✈<font color=	#FF4500 size=4 style="font-family:Courier New">推荐阅读：</font>

 - [python subprocess 模块执行shell](https://blog.csdn.net/xixihahalelehehe/article/details/106762051?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522165055868916780366596245%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fblog.%2522%257D&request_id=165055868916780366596245&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_ecpm_v1~rank_v31_ecpm-1-106762051.nonecase&utm_term=python%20subprocess&spm=1018.2226.3001.4450)
 - [python psutil 模块获取主机更多细节](https://blog.csdn.net/xixihahalelehehe/article/details/105203250?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522165055868916780366596245%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fblog.%2522%257D&request_id=165055868916780366596245&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_ecpm_v1~rank_v31_ecpm-3-105203250.nonecase&utm_term=python%20subprocess&spm=1018.2226.3001.4450)
 - [python os模块文件目录操作](https://blog.csdn.net/xixihahalelehehe/article/details/104253123)
 - [python sys模块](https://blog.csdn.net/xixihahalelehehe/article/details/106693564?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522165060724416782248546311%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fblog.%2522%257D&request_id=165060724416782248546311&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_ecpm_v1~rank_v31_ecpm-1-106693564.nonecase&utm_term=sys&spm=1018.2226.3001.4450)

![在这里插入图片描述](https://img-blog.csdnimg.cn/2e8a51e89bca4c9bb1e662d13e32c5a4.gif#pic_center)

