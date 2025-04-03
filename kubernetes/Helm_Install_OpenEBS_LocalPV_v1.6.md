
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/c8c558db11a543169de2ad071cdba056.png)




# 1. 准备

## 1. 安装集群

- [Kubespray v2.25.0 Online Install Iubernetes v1.29.5](https://ghostwritten.blog.csdn.net/article/details/141222163)

## 2. 划分卷组

- [linux LVM /dev/sdb mount dir /data](https://ghostwritten.blog.csdn.net/article/details/134633286)

```bash
$ docker exec -ti kubespray bash
root@bastion01:/kubespray# ansible -i inventory/sample/inventory.ini all -m shell -a "fdisk -l /dev/sdb |grep sdb"
[WARNING]: Skipping callback plugin 'ara_default', unable to load
kube-node01 | CHANGED | rc=0 >>
Disk /dev/sdb: 50 GiB, 53687091200 bytes, 104857600 sectors
kube-node02 | CHANGED | rc=0 >>
Disk /dev/sdb: 100 GiB, 107374182400 bytes, 209715200 sectors
kube-node03 | CHANGED | rc=0 >>
Disk /dev/sdb: 100 GiB, 107374182400 bytes, 209715200 sectors
kube-node04 | CHANGED | rc=0 >>
Disk /dev/sdb: 100 GiB, 107374182400 bytes, 209715200 sectors
kube-master01 | CHANGED | rc=0 >>
Disk /dev/sdb: 50 GiB, 53687091200 bytes, 104857600 sectors
root@bastion01:/kubespray# ansible -i inventory/sample/inventory.ini all -m shell -a "vgs"
[WARNING]: Skipping callback plugin 'ara_default', unable to load
kube-node02 | CHANGED | rc=0 >>
  VG      #PV #LV #SN Attr   VSize    VFree   
  data-vg   1   0   0 wz--n- <100.00g <100.00g
  rl        1   2   0 wz--n-   38.41g       0 
kube-node01 | CHANGED | rc=0 >>
  VG      #PV #LV #SN Attr   VSize   VFree  
  data-vg   1   0   0 wz--n- <50.00g <50.00g
  rl        1   2   0 wz--n-  38.41g      0 
kube-node03 | CHANGED | rc=0 >>
  VG      #PV #LV #SN Attr   VSize    VFree   
  data-vg   1   0   0 wz--n- <100.00g <100.00g
  rl        1   2   0 wz--n-   38.41g       0 
kube-node04 | CHANGED | rc=0 >>
  VG      #PV #LV #SN Attr   VSize    VFree   
  data-vg   1   0   0 wz--n- <100.00g <100.00g
  rl        1   2   0 wz--n-   38.41g       0 
kube-master01 | CHANGED | rc=0 >>
  VG      #PV #LV #SN Attr   VSize   VFree  
  data-vg   1   0   0 wz--n- <50.00g <50.00g
  rl        1   2   0 wz--n-  38.41g      0 
```

# 2. 驱出 master 污点（可选）
```bash
$ kubectl describe node kube-master01 | grep Taint
Taints:             node-role.kubernetes.io/control-plane:NoSchedule

$ kubectl taint nodes kube-master01 node-role.kubernetes.io/control-plane:NoSchedule-
node/kube-master01 untainted

$ kubectl describe node kube-master01 | grep Taint
Taints:             <none>
```


# 3. 安装 openebs

```bash
export OPENEBS_CONTROLLER_NODE_NAMES="kube-master01"
export OPENEBS_DATA_NODE_NAMES="kube-node01,kube-node02,kube-node03,kube-node04"
export OPENEBS_STORAGECLASS_NAME="openebs-lvmsc-hdd"
export OPENEBS_VG_NAME="data-vg"
export OPENEBS_CREATE_STORAGECLASS="true"
export OPENEBS_KUBE_NAMESPACE="openebs"
```

脚本 `install_openebs_lvmlocalpv.sh`

```bash
#!/usr/bin/env bash

# You must be prepared as follows before run install.sh:
#
# 1. OPENEBS_CONTROLLER_NODE_NAMES MUST be set as environment variable, for an example:
#
#        export OPENEBS_CONTROLLER_NODE_NAMES="master01,master02"
#
# 2. OPENEBS_DATA_NODE_NAMES MUST be set as environment variable, for an example:
#
#        export OPENEBS_DATA_NODE_NAMES="node01,node02"
#
# 4. OPENEBS_STORAGECLASS_NAME MUST be set as environment variable, for an example:
#
#        export OPENEBS_STORAGECLASS_NAME="openebs-lvmsc-hdd"
#
# 3. OPENEBS_VG_NAME MUST be set as environment variable, for an example:
#
#        export OPENEBS_VG_NAME="local_HDD_VG"
#

readonly CHART="openebs-lvmlocalpv/lvm-localpv"
readonly RELEASE="openebs-lvmlocalpv"
readonly TIME_OUT_SECOND="600s"
readonly CHART_VERSION="1.6.0"

OFFLINE_INSTALL="${OFFLINE_INSTALL:-false}"
OPENEBS_KUBE_NAMESPACE="${OPENEBS_KUBE_NAMESPACE:-openebs}"
OPENEBS_CREATE_STORAGECLASS="${OPENEBS_CREATE_STORAGECLASS:-false}"
OPENEBS_STORAGECLASS_YAML="${OPENEBS_STORAGECLASS_YAML:-/tmp/storageclass.yaml}"
INSTALL_LOG_PATH=/tmp/openebs-lvmlocalpv_install-$(date +'%Y-%m-%d_%H-%M-%S').log

OPENEBS_CONTROLLER_RESOURCE_LIMITS_CPU="${OPENEBS_CONTROLLER_RESOURCE_LIMITS_CPU:-500m}"
OPENEBS_CONTROLLER_RESOURCE_LIMITS_MEMORY="${OPENEBS_CONTROLLER_RESOURCE_LIMITS_MEMORY:-512Mi}"
OPENEBS_CONTROLLER_RESOURCE_REQUESTS_CPU="${OPENEBS_CONTROLLER_RESOURCE_REQUESTS_CPU:-500m}"
OPENEBS_CONTROLLER_RESOURCE_REQUESTS_MEMORY="${OPENEBS_CONTROLLER_RESOURCE_REQUESTS_MEMORY:-512Mi}"
OPENEBS_NODE_RESOURCE_LIMITS_CPU="${OPENEBS_NODE_RESOURCE_LIMITS_CPU:-500m}"
OPENEBS_NODE_RESOURCE_LIMITS_MEMORY="${OPENEBS_NODE_RESOURCE_LIMITS_MEMORY:-512Mi}"
OPENEBS_NODE_RESOURCE_REQUESTS_CPU="${OPENEBS_NODE_RESOURCE_REQUESTS_CPU:-500m}"
OPENEBS_NODE_RESOURCE_REQUESTS_MEMORY="${OPENEBS_NODE_RESOURCE_REQUESTS_MEMORY:-512Mi}"

info() {
  echo "[Info][$(date +'%Y-%m-%dT%H:%M:%S%z')]: $*" | tee -a "${INSTALL_LOG_PATH}"
}

error() {
  echo "[Error][$(date +'%Y-%m-%dT%H:%M:%S%z')]: $*" | tee -a "${INSTALL_LOG_PATH}"
  exit 1
}

installed() {
  command -v "$1" >/dev/null 2>&1
}

online_install_lvmlocalpv() {
  # check if openebs-lvmlocalpv already installed
  if helm status ${RELEASE} -n "${OPENEBS_KUBE_NAMESPACE}" &>/dev/null; then
    error "${RELEASE} already installed. Use helm remove it first"
  fi

  info "Start add helm openebs-lvmlocalpv repo"
  helm repo add openebs-lvmlocalpv https://openebs.github.io/lvm-localpv &>/dev/null || error "Helm add openebs-lvmlocalpv repo error."
  info "Start update helm openebs-lvmlocalpv repo"
  helm repo update openebs-lvmlocalpv 2>/dev/null || error "Helm update openebs-lvmlocalpv repo error."

  info "Install openebs-lvmlocalpv, It might take a long time..."
  helm install ${RELEASE} ${CHART} \
    --version "${CHART_VERSION}" \
    --namespace "${OPENEBS_KUBE_NAMESPACE}" \
    --create-namespace \
    --set lvmController.nodeSelector."openebs\.io/control-plane"="enable" \
    --set-string lvmController.resources.limits.cpu="${OPENEBS_CONTROLLER_RESOURCE_LIMITS_CPU}" \
    --set-string lvmController.resources.limits.memory="${OPENEBS_CONTROLLER_RESOURCE_LIMITS_MEMORY}" \
    --set-string lvmController.resources.requests.cpu="${OPENEBS_CONTROLLER_RESOURCE_REQUESTS_CPU}" \
    --set-string lvmController.resources.requests.memory="${OPENEBS_CONTROLLER_RESOURCE_REQUESTS_MEMORY}" \
    --set lvmNode.nodeSelector."openebs\.io/node"="enable" \
    --set-string lvmNode.resources.limits.cpu="${OPENEBS_NODE_RESOURCE_LIMITS_CPU}" \
    --set-string lvmNode.resources.limits.memory="${OPENEBS_NODE_RESOURCE_LIMITS_MEMORY}" \
    --set-string lvmNode.resources.requests.cpu="${OPENEBS_NODE_RESOURCE_REQUESTS_CPU}" \
    --set-string lvmNode.resources.requests.memory="${OPENEBS_NODE_RESOURCE_REQUESTS_MEMORY}" \
    --set lvmPlugin.allowedTopologies='kubernetes\.io/hostname\,openebs\.io/node' \
    --set analytics.enabled=false \
    --timeout $TIME_OUT_SECOND \
    --wait 2>&1 | tee -a "${INSTALL_LOG_PATH}" || {
    error "Fail to install ${RELEASE}."
  }

  #TODO: check more resources after install
}

offline_install_lvmlocalpv() {
  # check if openebs-lvmlocalpv already installed
  if helm status ${RELEASE} -n "${OPENEBS_KUBE_NAMESPACE}" &>/dev/null; then
    error "${RELEASE} already installed. Use helm remove it first"
  fi

  [[ -d "${OPENEBS_CHART_DIR}" ]] || error "OPENEBS_CHART_DIR not exist."

  local image_registry plugin_image_registry
  if [[ -z "${OPENEBS_IMAGE_REGISTRY}" ]]; then
    image_registry='registry.k8s.io/'
    plugin_image_registry=''
  else
    image_registry="${OPENEBS_IMAGE_REGISTRY}/"
    plugin_image_registry="${OPENEBS_IMAGE_REGISTRY}/"
  fi

  info "Install openebs-lvmlocalpv, It might take a long time..."
  helm install ${RELEASE} "${OPENEBS_CHART_DIR}" \
    --namespace "${OPENEBS_KUBE_NAMESPACE}" \
    --create-namespace \
    --set-string lvmController.resizer.image.registry="${image_registry}" \
    --set-string lvmController.snapshotter.image.registry="${image_registry}" \
    --set-string lvmController.snapshotController.image.registry="${image_registry}" \
    --set-string lvmController.provisioner.image.registry="${image_registry}" \
    --set lvmController.nodeSelector."openebs\.io/control-plane"="enable" \
    --set-string lvmController.resources.limits.cpu="${OPENEBS_CONTROLLER_RESOURCE_LIMITS_CPU}" \
    --set-string lvmController.resources.limits.memory="${OPENEBS_CONTROLLER_RESOURCE_LIMITS_MEMORY}" \
    --set-string lvmController.resources.requests.cpu="${OPENEBS_CONTROLLER_RESOURCE_REQUESTS_CPU}" \
    --set-string lvmController.resources.requests.memory="${OPENEBS_CONTROLLER_RESOURCE_REQUESTS_MEMORY}" \
    --set-string lvmNode.driverRegistrar.image.registry="${image_registry}" \
    --set lvmNode.nodeSelector."openebs\.io/node"="enable" \
    --set-string lvmNode.resources.limits.cpu="${OPENEBS_NODE_RESOURCE_LIMITS_CPU}" \
    --set-string lvmNode.resources.limits.memory="${OPENEBS_NODE_RESOURCE_LIMITS_MEMORY}" \
    --set-string lvmNode.resources.requests.cpu="${OPENEBS_NODE_RESOURCE_REQUESTS_CPU}" \
    --set-string lvmNode.resources.requests.memory="${OPENEBS_NODE_RESOURCE_REQUESTS_MEMORY}" \
    --set-string lvmPlugin.image.registry="${plugin_image_registry}" \
    --set lvmPlugin.allowedTopologies='kubernetes\.io/hostname\,openebs\.io/node' \
    --set analytics.enabled=false \
    --timeout $TIME_OUT_SECOND \
    --wait 2>&1 | tee -a "${INSTALL_LOG_PATH}" || {
    error "Fail to install ${RELEASE}."
  }

  #TODO: check more resources after install
}

verify_supported() {
  installed helm || error "helm is required"
  installed kubectl || error "kubectl is required"
  installed curl || error "curl is required"
  installed envsubst || error "envsubst is required"

  [[ -n "${OPENEBS_STORAGECLASS_NAME}" ]] || error "OPENEBS_STORAGECLASS_NAME MUST set in environment variable."
  [[ -n "${OPENEBS_VG_NAME}" ]] || error "OPENEBS_VG_NAME MUST set in environment variable."

  [[ -n "${OPENEBS_CONTROLLER_NODE_NAMES}" ]] || error "OPENEBS_CONTROLLER_NODE_NAMES MUST set in environment variable."
  local node
  local control_node_array
  IFS="," read -r -a control_node_array <<<"${OPENEBS_CONTROLLER_NODE_NAMES}"
  for node in "${control_node_array[@]}"; do
    kubectl label node "${node}" 'openebs.io/control-plane=enable' --overwrite &>/dev/null || {
      error "kubectl label node ${node} 'openebs.io/control-plane=enable' failed, use kubectl to check reason"
    }
  done
  [[ -n "${OPENEBS_DATA_NODE_NAMES}" ]] || error "OPENEBS_DATA_NODE_NAMES MUST set in environment variable."
  local data_node_array
  IFS="," read -r -a data_node_array <<<"${OPENEBS_DATA_NODE_NAMES}"
  for node in "${data_node_array[@]}"; do
    kubectl label node "${node}" 'openebs.io/node=enable' --overwrite &>/dev/null || {
      error "kubectl label node ${node} 'openebs.io/node=enable' failed, use kubectl to check reason"
    }
  done
}

init_log() {
  touch "${INSTALL_LOG_PATH}" || error "Create log file ${INSTALL_LOG_PATH} error"
  info "Log file create in path ${INSTALL_LOG_PATH}"
}

############################################
# Check if helm release deployment correctly
# Arguments:
#   release
#   namespace
############################################
verify_installed() {
  local status
  status=$(helm status "${RELEASE}" -n "${OPENEBS_KUBE_NAMESPACE}" | grep ^STATUS: | awk '{print $2}')
  [[ "${status}" == "deployed" ]] || {
    error "Helm release ${RELEASE} status is not deployed, use helm to check reason"
  }

  info "${RELEASE} Deployment Completed!"
}

create_storageclass() {
  [[ -f ${OPENEBS_STORAGECLASS_YAML} ]] || {
    local download_url="https://raw.githubusercontent.com/upmio/upm-deploy/main/addons/openebs-lvmlocalpv/yaml/storageclass.yaml"
    curl -sSL "${download_url}" -o "${OPENEBS_STORAGECLASS_YAML}" || {
      error "curl get storageclass.yaml failed"
    }
  }

  OPENEBS_STORAGECLASS_NAME="${OPENEBS_STORAGECLASS_NAME}" \
    OPENEBS_VG_NAME="${OPENEBS_VG_NAME}" \
    envsubst <"${OPENEBS_STORAGECLASS_YAML}" | kubectl apply -f - || {
    error "kubectl create storageclass failed, check log use kubectl."
  }

  info "create storageclass successful!"
}

main() {
  init_log
  verify_supported
  if [[ ${OFFLINE_INSTALL} == "false" ]]; then
    online_install_lvmlocalpv
  elif [[ ${OFFLINE_INSTALL} == "true" ]]; then
    offline_install_lvmlocalpv
  fi
  verify_installed
  if [[ ${OPENEBS_CREATE_STORAGECLASS} == "true" ]]; then
    create_storageclass
  else
    info "OPENEBS_CREATE_STORAGECLASS disable, skip create storageclass."
  fi
}

main
```
执行：

```bash
$ sh install_openebs_lvmlocalpv.sh
```
输出：

```bash
[Info][2024-08-15T17:04:13+0800]: Log file create in path /tmp/openebs-lvmlocalpv_install-2024-08-15_17-04-13.log
[Info][2024-08-15T17:04:14+0800]: Start add helm openebs-lvmlocalpv repo
[Info][2024-08-15T17:04:14+0800]: Start update helm openebs-lvmlocalpv repo
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "openebs-lvmlocalpv" chart repository
Update Complete. ⎈Happy Helming!⎈
[Info][2024-08-15T17:04:15+0800]: Install openebs-lvmlocalpv, It might take a long time...
WARNING: Kubernetes configuration file is group-readable. This is insecure. Location: /root/.kube/config
WARNING: Kubernetes configuration file is world-readable. This is insecure. Location: /root/.kube/config
NAME: openebs-lvmlocalpv
LAST DEPLOYED: Thu Aug 15 17:04:18 2024
NAMESPACE: openebs
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
The OpenEBS LVM LocalPV has been installed. Check its status by running:
$ kubectl get pods -n openebs -l role=openebs-lvm

For more information, visit our Slack at https://openebs.io/community or view 
the documentation online at http://docs.openebs.io/.

[Info][2024-08-15T17:05:26+0800]: openebs-lvmlocalpv Deployment Completed!
storageclass.storage.k8s.io/openebs-lvmsc-hdd created
[Info][2024-08-15T17:05:29+0800]: create storageclass successful!
```
检查运行
```bash
$ helm ls -n openebs
NAME                    NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                   APP VERSION
openebs-lvmlocalpv      openebs         1               2024-08-15 17:04:18.917045172 +0800 CST deployed        lvm-localpv-1.6.0       1.6.0 

$ kubectl get pods -n openebs -l role=openebs-lvm
NAME                                                         READY   STATUS    RESTARTS   AGE
openebs-lvmlocalpv-lvm-localpv-controller-55f4c9d59c-mnjx5   5/5     Running   0          2m17s
openebs-lvmlocalpv-lvm-localpv-node-2lkxv                    2/2     Running   0          2m17s
openebs-lvmlocalpv-lvm-localpv-node-556zb                    2/2     Running   0          2m17s
openebs-lvmlocalpv-lvm-localpv-node-5tf76                    2/2     Running   0          2m17s
openebs-lvmlocalpv-lvm-localpv-node-hch94                    2/2     Running   0          2m17s

$ kubectl get sc
NAME                PROVISIONER            RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
openebs-lvmsc-hdd   local.csi.openebs.io   Delete          WaitForFirstConsumer   true                   9m30s

#设置默认sc
$ kubectl patch storageclass openebs-lvmsc-hdd  -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
storageclass.storage.k8s.io/openebs-lvmsc-hdd patched

$ kubectl get sc
NAME                          PROVISIONER            RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
openebs-lvmsc-hdd (default)   local.csi.openebs.io   Delete          WaitForFirstConsumer   true                   159m
```


参考：

- [https://github.com/openebs/lvm-localpv](https://github.com/openebs/lvm-localpv)
