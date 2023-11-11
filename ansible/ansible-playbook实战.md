[https://www.kancloud.cn/willseecloud/ansible/1472273](https://www.kancloud.cn/willseecloud/ansible/1472273)

```bash
cat yum.yaml 
---

- name: yum 
  gather_facts: false
  hosts: all
  tasks:

    - name: copy yum files
      copy:
        src: '{{ item.src }}'
        dest: '/etc/yum.repos.d/'
      with_items:
        - { src: '/data/yum/CentOS-Base.repo' }
        - { src: '/data/yum/docker-ce.repo' }
        - { src: '/data/yum/kubernetes.repo' }
```

