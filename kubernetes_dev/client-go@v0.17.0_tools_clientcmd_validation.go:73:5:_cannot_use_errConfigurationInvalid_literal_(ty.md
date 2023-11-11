问题：
client-go@v0.17.0/tools/clientcmd/validation.go:73:5: cannot use errConfigurationInvalid literal (type errConfigurationInvalid) as type "k8s.io/apimachinery/pkg/util/errors".

解决方法：

```bash
require (
	k8s.io/apimachinery v0.17.0
	k8s.io/client-go v0.17.0
)
```

