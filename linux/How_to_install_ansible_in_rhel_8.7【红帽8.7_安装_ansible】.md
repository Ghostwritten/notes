
![](https://i-blog.csdnimg.cn/blog_migrate/1c6cb91f6c1bb13392d3d8412ff22661.png)


```bash
sudo subscription-manager register
username:xxxx
password:xxxxx
sudo subscription-manager repos - enable ansible-2.9-for-rhel-8-x86_64-rpms
subscription-manager release - list
subscription-manager release - unset && subscription-manager release - set=8.7
sudo yum install ansible
```

> 注意：当遇到以下安装失败的问题

```bash
$ sudo yum -y install ansible
Updating Subscription Management repositories.
Red Hat Enterprise Linux 8 for x86_64 - AppStream (RPMs)                                                                                                            2.0 MB/s |  60 MB     00:30    
Errors during downloading metadata for repository 'rhel-8-for-x86_64-appstream-rpms':
  - Status code: 404 for https://cdn.redhat.com/content/dist/rhel8/8/x86_64/appstream/os/repodata/ab77593b3e45496bbf1cb280e7bc78a45b4fff2ae3fd1180fca4547d5a1d5239-comps.xml (IP: 173.223.228.251)
Error: Failed to download metadata for repo 'rhel-8-for-x86_64-appstream-rpms': Yum repo downloading error: Downloading error(s): repodata/ab77593b3e45496bbf1cb280e7bc78a45b4fff2ae3fd1180fca4547d5a1d5239-comps.xml - Cannot download, all mirrors were already tried without success
```
解决方法：

```bash
$ subscription-manager release --list
+-------------------------------------------+
          Available Releases
+-------------------------------------------+
8
8.0
8.1
8.2
8.3
8.4
8.5
8.6
8.7
8.8
8.9
$ hostnamectl 
   Static hostname: download
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 6040b755bb074dde82f2c7466e2f4bf0
           Boot ID: e2611826953c4fe49fa935b831286cb0
    Virtualization: vmware
  Operating System: Red Hat Enterprise Linux 8.7 (Ootpa)
       CPE OS Name: cpe:/o:redhat:enterprise_linux:8::baseos
            Kernel: Linux 4.18.0-425.3.1.el8.x86_64
      Architecture: x86-64
[root@localhost ~]# subscription-manager release --unset && subscription-manager release  --set=8.7
Release preference has been unset
Release set to: 8.7
$ sudo yum -y install ansible
Updating Subscription Management repositories.
Red Hat Enterprise Linux 8 for x86_64 - BaseOS (RPMs)                                                                                                               2.3 MB/s |  59 MB     00:25    
Red Hat Ansible Engine 2.9 for RHEL 8 x86_64 (RPMs)                                                                                                                 3.2 kB/s | 4.0 kB     00:01    
Red Hat Enterprise Linux 8 for x86_64 - AppStream (RPMs)                                                                                                            2.6 MB/s |  54 MB     00:20    
rhel 8.7 x64                                                                                                                                                        901 kB/s | 2.4 MB     00:02    
rhel 8.7 x64                                                                                                                                                        1.0 MB/s | 7.8 MB     00:07    
Dependencies resolved.
====================================================================================================================================================================================================
 Package                                         Architecture                      Version                                      Repository                                                     Size
====================================================================================================================================================================================================
Installing:
 ansible                                         noarch                            2.9.27-1.el8ae                               ansible-2.9-for-rhel-8-x86_64-rpms                             17 M
Installing dependencies:
 python3-babel                                   noarch                            2.5.1-7.el8                                  rhel-8-for-x86_64-appstream-rpms                              4.8 M
 python3-cffi                                    x86_64                            1.11.5-5.el8                                 rhel-8-for-x86_64-baseos-rpms                                 238 k
 python3-cryptography                            x86_64                            3.2.1-5.el8                                  rhel-8-for-x86_64-baseos-rpms                                 559 k
 python3-jinja2                                  noarch                            2.10.1-3.el8                                 rhel-8-for-x86_64-appstream-rpms                              538 k
 python3-markupsafe                              x86_64                            0.23-19.el8                                  rhel-8-for-x86_64-appstream-rpms                               39 k
 python3-ply                                     noarch                            3.9-9.el8                                    rhel-8-for-x86_64-baseos-rpms                                 111 k
 python3-pycparser                               noarch                            2.14-14.el8                                  rhel-8-for-x86_64-baseos-rpms                                 109 k
 python3-pytz                                    noarch                            2017.2-9.el8                                 rhel-8-for-x86_64-appstream-rpms                               54 k
 sshpass                                         x86_64                            1.09-4.el8                                   rhel-8-for-x86_64-appstream-rpms                               30 k
Installing weak dependencies:
 python3-jmespath                                noarch                            0.9.0-11.el8                                 rhel-8-for-x86_64-appstream-rpms                               45 k

Transaction Summary
====================================================================================================================================================================================================
Install  11 Packages

Total download size: 23 M
Installed size: 124 M
Downloading Packages:
(1/11): python3-ply-3.9-9.el8.noarch.rpm                                                                                                                             47 kB/s | 111 kB     00:02    
(2/11): python3-pycparser-2.14-14.el8.noarch.rpm                                                                                                                     45 kB/s | 109 kB     00:02    
(3/11): python3-cffi-1.11.5-5.el8.x86_64.rpm                                                                                                                        218 kB/s | 238 kB     00:01    
(4/11): python3-cryptography-3.2.1-5.el8.x86_64.rpm                                                                                                                 131 kB/s | 559 kB     00:04    
(5/11): sshpass-1.09-4.el8.x86_64.rpm                                                                                                                                29 kB/s |  30 kB     00:01    
(6/11): python3-pytz-2017.2-9.el8.noarch.rpm                                                                                                                         85 kB/s |  54 kB     00:00    
(7/11): python3-jmespath-0.9.0-11.el8.noarch.rpm                                                                                                                     43 kB/s |  45 kB     00:01    
(8/11): python3-jinja2-2.10.1-3.el8.noarch.rpm                                                                                                                      442 kB/s | 538 kB     00:01    
(9/11): python3-markupsafe-0.23-19.el8.x86_64.rpm                                                                                                                    27 kB/s |  39 kB     00:01    
(10/11): python3-babel-2.5.1-7.el8.noarch.rpm                                                                                                                       1.7 MB/s | 4.8 MB     00:02    
(11/11): ansible-2.9.27-1.el8ae.noarch.rpm                                                                                                                          852 kB/s |  17 MB     00:20    
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Total                                                                                                                                                               1.0 MB/s |  23 MB     00:22     
Red Hat Enterprise Linux 8 for x86_64 - BaseOS (RPMs)                                                                                                               4.9 MB/s | 5.0 kB     00:00    
Importing GPG key 0xFD431D51:
 Userid     : "Red Hat, Inc. (release key 2) <security@redhat.com>"
 Fingerprint: 567E 347A D004 4ADE 55BA 8A5F 199E 2F91 FD43 1D51
 From       : /etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release
Key imported successfully
Importing GPG key 0xD4082792:
 Userid     : "Red Hat, Inc. (auxiliary key) <security@redhat.com>"
 Fingerprint: 6A6A A7C9 7C88 90AE C6AE BFE2 F76F 66C3 D408 2792
 From       : /etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release
Key imported successfully
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                                                                                                                                            1/1 
  Installing       : python3-markupsafe-0.23-19.el8.x86_64                                                                                                                                     1/11 
  Installing       : python3-pytz-2017.2-9.el8.noarch                                                                                                                                          2/11 
  Installing       : python3-babel-2.5.1-7.el8.noarch                                                                                                                                          3/11 
  Installing       : python3-jinja2-2.10.1-3.el8.noarch                                                                                                                                        4/11 
  Installing       : python3-jmespath-0.9.0-11.el8.noarch                                                                                                                                      5/11 
  Installing       : sshpass-1.09-4.el8.x86_64                                                                                                                                                 6/11 
  Installing       : python3-ply-3.9-9.el8.noarch                                                                                                                                              7/11 
  Installing       : python3-pycparser-2.14-14.el8.noarch                                                                                                                                      8/11 
  Installing       : python3-cffi-1.11.5-5.el8.x86_64                                                                                                                                          9/11 
  Installing       : python3-cryptography-3.2.1-5.el8.x86_64                                                                                                                                  10/11 
  Installing       : ansible-2.9.27-1.el8ae.noarch                                                                                                                                            11/11 
  Running scriptlet: ansible-2.9.27-1.el8ae.noarch                                                                                                                                            11/11 
/sbin/ldconfig: File /lib64/libgpm.so.2 is empty, not checked.
/sbin/ldconfig: File /lib64/libgpm.so.2.1.0 is empty, not checked.
/sbin/ldconfig: File /lib64/libmetalink.so.3 is empty, not checked.
/sbin/ldconfig: File /lib64/libmetalink.so.3.1.0 is empty, not checked.

  Verifying        : python3-pycparser-2.14-14.el8.noarch                                                                                                                                      1/11 
  Verifying        : python3-ply-3.9-9.el8.noarch                                                                                                                                              2/11 
  Verifying        : python3-cryptography-3.2.1-5.el8.x86_64                                                                                                                                   3/11 
  Verifying        : python3-cffi-1.11.5-5.el8.x86_64                                                                                                                                          4/11 
  Verifying        : ansible-2.9.27-1.el8ae.noarch                                                                                                                                             5/11 
  Verifying        : sshpass-1.09-4.el8.x86_64                                                                                                                                                 6/11 
  Verifying        : python3-jmespath-0.9.0-11.el8.noarch                                                                                                                                      7/11 
  Verifying        : python3-pytz-2017.2-9.el8.noarch                                                                                                                                          8/11 
  Verifying        : python3-markupsafe-0.23-19.el8.x86_64                                                                                                                                     9/11 
  Verifying        : python3-jinja2-2.10.1-3.el8.noarch                                                                                                                                       10/11 
  Verifying        : python3-babel-2.5.1-7.el8.noarch                                                                                                                                         11/11 
Installed products updated.

Installed:
  ansible-2.9.27-1.el8ae.noarch          python3-babel-2.5.1-7.el8.noarch        python3-cffi-1.11.5-5.el8.x86_64   python3-cryptography-3.2.1-5.el8.x86_64   python3-jinja2-2.10.1-3.el8.noarch  
  python3-jmespath-0.9.0-11.el8.noarch   python3-markupsafe-0.23-19.el8.x86_64   python3-ply-3.9-9.el8.noarch       python3-pycparser-2.14-14.el8.noarch      python3-pytz-2017.2-9.el8.noarch    
  sshpass-1.09-4.el8.x86_64             

Complete!

$ ansible --version
ansible 2.9.27
  config file = /etc/ansible/ansible.cfg
  configured module search path = ['/root/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python3.6/site-packages/ansible
  executable location = /usr/bin/ansible
  python version = 3.6.8 (default, Jun 14 2022, 09:19:35) [GCC 8.5.0 20210514 (Red Hat 8.5.0-13)]

```

