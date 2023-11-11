## 标签

```bash
1.version
2.service
3.network



service：

1.image
2.build
3.command
4.container_name
5.depend_on
6.dns
7.tmpfs
8.entrypoint
9.env_file
10.environment
11.expose
12.external_links
13.external_hosts
14.labels
15.links
16.loging
17.pid
18.ports
19.security_opt
20.stop_signal
21.volumes?
22. volumes_from
23. cap_add, cap_drop
24.cgroup_parent
25. devices?
26. extends
27. network_mode
28. networks
```

还有这些标签：`cpu_shares, cpu_quota, cpuset, domainname, hostname, ipc, mac_address, mem_limit, memswap_limit, privileged, read_only, restart, shm_size, stdin_open, tty, user, working_dir`
上面这些都是一个单值的标签，类似于使用docker run的效果。

## 实例
