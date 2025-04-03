

```bash
$ subscription-manager register
Registering to: subscription.rhsm.redhat.com:443/subscription
Username: xxx@xx.com
Password: 
The system has been registered with ID: 4d08e201xxxxxxx1
The registered system name is: localhost.localdomain

$ dnf repolist --verbose
Loaded plugins: builddep, changelog, config-manager, copr, debug, debuginfo-install, download, generate_completion_cache, groups-manager, kpatch, needs-restarting, notify-packagekit, playground, product-id, repoclosure, repodiff, repograph, repomanage, reposync, subscription-manager, system-upgrade, uploadprofile
Updating Subscription Management repositories.
DNF version: 4.14.0
cachedir: /var/cache/dnf
Last metadata expiration check: 0:02:53 ago on Wed 30 Oct 2024 02:53:11 PM CST.
Repo-id            : rhel-9-for-x86_64-appstream-rpms
Repo-name          : Red Hat Enterprise Linux 9 for x86_64 - AppStream (RPMs)
Repo-revision      : 1730247754
Repo-updated       : Wed 30 Oct 2024 08:22:33 AM CST
Repo-pkgs          : 19,639
Repo-available-pkgs: 18,927
Repo-size          : 69 G
Repo-baseurl       : https://cdn.redhat.com/content/dist/rhel9/9/x86_64/appstream/os
Repo-expire        : 86,400 second(s) (last: Wed 30 Oct 2024 02:53:10 PM CST)
Repo-filename      : /etc/yum.repos.d/redhat.repo

Repo-id            : rhel-9-for-x86_64-baseos-rpms
Repo-name          : Red Hat Enterprise Linux 9 for x86_64 - BaseOS (RPMs)
Repo-revision      : 1730247532
Repo-updated       : Wed 30 Oct 2024 08:18:52 AM CST
Repo-pkgs          : 7,326
Repo-available-pkgs: 7,326
Repo-size          : 18 G
Repo-baseurl       : https://cdn.redhat.com/content/dist/rhel9/9/x86_64/baseos/os
Repo-expire        : 86,400 second(s) (last: Wed 30 Oct 2024 02:53:11 PM CST)
Repo-filename      : /etc/yum.repos.d/redhat.repo
Total packages: 26,965

$ dnf -y update
```


参考：

- [Registering the system and managing subscriptions](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/9/html/configuring_basic_system_settings/assembly_registering-the-system-and-managing-subscriptions_configuring-basic-system-settings#assembly_registering-the-system-and-managing-subscriptions_configuring-basic-system-settings)
