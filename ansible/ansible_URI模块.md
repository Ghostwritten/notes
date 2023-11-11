```bash
tasks:

  - action: uri url=http://www.example.com return_content=yes
    register: webpage

  - fail: msg='service is not happy'
    when: "'AWESOME' not in webpage.content"
```

