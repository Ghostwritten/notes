```bash
root@node2:~# apt-get update
Hit:1 http://mirrors.ustc.edu.cn/ubuntu xenial InRelease
Hit:2 http://mirrors.ustc.edu.cn/ubuntu xenial-security InRelease                                                                                        
Hit:3 http://mirrors.ustc.edu.cn/ubuntu xenial-updates InRelease                                                                                         
Get:4 https://mirrors.aliyun.com/kubernetes/apt kubernetes-xenial InRelease [9,383 B]                                     
Hit:5 http://mirrors.ustc.edu.cn/ubuntu xenial-proposed InRelease                                                                      
Hit:6 http://mirrors.ustc.edu.cn/ubuntu xenial-backports InRelease                                  
Ign:7 https://download.falco.org/packages/deb stable InRelease                                      
Hit:8 https://download.falco.org/packages/deb stable Release
Err:4 https://mirrors.aliyun.com/kubernetes/apt kubernetes-xenial InRelease
  The following signatures couldn't be verified because the public key is not available: NO_PUBKEY FEEA9169307EA071 NO_PUBKEY 8B57C5C2836F4BEB
Reading package lists... Done
W: An error occurred during the signature verification. The repository is not updated and the previous index files will be used. GPG error: https://mirrors.aliyun.com/kubernetes/apt kubernetes-xenial InRelease: The following signatures couldn't be verified because the public key is not available: NO_PUBKEY FEEA9169307EA071 NO_PUBKEY 8B57C5C2836F4BEB
W: Target Packages (main/binary-amd64/Packages) is configured multiple times in /etc/apt/sources.list.d/falcosecurity.list:1 and /etc/apt/sources.list.d/falcosecurity.list:2
W: Target Packages (main/binary-i386/Packages) is configured multiple times in /etc/apt/sources.list.d/falcosecurity.list:1 and /etc/apt/sources.list.d/falcosecurity.list:2
W: Target Packages (main/binary-all/Packages) is configured multiple times in /etc/apt/sources.list.d/falcosecurity.list:1 and /etc/apt/sources.list.d/falcosecurity.list:2
W: Target Translations (main/i18n/Translation-en_US) is configured multiple times in /etc/apt/sources.list.d/falcosecurity.list:1 and /etc/apt/sources.list.d/falcosecurity.list:2
W: Target Translations (main/i18n/Translation-en) is configured multiple times in /etc/apt/sources.list.d/falcosecurity.list:1 and /etc/apt/sources.list.d/falcosecurity.list:2
N: Skipping acquire of configured file 'main/binary-i386/Packages' as repository 'https://download.falco.org/packages/deb stable InRelease' doesn't support architecture 'i386'
W: Failed to fetch https://mirrors.aliyun.com/kubernetes/apt/dists/kubernetes-xenial/InRelease  The following signatures couldn't be verified because the public key is not available: NO_PUBKEY FEEA9169307EA071 NO_PUBKEY 8B57C5C2836F4BEB
W: Some index files failed to download. They have been ignored, or old ones used instead.
W: Target Packages (main/binary-amd64/Packages) is configured multiple times in /etc/apt/sources.list.d/falcosecurity.list:1 and /etc/apt/sources.list.d/falcosecurity.list:2
W: Target Packages (main/binary-i386/Packages) is configured multiple times in /etc/apt/sources.list.d/falcosecurity.list:1 and /etc/apt/sources.list.d/falcosecurity.list:2
W: Target Packages (main/binary-all/Packages) is configured multiple times in /etc/apt/sources.list.d/falcosecurity.list:1 and /etc/apt/sources.list.d/falcosecurity.list:2
W: Target Translations (main/i18n/Translation-en_US) is configured multiple times in /etc/apt/sources.list.d/falcosecurity.list:1 and /etc/apt/sources.list.d/falcosecurity.list:2
W: Target Translations (main/i18n/Translation-en) is configured multiple times in /etc/apt/sources.list.d/falcosecurity.list:1 and /etc/apt/sources.list.d/falcosecurity.list:2



root@node2:~# sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys FEEA9169307EA071
Executing: /tmp/apt-key-gpghome.CG5K5gybKe/gpg.1.sh --keyserver keyserver.ubuntu.com --recv-keys FEEA9169307EA071
gpg: key FEEA9169307EA071: public key "Rapture Automatic Signing Key (cloud-rapture-signing-key-2021-03-01-08_01_09.pub)" imported
gpg: Total number processed: 1
gpg:               imported: 1




root@node2:~# sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 8B57C5C2836F4BEB
Executing: /tmp/apt-key-gpghome.4OEb4pQUxr/gpg.1.sh --keyserver keyserver.ubuntu.com --recv-keys 8B57C5C2836F4BEB
gpg: key 8B57C5C2836F4BEB: public key "gLinux Rapture Automatic Signing Key (//depot/google3/production/borg/cloud-rapture/keys/cloud-rapture-pubkeys/cloud-rapture-signing-key-2020-12-03-16_08_05.pub) <glinux-team@google.com>" imported
gpg: Total number processed: 1
gpg:               imported: 1



root@node2:~# apt-get update
Hit:1 http://mirrors.ustc.edu.cn/ubuntu xenial InRelease
Hit:2 http://mirrors.ustc.edu.cn/ubuntu xenial-security InRelease                                                                                       
Hit:3 http://mirrors.ustc.edu.cn/ubuntu xenial-updates InRelease                                                                                        
Get:4 https://mirrors.aliyun.com/kubernetes/apt kubernetes-xenial InRelease [9,383 B]                                             
Hit:5 http://mirrors.ustc.edu.cn/ubuntu xenial-proposed InRelease                                                                      
Hit:6 http://mirrors.ustc.edu.cn/ubuntu xenial-backports InRelease                                  
Ign:7 https://download.falco.org/packages/deb stable InRelease                                         
Hit:8 https://download.falco.org/packages/deb stable Release                                           
Ign:9 https://mirrors.aliyun.com/kubernetes/apt kubernetes-xenial/main amd64 Packages
Get:9 https://mirrors.aliyun.com/kubernetes/apt kubernetes-xenial/main amd64 Packages [51.6 kB]
Fetched 51.6 kB in 2s (25.8 kB/s)                             
Reading package lists... Done
```

