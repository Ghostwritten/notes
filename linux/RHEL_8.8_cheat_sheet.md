# RHEL_8.8_cheat_sheet

## install package

```bash
yum -y install vim wget zip unzip bash-completion
```

## install ansible

```bash
sudo subscription-manager register
sudo subscription-manager repos --enable ansible-2.9-for-rhel-8-x86_64-rpms
sudo yum install ansible
```
