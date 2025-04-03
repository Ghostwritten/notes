
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8e64e87868aeeea7e594929a427a9a60.jpeg#pic_center)

## 1. 简介

为了应对手动修改containerd配置过程中遇到的复杂性和维护性问题，我编写了一个脚本，该脚本可以批量修改containerd配置，从而简化这一过程。以下是手动修改containerd配置的主要缺点以及该脚本如何解决这些问题：

手动修改containerd配置的缺点

- 复杂性和出错风险：

  - 多文件、多参数：containerd的配置文件可能包含多个参数，手动修改容易出错。
  - 不一致性：手动修改容易导致配置不一致，特别是在大规模集群中。

- 时间消耗：

  - 逐一修改：手动修改每个节点上的配置文件需要耗费大量时间，尤其是在大规模部署中。
  - 验证和测试：手动修改后需要逐一验证和测试，增加了时间成本。

- 可维护性差：

  - 手动记录：手动修改后需要记录和跟踪修改内容，容易遗漏或记录不完整。
  - 版本控制：难以有效管理和版本控制配置的变更。

- 重复性工作：

  - 重复操作：在多个环境中（如开发、测试、生产）重复相同的修改工作，效率低下。

脚本的解决方案
针对以上缺点，我编写的脚本具有以下特点和优势：

- 自动化批量修改：

  - 批量处理：脚本可以一次性对多个节点的containerd配置进行批量修改，确保配置的一致性。
  - 减少出错风险：通过自动化脚本，减少了手动修改时的出错风险。

- 时间效率：

  - 快速执行：脚本能够快速执行配置修改，节省大量时间，特别是在大规模集群环境中。
  - 并行处理：支持并行修改多个节点的配置，加快整体操作速度。

- 易于维护和管理：

  - 集中管理：脚本提供集中管理的能力，可以统一记录和跟踪配置的变更。
  - 版本控制友好：脚本的修改记录可以方便地纳入版本控制系统，管理配置变更更加高效。

- 降低重复性工作：

  - 模板化配置：通过脚本实现配置模板化，在不同环境中可以轻松复用，减少重复工作量。
  - 环境一致性：确保不同环境下的配置一致性，简化运维工作。

通过使用该脚本，可以大幅简化containerd配置的修改过程，提高效率，降低出错风险，并增强配置管理的可维护性。


## 2. 功能

 - [✅] 添加非认证镜像仓库
 - [✅] 添加用户&无证书认证镜像仓库
 - [✅] 添加用户&证书认证镜像仓库
 - [ ] 删除非认证镜像仓库


## 3. 脚本


```bash
#!/bin/bash

reg_name=$1
reg_port=$2
reg_user=$3
reg_password=$4
reg_str='registry.configs'
[ -n "$reg_port" ] && reg_port=":$2"

reload_config() {

  [ -n "$reg_name" ] || { echo "错误：变量 reg_name 不能为空！"; exit 1; }
  local config='/etc/containerd/config.toml'
  if ! grep -q "${reg_name}${reg_port}" $config;then
    if grep -q "registry.mirrors]" $config;then
      registry_mirrors=$(grep "registry.mirrors]" $config | awk '{print $1}' |  sed 's/\[/\\\[/g;s/\]/\\\]/g;s/\./\\\./g;s/\"/\\\"/g' )

      sudo sed -i "/${registry_mirrors}/ a\ \ \ \ \ \ \ \ \[plugins.\"io.containerd.grpc.v1.cri\".registry.mirrors.\"${reg_name}${reg_port}\"]\n\ \ \ \ \ \ \ \ \ \ endpoint = [\"http://${reg_name}${reg_port}\"]" $config

    fi
  fi
  systemctl  restart containerd && systemctl status containerd |grep Active
}

reload_configs_tls() {

  [ -n "$reg_name" ] || { echo "错误：变量 reg_name 不能为空！"; exit 1; }

  local config='/etc/containerd/config.toml'

  if ! grep -q "$reg_name${reg_port}" $config;then
    if grep -q "${reg_str}]" $config;then
       registry_configs=$(grep "${reg_str}]" $config | awk '{print $1}' |  sed 's/\[/\\\[/g;s/\]/\\\]/g;s/\./\\\./g;s/\"/\\\"/g' )
       sed -i "/${registry_configs}/ a\ \ \ \ \ \ \ \ \[plugins\.\"io\.containerd\.grpc\.v1\.cri\".${reg_str}.\"${reg_name}${reg_port}\".tls\]\n\ \ \ \ \ \ \ \ \ \ insecure_skip_verify\ =\ true\n" $config
       sed -i "/${registry_configs}/ a\ \ \ \ \ \ \ \ \[plugins\.\"io\.containerd\.grpc\.v1\.cri\".${reg_str}.\"${reg_name}${reg_port}\".auth\]\n\ \ \ \ \ \ \ \ \ \ username\ =\ \"${reg_user}\"\n\ \ \ \ \ \     password\ =\ \"${reg_password}\"" $config
    fi
  fi
  systemctl  restart containerd && systemctl status containerd |grep Active
 
}

reload_configs_tls_certs() {

  [ -n "$reg_name" ] || { echo "错误：变量 reg_name 不能为空！"; exit 1; }

  local config='/etc/containerd/config.toml'

  if ! grep -q "$reg_name${reg_port}" $config;then
    if grep -q "${reg_str}]" $config;then
       registry_configs=$(grep "${reg_str}]" $config | awk '{print $1}' |  sed 's/\[/\\\[/g;s/\]/\\\]/g;s/\./\\\./g;s/\"/\\\"/g' )
       sed -i "/${registry_configs}/ a\ \ \ \ \ \ \ \ \[plugins\.\"io\.containerd\.grpc\.v1\.cri\".${reg_str}.\"${reg_name}${reg_port}\".tls\]\n\ \ \ \ \ \ \ \ \ \ insecure_skip_verify\ =\ true\n\ \ \ \ \ \     ca_file\ =\ \"\/etc\/docker\/certs.d\/${reg_name}${reg_port}\/${reg_name}.crt\"" $config
       sed -i "/${registry_configs}/ a\ \ \ \ \ \ \ \ \[plugins\.\"io\.containerd\.grpc\.v1\.cri\".${reg_str}.\"${reg_name}${reg_port}\".auth\]\n\ \ \ \ \ \ \ \ \ \ username\ =\ \"${reg_user}\"\n\ \ \ \ \ \     password\ =\ \"${reg_password}\"" $config
    fi
  fi
  systemctl  restart containerd && systemctl status containerd |grep Active
 
}



reload_mirrors_delete() {

  local config='/etc/containerd/config.toml'
 # local config='config.toml'
  if  grep -q "$reg_name" $config;then
    if grep -q "${reg_str}]" $config;then
      registry_configs=$(grep "${reg_str}]" $config | awk '{print $1}' |  sed 's/\[/\\\[/g;s/\]/\\\]/g;s/\./\\\./g;s/\"/\\\"/g' )

      sudo sed -i "/\[plugins.\"io.containerd.grpc.v1.cri\".registry.mirrors.\"${reg_name}:${reg_port}\"\]/d;/endpoint\ \=\ \[\"http:\/\/${reg_name}:${reg_port}\"\]/d" $config

    else
      sudo sed -i "/\[plugins.\"io.containerd.grpc.v1.cri\".registry.mirrors.\"${reg_name}:${reg_port}\"\]/d;/endpoint\ \=\ \[\"http:\/\/${reg_name}:${reg_port}\"\]/d" $config
    fi
  fi
  systemctl  restart containerd && systemctl status containerd |grep Active


}

#reload_config
#reload_configs_tls
#reload_configs_tls_certs
#reload_mirrors_delete
```

