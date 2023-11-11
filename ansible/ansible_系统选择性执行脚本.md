```bash
when: ansible_os_family == "CentOS"
when: ansible_os_family == "Redhat"
when: ansible_os_family == "Darwin"
when: ansible_os_family == "Debian"
when: ansible_os_family == "Windows"
```

这里的任意任务，不需要，但重点是您可以在`main.yml`中直接拥有任何通用任务

```bash
- name: get the date
  shell: `date`
  register: date

- include: debian.yml
  when: ansible_os_family == 'Debian'

- include: redhat.yml
  when: ansible_os_family == 'RedHat'
```

**debian.yml** 

```bash
- name: install requirements
  apt: name={{item}} state=latest update_cache=true
  with_items:
      - gcc
      - python-dev
      - python-setuptools
      - python-software-properties
```

**redhat.yml**

```bash
- name: install requirements
  yum: name={{item}} state=latest update_cache=true
  with_items:
      - gcc
      - python-dev
      - python-setuptools
      - python-software-properties
```


如果你想，你也可以有条件地包含OS家族(或任何你可以检查的事实)特定的变量，像这样:

```bash
- name: Include OS-specific variables.
  include_vars: "{{ item }}"
  with_first_found:
    - ../vars/{{ ansible_distribution | lower }}.yml
    - ../vars/{{ ansible_os_family | lower }}.yml
```

然后在`vars/debian`中设置依赖列表

```bash
python_dependencies:
  - gcc
  - python-dev
  - python-setuptools
  - python-software-properties
```

**tasks/debian.yml**

```bash
- name: install requirements
  apt: name={{item}} state=latest update_cache=true
  with_items: python_dependencies
```

