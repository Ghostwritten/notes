

--

## 1. 背景
当我们写了一遍`vaule.yaml`，再写一遍`value.schema.json`这是非常繁琐的。因此，有这样一块关于helm的插件使用一条命令即可达到目的。

[helm-schema-gen](https://github.com/karuppiah7890/helm-schema-gen)

##  2. 安装

该插件适用于 Helm v2 和 v3 版本，因为它与 Helm 二进制版本无关

```bash
$ helm plugin install https://github.com/karuppiah7890/helm-schema-gen.git
karuppiah7890/helm-schema-gen info checking GitHub for tag '0.0.4'
karuppiah7890/helm-schema-gen info found version: 0.0.4 for 0.0.4/Darwin/x86_64
karuppiah7890/helm-schema-gen info installed ./bin/helm-schema-gen
Installed plugin: schema-gen
```

或者你也可以下载[helm-schema-gen](https://github.com/karuppiah7890/helm-schema-gen/releases/tag/0.0.4)命令

```bash
tar tvf helm-schema-gen_0.0.4_Linux_x86_64.tar
./helm-schema-gen --help
mv helm-schema-gen /usr/local/bin/
```

##  3. 用法
该插件适用于 Helm v2 和 v3 版本

让我们采取values.yaml如下示例

```yaml
replicaCount: 1

image:
  repository: nginx
  pullPolicy: IfNotPresent

imagePullSecrets: []
nameOverride: ""
fullnameOverride: ""

serviceAccount:
  # Specifies whether a service account should be created
  create: true
  # The name of the service account to use.
  # If not set and create is true, a name is generated using the fullname template
  name:

podSecurityContext: {}
  # fsGroup: 2000

securityContext: {}
  # capabilities:
  #   drop:
  #   - ALL
  # readOnlyRootFilesystem: true
  # runAsNonRoot: true
  # runAsUser: 1000

service:
  type: ClusterIP
  port: 80

ingress:
  enabled: false
  annotations: {}
    # kubernetes.io/ingress.class: nginx
    # kubernetes.io/tls-acme: "true"
  hosts:
    - host: chart-example.local
      paths: []
  tls: []
  #  - secretName: chart-example-tls
  #    hosts:
  #      - chart-example.local

resources: {}
  # We usually recommend not to specify default resources and to leave this as a conscious
  # choice for the user. This also increases chances charts run on environments with little
  # resources, such as Minikube. If you do want to specify resources, uncomment the following
  # lines, adjust them as necessary, and remove the curly braces after 'resources:'.
  # limits:
  #   cpu: 100m
  #   memory: 128Mi
  # requests:
  #   cpu: 100m
  #   memory: 128Mi

nodeSelector: {}

tolerations: []

affinity: {}
```
现在，如果您使用插件并将 传递`values.yaml`给它，您将获得 JSON `Schemavalues.yaml`

```vbnet
$ helm schema-gen values.yaml
{
    "$schema": "http://json-schema.org/schema#",
    "type": "object",
    "properties": {
        "affinity": {
            "type": "object"
        },
        "fullnameOverride": {
            "type": "string"
        },
        "image": {
            "type": "object",
            "properties": {
                "pullPolicy": {
                    "type": "string"
                },
                "repository": {
                    "type": "string"
                }
            }
        },
        "imagePullSecrets": {
            "type": "array"
        },
        "ingress": {
            "type": "object",
            "properties": {
                "annotations": {
                    "type": "object"
                },
                "enabled": {
                    "type": "boolean"
                },
                "hosts": {
                    "type": "array",
                    "items": {
                        "type": "object",
                        "properties": {
                            "host": {
                                "type": "string"
                            },
                            "paths": {
                                "type": "array"
                            }
                        }
                    }
                },
                "tls": {
                    "type": "array"
                }
            }
        },
        "nameOverride": {
            "type": "string"
        },
        "nodeSelector": {
            "type": "object"
        },
        "podSecurityContext": {
            "type": "object"
        },
        "replicaCount": {
            "type": "integer"
        },
        "resources": {
            "type": "object"
        },
        "securityContext": {
            "type": "object"
        },
        "service": {
            "type": "object",
            "properties": {
                "port": {
                    "type": "integer"
                },
                "type": {
                    "type": "string"
                }
            }
        },
        "serviceAccount": {
            "type": "object",
            "properties": {
                "create": {
                    "type": "boolean"
                },
                "name": {
                    "type": "null"
                }
            }
        },
        "tolerations": {
            "type": "array"
        }
    }
}
```
您可以将其保存到这样的文件中

```vbnet
$ helm schema-gen values.yaml > values.schema.json
```
如果是下载的[helm-schema-gen](https://github.com/karuppiah7890/helm-schema-gen/releases/tag/0.0.4)命令

```bash
helm-schema-gen value.yaml  > values.schema.json
```
---
✈<font color=	#FF4500 size=4 style="font-family:Courier New">推荐阅读：</font>


 - [helm 快速学习手册](https://ghostwritten.blog.csdn.net/article/details/122882625)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/cd65c4a1beab01c3cc8e1e13f556ab42.gif#pic_center)


