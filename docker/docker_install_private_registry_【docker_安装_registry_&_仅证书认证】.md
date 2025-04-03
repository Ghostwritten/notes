
预备条件：
- [安装docker](https://blog.csdn.net/xixihahalelehehe/article/details/104293170)

我们设定镜像仓库域名为`registry01.dev.com`

配置/etc/hosts

```bash
192.168.23.51 registry01.dev.com
```

安装 registry
```bash
#!/bin/bash

reg_ip=$1
reg_n=$2
reg_port=$3

if [ $# -eq 0 ]; then
  echo "Usage: $0 [reg_ip] [registry_name]"
  echo "Please provide one or more arguments."
  exit 1
fi

BASE_DIR="$(dirname "$(readlink -f "${0}")")"
DEST_DIR='/registry'
certs_dir='/registry/certs'
data_dir='/data/registry'
mkdir -p $DEST_DIR
mkdir -p $certs_dir
mkdir -p $data_dir


image_load(){
  docker load -i ${DEST_DIR}/images/registry_latest.tar

}


# create tls certs for docker registry
create_certs() {


cat << EOF > ${DEST_DIR}/ssl.conf
[ req ]
prompt             = no
distinguished_name = req_subj
x509_extensions    = x509_ext

[ req_subj ]
CN = Localhost

[ x509_ext ]
subjectKeyIdentifier   = hash
authorityKeyIdentifier = keyid,issuer
basicConstraints       = CA:true
subjectAltName         = @alternate_names

[ alternate_names ]
DNS.1 = $reg_n
IP.1  = $reg_ip
EOF




openssl req -config  ${DEST_DIR}/ssl.conf -new -x509 -nodes -sha256 -days 365 -newkey rsa:4096 -keyout ${DEST_DIR}/${reg_n}.key -out ${DEST_DIR}/${reg_n}.crt
openssl x509 -inform PEM -in ${DEST_DIR}/${reg_n}.crt -out ${DEST_DIR}/${reg_n}.cert

}

# deploy docker registry
run_reg () {

cp ${DEST_DIR}/${reg_n}.key ${DEST_DIR}/${reg_n}.crt ${DEST_DIR}/${reg_n}.cert  $certs_dir


 docker run -d --privileged=true --restart=always --name registry-tls-certs  -v ${certs_dir}:/certs  -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/${reg_n}.crt -e REGISTRY_HTTP_TLS_KEY=/certs/${reg_n}.key -e REGISTRY_COMPATIBILITY_SCHEMA1_ENABLED=true -e REGISTRY_STORAGE_DELETE_ENABLED=true  -p 443:443 -p $reg_port:5000  -v ${data_dir}:/var/lib/registry/docker/registry  registry
 if [ $? != 0 ];then
    echo "contianer create failed" && exit 1
 fi

[ -d /etc/docker/certs.d/${reg_n}:$reg_port ]  || mkdir -p /etc/docker/certs.d/${reg_n}:${reg_port}
cp -r ${certs_dir}/${reg_n}.crt   /etc/docker/certs.d/${reg_n}:${reg_port}/
systemctl restart docker

}

# test push
push_images() {

 docker tag registry:latest ${reg_n}:${reg_port}/registry:latest
 docker push ${reg_n}:${reg_port}/registry:latest

}

image_load
create_certs
run_reg
push_images

```

执行

```bash
sh -x  install_registry.sh  192.168.23.51 registry01.dev.com 80
```
输出：

```bash
sh -x install_registry.sh 192.168.23.52 registry02.dev.com 80
+ reg_ip=192.168.23.52
+ reg_n=registry02.dev.com
+ reg_port=80
+ '[' 3 -eq 0 ']'
+++ readlink -f install_registry.sh
++ dirname /root/install_registry.sh
+ BASE_DIR=/root
+ DEST_DIR=/registry
+ certs_dir=/registry/certs
+ data_dir=/data/registry
+ mkdir -p /registry
+ mkdir -p /registry/certs
+ mkdir -p /data/registry
+ image_load
+ docker load -i /registry/images/registry_latest.tar
open /registry/images/registry_latest.tar: no such file or directory
+ create_certs
+ cat
+ openssl req -config /registry/ssl.conf -new -x509 -nodes -sha256 -days 365 -newkey rsa:4096 -keyout /registry/registry02.dev.com.key -out /registry/registry02.dev.com.crt
Generating a RSA private key
.........................++++
............................................................................................................................................................................................................................................................................................................................................................................++++
writing new private key to '/registry/registry02.dev.com.key'
-----
+ openssl x509 -inform PEM -in /registry/registry02.dev.com.crt -out /registry/registry02.dev.com.cert
+ run_reg
+ cp /registry/registry02.dev.com.key /registry/registry02.dev.com.crt /registry/registry02.dev.com.cert /registry/certs
+ docker run -d --privileged=true --restart=always --name registry-tls-certs -v /registry/certs:/certs -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/registry02.dev.com.crt -e REGISTRY_HTTP_TLS_KEY=/certs/registry02.dev.com.key -e REGISTRY_COMPATIBILITY_SCHEMA1_ENABLED=true -e REGISTRY_STORAGE_DELETE_ENABLED=true -p 443:443 -p 80:5000 -v /data/registry:/var/lib/registry/docker/registry registry
Unable to find image 'registry:latest' locally
latest: Pulling from library/registry
619be1103602: Pull complete 
2ba4b87859f5: Pull complete 
0da701e3b4d6: Pull complete 
14a4d5d702c7: Pull complete 
d1a4f6454cb2: Pull complete 
Digest: sha256:f4e1b878d4bc40a1f65532d68c94dcfbab56aa8cba1f00e355a206e7f6cc9111
Status: Downloaded newer image for registry:latest
ef764fc4e390850d45f5b97bc44cccba8aa630e1732be41503ddc2d1f91a31a6
+ '[' 0 '!=' 0 ']'
+ '[' -d /etc/docker/certs.d/registry02.dev.com:80 ']'
+ mkdir -p /etc/docker/certs.d/registry02.dev.com:80
+ cp -r /registry/certs/registry02.dev.com.crt /etc/docker/certs.d/registry02.dev.com:80/
+ systemctl restart docker
+ push_images
+ docker tag registry:latest registry02.dev.com:80/registry:latest
+ docker push registry02.dev.com:80/registry:latest
The push refers to repository [registry02.dev.com:80/registry]
a2e9568f0343: Pushed 
95d5b7fa5097: Pushed 
bf7f68cf6cd2: Pushed 
98e9164d5432: Pushed 
aedc3bda2944: Pushed 
latest: digest: sha256:12202eb78732e22f8658d595bd6e3d47ef9f13ede78e94e90974c020c7d7c1b3 size: 1363
```

