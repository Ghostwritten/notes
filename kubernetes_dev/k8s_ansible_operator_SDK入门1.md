
初始化项目
```bash
$ operator-sdk new memcached-operator --api-version=cache.example.com/v1alpha1 --kind=Memcached --type=ansible
INFO[0000] Creating new Ansible operator 'memcached-operator'. 
INFO[0000] Created deploy/service_account.yaml          
INFO[0000] Created deploy/role.yaml                     
INFO[0000] Created deploy/role_binding.yaml             
INFO[0000] Created deploy/crds/cache.example.com_memcacheds_crd.yaml 
INFO[0000] Created deploy/crds/cache.example.com_v1alpha1_memcached_cr.yaml 
INFO[0000] Created build/Dockerfile                     
INFO[0000] Created roles/memcached/README.md            
INFO[0000] Created roles/memcached/meta/main.yml        
INFO[0000] Created roles/memcached/files/.placeholder   
INFO[0000] Created roles/memcached/templates/.placeholder 
INFO[0000] Created roles/memcached/vars/main.yml        
INFO[0000] Created molecule/test-local/playbook.yml     
INFO[0000] Created roles/memcached/defaults/main.yml    
INFO[0000] Created roles/memcached/tasks/main.yml       
INFO[0000] Created molecule/default/molecule.yml        
INFO[0000] Created build/test-framework/Dockerfile      
INFO[0000] Created molecule/test-cluster/molecule.yml   
INFO[0000] Created molecule/default/prepare.yml         
INFO[0000] Created molecule/default/playbook.yml        
INFO[0000] Created build/test-framework/ansible-test.sh 
INFO[0000] Created molecule/default/asserts.yml         
INFO[0000] Created molecule/test-cluster/playbook.yml   
INFO[0000] Created roles/memcached/handlers/main.yml    
INFO[0000] Created watches.yaml                         
INFO[0000] Created deploy/operator.yaml                 
INFO[0000] Created .travis.yml                          
INFO[0000] Created molecule/test-local/molecule.yml     
INFO[0000] Created molecule/test-local/prepare.yml      
INFO[0000] Project creation complete.     
```

```bash
# ls
build  deploy  molecule  roles  watches.yaml
```

```go
build      #构建执行ansible-playbook命令的环境容器
deploy     #部署operator
molecule   #集中管理ansible-playbook功能
roles      #实现ansible-playbook子任务 
watches.yaml            
```
