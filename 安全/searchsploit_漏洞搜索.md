

---

## 1. 背景

在提权过程中需要通过掌握的信息来对系统、软件等存在的漏洞进行搜索，获取其利用的poc，通过编译后，实施提权。searchsploit提供漏洞本地和在线查询，是渗透测试中提权的重要武器。

## 2. 简介

[Exploit Database](https://github.com/offensive-security/exploit-database) 这是 [Offensive Security](https://www.offensive-security.com/) 赞助的一个项目。存储了大量的漏洞利用程序，可以帮助安全研究者和渗透测试工程师更好的进行安全测试工作，目前是世界上公开收集漏洞最全的数据库，该仓库每天都会更新，exploit-db提供searchsploit利用files.csv进行搜索离线漏洞库文件的位置。

## 3. 下载、安装、更新

- 直接[下载](https://codeload.github.com/offensive-security/exploit-database/zip/master)
- git clone [https://github.com/offensive-security/exploit-database.git](https://github.com/offensive-security/exploit-database.git)

安装

- centos 安装：yum install exploitdb
- MacOS安装：brew update && brew install exploitdb
- kali安装：apt update && apt -y install exploitdb

使用命令关联searchsploit：

```bash
ln -sf /opt/exploit-database/searchsploit   /usr/local/bin/searchsploit
```

更新

```bash
searchsploit –u
```

## 4. 语法

用法

```bash
searchsploit  [选线] term1 [term2] ... [termN]
```

选项：

- c, --case [Term] 执行区分大小写的搜索，缺省是对大小写不敏感。
- e, --exact [Term] 对exploit标题执行EXACT匹配（默认为AND）
- h, --help 在屏幕上显示帮助
- j, --json [Term] 以JSON格式显示结果
- m, --mirror [EDB-ID] 将一个漏洞利用镜像（副本）到当前工作目录，后面跟漏洞ID号
- o, --overflow [Term] Exploit标题被允许溢出其列
- p, --path [EDB-ID] 显示漏洞利用的完整路径（如果可能，还将路径复制到剪贴板），后面跟漏洞ID号
- t, --title [Term] 仅仅搜索漏洞标题（默认是标题和文件的路径）
- u, --update 检查并安装任何exploitdb软件包更新（deb或git）
- w, --www [Term] 显示Exploit-DB.com的URL而不是本地路径（在线搜索）
- x, --examine [EDB-ID] 使用$ PAGER检查（副本）漏洞利用
- -colour 在搜索结果中禁用颜色突出显示.
- -id 显示EDB-ID值而不是本地路径
- -nmap [file.xml] 使用服务版本检查Nmap XML输出中的所有结果（例如：nmap -sV -oX file.xml）。
- -exclude="term" 从结果中删除值。通过使用“|”分隔多个值，例如--exclude =“term1 | term2 | term3”。

## 5. 实例

### 5.1 帮助

```bash
$ ./searchsploit -h
  Usage: searchsploit [options] term1 [term2] ... [termN]

==========
 Examples 
==========
  searchsploit afd windows local
  searchsploit -t oracle windows
  searchsploit -p 39446
  searchsploit linux kernel 3.2 --exclude="(PoC)|/dos/"
  searchsploit -s Apache Struts 2.0.0
  searchsploit linux reverse password
  searchsploit -j 55555 | json_pp

  For more examples, see the manual: https://www.exploit-db.com/searchsploit

=========
 Options 
=========
## Search Terms
   -c, --case     [Term]      Perform a case-sensitive search (Default is inSEnsITiVe)
   -e, --exact    [Term]      Perform an EXACT & order match on exploit title (Default is an AND match on each term) [Implies "-t"]
                                e.g. "WordPress 4.1" would not be detect "WordPress Core 4.1")
   -s, --strict               Perform a strict search, so input values must exist, disabling fuzzy search for version range
                                e.g. "1.1" would not be detected in "1.0 < 1.3")
   -t, --title    [Term]      Search JUST the exploit title (Default is title AND the file's path)
       --exclude="term"       Remove values from results. By using "|" to separate, you can chain multiple values
                                e.g. --exclude="term1|term2|term3"

## Output
   -j, --json     [Term]      Show result in JSON format
   -o, --overflow [Term]      Exploit titles are allowed to overflow their columns
   -p, --path     [EDB-ID]    Show the full path to an exploit (and also copies the path to the clipboard if possible)
   -v, --verbose              Display more information in output
   -w, --www      [Term]      Show URLs to Exploit-DB.com rather than the local path
       --id                   Display the EDB-ID value rather than local path
       --colour               Disable colour highlighting in search results

## Non-Searching
   -m, --mirror   [EDB-ID]    Mirror (aka copies) an exploit to the current working directory
   -x, --examine  [EDB-ID]    Examine (aka opens) the exploit using $PAGER

## Non-Searching
   -h, --help                 Show this help screen
   -u, --update               Check for and install any exploitdb package updates (brew, deb & git)

## Automation
       --nmap     [file.xml]  Checks all results in Nmap's XML output with service version
                                e.g.: nmap [host] -sV -oX file.xml

=======
 Notes 
=======
 * You can use any number of search terms
 * By default, search terms are not case-sensitive, ordering is irrelevant, and will search between version ranges
   * Use '-c' if you wish to reduce results by case-sensitive searching
   * And/Or '-e' if you wish to filter results by using an exact match
   * And/Or '-s' if you wish to look for an exact version match
 * Use '-t' to exclude the file's path to filter the search results
   * Remove false positives (especially when searching using numbers - i.e. versions)
 * When using '--nmap', adding '-v' (verbose), it will search for even more combinations
 * When updating or displaying help, search terms will be ignored
```

### 5.2 搜索漏洞关键字afd的Windows本地利用漏洞

```bash
$ ./searchsploit afd windows local
[i] Found (#1): /root/exploitdb-master/files_exploits.csv
[i] To remove this message, please edit "/root/exploitdb-master/.searchsploit_rc" for "files_exploits.csv" (package_array: exploitdb)

[i] Found (#1): /root/exploitdb-master/files_shellcodes.csv
[i] To remove this message, please edit "/root/exploitdb-master/.searchsploit_rc" for "files_shellcodes.csv" (package_array: exploitdb)

---------------------------------------------------------------------- --------------------------------- Exploit Title                                                        |  Path
---------------------------------------------------------------------- ---------------------------------Microsoft Windows (x86) - 'afd.sys' Local Privilege Escalation (MS11- | windows_x86/local/40564.c
Microsoft Windows - 'afd.sys' Local Kernel (PoC) (MS11-046)           | windows/dos/18755.c
Microsoft Windows - 'AfdJoinLeaf' Local Privilege Escalation (MS11-08 | windows/local/21844.rb
Microsoft Windows 7 (x64) - 'afd.sys' Dangling Pointer Privilege Esca | windows_x86-64/local/39525.py
Microsoft Windows 7 (x86) - 'afd.sys' Dangling Pointer Privilege Esca | windows_x86/local/39446.py
Microsoft Windows XP - 'afd.sys' Local Kernel Denial of Service       | windows/dos/17133.c
Microsoft Windows XP/2003 - 'afd.sys' Local Privilege Escalation (K-p | windows/local/6757.txt
Microsoft Windows XP/2003 - 'afd.sys' Local Privilege Escalation (MS1 | windows/local/18176.py
---------------------------------------------------------------------- ---------------------------------Shellcodes: No Results
```

### 5.3 搜索标题中包含oracle windows的漏洞

```bash
$ ./searchsploit -t oracle windows
[i] Found (#1): /root/exploitdb-master/files_exploits.csv
[i] To remove this message, please edit "/root/exploitdb-master/.searchsploit_rc" for "files_exploits.csv" (package_array: exploitdb)

[i] Found (#1): /root/exploitdb-master/files_shellcodes.csv
[i] To remove this message, please edit "/root/exploitdb-master/.searchsploit_rc" for "files_shellcodes.csv" (package_array: exploitdb)

---------------------------------------------------------------------- --------------------------------- Exploit Title                                                        |  Path
---------------------------------------------------------------------- ---------------------------------
Oracle 10g (Windows x86) - 'PROCESS_DUP_HANDLE' Local Privilege Escal | windows_x86/local/3451.c
Oracle 9i XDB (Windows x86) - FTP PASS Overflow (Metasploit)          | windows_x86/remote/16731.rb
Oracle 9i XDB (Windows x86) - FTP UNLOCK Overflow (Metasploit)        | windows_x86/remote/16714.rb
Oracle 9i XDB (Windows x86) - HTTP PASS Overflow (Metasploit)         | windows_x86/remote/16809.rb
Oracle MySQL (Windows) - FILE Privilege Abuse (Metasploit)            | windows/remote/35777.rb
Oracle MySQL (Windows) - MOF Execution (Metasploit)                   | windows/remote/23179.rb
Oracle MySQL for Microsoft Windows - Payload Execution (Metasploit)   | windows/remote/16957.rb
Oracle VirtualBox Guest Additions 5.1.18 - Unprivileged Windows User- | multiple/dos/41932.cpp
Oracle VM VirtualBox 5.0.32 r112930 (x64) - Windows Process COM Injec | windows_x86-64/local/41908.txt
---------------------------------------------------------------------- ---------------------------------Shellcodes: No Results
```

### 5.4 搜索漏洞号为39446的漏洞

```bash
$ ./searchsploit -p 39446
[i] Found (#1): /root/exploitdb-master/files_exploits.csv
[i] To remove this message, please edit "/root/exploitdb-master/.searchsploit_rc" for "files_exploits.csv" (package_array: exploitdb)

[i] Found (#1): /root/exploitdb-master/files_shellcodes.csv
[i] To remove this message, please edit "/root/exploitdb-master/.searchsploit_rc" for "files_shellcodes.csv" (package_array: exploitdb)

  Exploit: Microsoft Windows 7 (x86) - 'afd.sys' Dangling Pointer Privilege Escalation (MS14-040)
      URL: https://www.exploit-db.com/exploits/39446
     Path: /root/exploitdb-master/exploits/windows_x86/local/39446.py
File Type: Python script, ASCII text executable
```

### 5.5 排除dos以及PoC值的包含linux kernel 3.2的漏洞

```bash
$ ./searchsploit linux kernel 3.2 --exclude="(PoC)|/dos/"
[i] Found (#1): /root/exploitdb-master/files_exploits.csv
[i] To remove this message, please edit "/root/exploitdb-master/.searchsploit_rc" for "files_exploits.csv" (package_array: exploitdb)

[i] Found (#1): /root/exploitdb-master/files_shellcodes.csv
[i] To remove this message, please edit "/root/exploitdb-master/.searchsploit_rc" for "files_shellcodes.csv" (package_array: exploitdb)

---------------------------------------------------------------------- --------------------------------- Exploit Title                                                        |  Path
---------------------------------------------------------------------- ---------------------------------
Linux Kernel (Solaris 10 / < 5.10 138888-01) - Local Privilege Escala | solaris/local/15962.c
Linux Kernel 2.6.19 < 5.9 - 'Netfilter Local Privilege Escalation     | linux/local/50135.c
Linux Kernel 2.6.22 < 3.9 (x86/x64) - 'Dirty COW /proc/self/mem' Race | linux/local/40616.c
Linux Kernel 2.6.22 < 3.9 - 'Dirty COW /proc/self/mem' Race Condition | linux/local/40847.cpp
Linux Kernel 2.6.22 < 3.9 - 'Dirty COW PTRACE_POKEDATA' Race Conditio | linux/local/40838.c
Linux Kernel 2.6.22 < 3.9 - 'Dirty COW' 'PTRACE_POKEDATA' Race Condit | linux/local/40839.c
Linux Kernel 2.6.22 < 3.9 - 'Dirty COW' /proc/self/mem Race Condition | linux/local/40611.c
Linux Kernel 2.6.39 < 3.2.2 (Gentoo / Ubuntu x86/x64) - 'Mempodipper' | linux/local/18411.c
Linux Kernel 2.6.39 < 3.2.2 (x86/x64) - 'Mempodipper' Local Privilege | linux/local/35161.c
Linux Kernel 3.0 < 3.3.5 - 'CLONE_NEWUSER|CLONE_FS' Local Privilege E | linux/local/38390.c
Linux Kernel 3.14-rc1 < 3.15-rc4 (x64) - Raw Mode PTY Echo Race Condi | linux_x86-64/local/33516.c
Linux Kernel 3.2.0-23/3.5.0-23 (Ubuntu 12.04/12.04.1/12.04.2 x64) - ' | linux_x86-64/local/33589.c
Linux Kernel 3.2.x - 'uname()' System Call Local Information Disclosu | linux/local/37937.c
Linux Kernel 3.4 < 3.13.2 (Ubuntu 13.04/13.10 x64) - 'CONFIG_X86_X32= | linux_x86-64/local/31347.c
Linux Kernel 3.4 < 3.13.2 (Ubuntu 13.10) - 'CONFIG_X86_X32' Arbitrary | linux/local/31346.c
Linux Kernel 4.8.0 UDEV < 232 - Local Privilege Escalation            | linux/local/41886.c
Linux Kernel < 3.16.1 - 'Remount FUSE' Local Privilege Escalation     | linux/local/34923.c
Linux Kernel < 3.16.39 (Debian 8 x64) - 'inotfiy' Local Privilege Esc | linux_x86-64/local/44302.c
Linux Kernel < 3.2.0-23 (Ubuntu 12.04 x64) - 'ptrace/sysret' Local Pr | linux_x86-64/local/34134.c
Linux Kernel < 3.4.5 (Android 4.2.2/4.4 ARM) - Local Privilege Escala | arm/local/31574.c
Linux Kernel < 3.5.0-23 (Ubuntu 12.04.2 x64) - 'SOCK_DIAG' SMEP Bypas | linux_x86-64/local/44299.c
Linux Kernel < 3.8.9 (x86-64) - 'perf_swevent_init' Local Privilege E | linux_x86-64/local/26131.c
Linux Kernel < 3.8.x - open-time Capability 'file_ns_capable()' Local | linux/local/25450.c
Linux kernel < 4.10.15 - Race Condition Privilege Escalation          | linux/local/43345.c
Linux Kernel < 4.11.8 - 'mq_notify: double sock_put()' Local Privileg | linux/local/45553.c
Linux Kernel < 4.13.9 (Ubuntu 16.04 / Fedora 27) - Local Privilege Es | linux/local/45010.c
Linux Kernel < 4.15.4 - 'show_floppy' KASLR Address Leak              | linux/local/44325.c
Linux Kernel < 4.4.0-116 (Ubuntu 16.04.4) - Local Privilege Escalatio | linux/local/44298.c
Linux Kernel < 4.4.0-21 (Ubuntu 16.04 x64) - 'netfilter target_offset | linux_x86-64/local/44300.c
Linux Kernel < 4.4.0-83 / < 4.8.0-58 (Ubuntu 14.04/16.04) - Local Pri | linux/local/43418.c
Linux Kernel < 4.4.0/ < 4.8.0 (Ubuntu 14.04/16.04 / Linux Mint 17/18  | linux/local/47169.c
---------------------------------------------------------------------- ---------------------------------Shellcodes: No Results
```

### 5.6 查找mssql的漏洞

```bash
$ ./searchsploit mssql
[i] Found (#1): /root/exploitdb-master/files_exploits.csv
[i] To remove this message, please edit "/root/exploitdb-master/.searchsploit_rc" for "files_exploits.csv" (package_array: exploitdb)

[i] Found (#1): /root/exploitdb-master/files_shellcodes.csv
[i] To remove this message, please edit "/root/exploitdb-master/.searchsploit_rc" for "files_shellcodes.csv" (package_array: exploitdb)

---------------------------------------------------------------------- --------------------------------- Exploit Title                                                        |  Path
---------------------------------------------------------------------- ---------------------------------ADODB 4.6/4.7 - 'Tmssql.php' Cross-Site Scripting                     | php/webapps/28104.txt
ADODB < 4.70 - 'tmssql.php' Denial of Service                         | php/dos/1651.php
AutoDealer 1.0/2.0 - MSSQL Injection                                  | php/webapps/12462.txt
MSSQL 7.0 - Remote Denial of Service                                  | windows/dos/562.c
PHP 4.4.6 - 'mssql_[p]connect()' Local Buffer Overflow                | windows/local/3417.php
XAMPP for Windows 1.6.0a - 'mssql_connect()' Remote Buffer Overflow   | windows/remote/3738.php
---------------------------------------------------------------------- ---------------------------------Shellcodes: No Results
```

### 5.7 查找和window XP有关的漏洞

```bash
$ ./searchsploit /xp
[i] Found (#2): ./files_exploits.csv
[i] To remove this message, please edit "./.searchsploit_rc" for "files_exploits.csv" (package_array: exploitdb)

[i] Found (#2): ./files_shellcodes.csv
[i] To remove this message, please edit "./.searchsploit_rc" for "files_shellcodes.csv" (package_array: exploitdb)

---------------------------------------------------------------------- --------------------------------- Exploit Title                                                        |  Path
---------------------------------------------------------------------- ---------------------------------Apple QuickTime 7.2/7.3 (Windows Vista/XP) - RSTP Response Code Execu | windows/remote/4651.cpp
Microsoft Office 2000/2003/2004/XP - File Memory Corruption           | windows/dos/31361.txt
Microsoft Windows 2000/XP - SMB Authentication Remote Overflow        | windows/remote/20.txt
Microsoft Windows 98/XP/ME - UPnP NOTIFY Buffer Overflow (1)          | windows/remote/21188.c
Microsoft Windows 98/XP/ME - UPnP NOTIFY Buffer Overflow (2)          | windows/remote/21189.c
Microsoft Windows NT/2000/2003/2008/XP/Vista/7 - 'KiTrap0D' User Mode | windows/local/11199.txt
Microsoft Windows NT/2000/2003/2008/XP/Vista/7/8 - 'EPATHOBJ' Local R | windows/local/25912.c
Mozilla Firefox 1.5.0.2 - 'js320.dll/xpcom_core.dll' Denial of Servic | multiple/dos/1716.html
Novell Client for Windows 2000/XP - ActiveX Remote Denial of Service  | windows/dos/9516.txt
PSOProxy 0.91 (Windows 2000/XP) - Remote Buffer Overflow              | windows/remote/156.c
---------------------------------------------------------------------- ------------------------------------------------------------------------------------------------------- --------------------------------- Shellcode Title                                                      |  Path
---------------------------------------------------------------------- ---------------------------------Windows (2000/XP/7) - URLDownloadToFile(http://bflow.security-portal. | windows/24318.c
Windows (9x/NT/2000/XP) - PEB Method Shellcode (29 bytes)             | windows_x86/13525.c
Windows (9x/NT/2000/XP) - PEB Method Shellcode (31 bytes)             | windows_x86/13526.c
Windows (9x/NT/2000/XP) - PEB Method Shellcode (35 bytes)             | windows_x86/13527.c
Windows (9x/NT/2000/XP) - Reverse Generic Without Loader (192.168.1.1 | windows_x86/13524.txt
Windows (NT/2000/XP) (Russian) - Add Administartor User (slim/shady)  | windows_x86/13523.c
Windows/x86 (NT/XP) - IsDebuggerPresent Shellcode (39 bytes)          | windows_x86/13518.c
Windows/x86 (NT/XP/2000/2003) - Bind (8721/TCP) Shell Shellcode (356  | windows_x86/43759.asm
---------------------------------------------------------------------- ---------------------------------
```

### 5.8 查找apple的漏洞

```bash
$ ./searchsploit apple
[i] Found (#2): ./files_exploits.csv
[i] To remove this message, please edit "./.searchsploit_rc" for "files_exploits.csv" (package_array: exploitdb)

[i] Found (#2): ./files_shellcodes.csv
[i] To remove this message, please edit "./.searchsploit_rc" for "files_shellcodes.csv" (package_array: exploitdb)

---------------------------------------------------------------------- --------------------------------- Exploit Title                                                        |  Path
---------------------------------------------------------------------- ---------------------------------
Apple 2.0.4 - Safari Local Cross-Site Scripting                       | osx/local/29950.js
Apple Airport - 802.11 Probe Response Kernel Memory Corruption (PoC)  | hardware/dos/2700.rb
Apple At Ease 5.0 - Information Disclosure                            | osx/local/19427.txt
Apple Bonjour for Windows 1.0.4 - mDNSResponder Null Pointer Derefere | windows/dos/32350.txt
Apple CFNetwork - HTTP Response Denial of Service                     | osx/dos/3200.rb
Apple Directory Services - Memory Corruption                          | osx/dos/15491.txt
..................
```

## 5.9 技巧

- 1.查询关键字采取AND运算,SearchSploit使用AND运算符，而不是OR运算符。使用的术语越多，滤除的结果越多。
- 2.使用名称搜索时尽量使用全称
- 3.使用“-t”选项，默认情况下，searchsploit将检查该漏洞利用的标题以及该路径。根据搜索条件，这可能会导致误报（特别是在搜索与平台和版本号匹配的术语时），使用“-t”选项去掉多余数据。例如`searchsploit -t oracle windows`显示7行数据而`searchsploit   oracle windows |wc –l`显示90行数据。
- 4.在线搜索exploit-db.com中的关键字漏洞：`searchsploit  WarFTP 1.65 -w`
- 5.搜索微软漏洞，搜索微软2014年的所有漏洞，关键字可以ms14，ms15，ms16，ms17，`searchsploit MS14`

- ****[How to Use Searchsploit in Kali Linux?](https://bughacking.com/how-to-use-searchsploit-in-kali-linux/)****
- **[SearchSploit – The Manual](https://www.exploit-db.com/searchsploit)**
- **[Finding Exploit offline using Searchsploit in Kali Linux](https://www.geeksforgeeks.org/finding-exploit-offline-using-searchsploit-in-kali-linux/)**

![在这里插入图片描述](https://img-blog.csdnimg.cn/ba75c97188ab48f0a0b8c1101b1e2508.gif#pic_center)


