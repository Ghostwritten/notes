Trying with --ignore-preflight-errors=all

ubuntu@ip-172-31-38-142:~$ sudo kubeadm join --token ca3f65.9e6258b8d6109e8f --discovery-token-ca-cert-hash --ignore-preflight-errors=all 172.31.19.230:6443
[preflight] Running pre-flight checks.
[preflight] Some fatal errors occurred:
[ERROR CRI]: unable to check if the container runtime at "/var/run/dockershim.sock" is running: exit status 1
[preflight] If you know what you are doing, you can make a check non-fatal with --ignore-preflight-errors=...

Same results with:

--ignore-preflight-errors=CRI
--ignore-preflight-errors=cri
--ignore-preflight-errors cri
--ignore-preflight-errors CRI

```bash
kubeadm init --ignore-preflight-errors=all
```

