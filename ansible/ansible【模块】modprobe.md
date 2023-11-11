

----
## 1. 简介
加载或卸载内核模块
## 2. 参数

```bash
name   要管理的内核模块的名称。
params  模块参数
state   模块是否应该存在或不存在。
```

## 3. 示例

```bash
 name: Add the 802.1q module
  modprobe:
    name: 8021q
    state: present

- name: Add the dummy module
  modprobe:
    name: dummy
    state: present
    params: 'numdummies=2'
    
- name: Modprode Kernel Module for IPVS
  modprobe:
    name: "{{ item }}"
    state: present
  with_items:
    - ip_vs
    - ip_vs_rr
    - ip_vs_wrr
    - ip_vs_sh
  when: kube_proxy_mode == 'ipvs'
  tags:
    - kube-proxy
```

