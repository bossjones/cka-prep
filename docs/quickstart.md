# Quickstart

```
mkdir ~/kodekloud
cd ~/kodekloud
git clone https://github.com/kodekloudhub/certified-kubernetes-administrator-course.git
cd certified-kubernetes-administrator-course/apple-silicon

./deploy-virtual-machines.sh
```


### expand disks
```
multipass stop kubemaster
multipass stop kubeworker01
multipass stop kubeworker02
multipass set local.kubemaster.disk=30G
multipass set local.kubeworker01.disk=30G
multipass set local.kubeworker02.disk=30G

multipass start kubemaster
multipass start kubeworker01
multipass start kubeworker02
```
