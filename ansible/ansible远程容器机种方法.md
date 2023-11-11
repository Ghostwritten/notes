
1.
```yaml
---
- hosts: [my-cluster-of-servers]

  tasks: 
    - name: Go Into Docker Container And Run Multiple Commands
      docker:
        name: [container-name]
        image: [image-ive-created-container-with-on-server]
        state: present
        command: docker exec -it [container-name] bash
```



2.
```yaml
  - name: Run docker exec command
    docker_exec: 
      command: <some command>
      docker_host: <docker host>
      name: <container name>
    register: exec_output

 - name: Show exec output
   debug: msg="{{ exec_output.result }}"
```

3.

```yaml
- name: add container to inventory
  add_host:
    name: [container-name]
    ansible_connection: docker
  changed_when: false

- name: run command in container
  delegate_to: [container-name]
  raw: bash
```

远程docker参数
ansible_docker_extra_args: "-H=tcp://[docker-host]:[api port]"

4.

```yaml
 tasks:
 - name: Execute commands in docker container
   command: docker exec -it my_container bash -c 'echo "Hello1"; echo "Hello2"'
```
5.

```yaml
- name: execute command in docker
  shell: |
  docker exec container sh -l -c "cat /tmp/secret"
  register: hello

- debug: msg="{{ hello.stdout }}"
```





















