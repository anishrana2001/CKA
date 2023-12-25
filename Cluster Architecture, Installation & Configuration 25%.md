# Cluster Architecture, Installation & Configuration 25%
##   Manage role based access control (RBAC)
##     Use Kubeadm to install a basic cluster
##     Manage a highly-available Kubernetes cluster
##     Provision underlying infrastructure to deploy a Kubernetes cluster
##     Perform a version upgrade on a Kubernetes cluster using Kubeadm
##     Implement etcd backup and restore 

## Perform a version upgrade on a Kubernetes cluster using Kubeadm
### Question: Given an existing kubernetes cluster running version 1.26.9, upgrade all of the Kubernetes control plan and node components on the master node only to version 1.27.6. You are also expected to upgrade kubelet and kubectl on the master node.

### Check the curent version.

### Open the "Kubernetes.io" page and Click on "Documentation" on the left hand side then search "kubeadm upgrade". Open the first link and click on tab as per your Operating System.

![image](https://github.com/anishrana2001/CKA/assets/93471182/19fde512-e86b-49fa-b4d3-5b9b11ef8ef3)

```
[root@master1 ~]# kubectl get nodes
NAME                      STATUS   ROLES           AGE    VERSION
master1.example.com       Ready    control-plane   382d   v1.26.9
workernode1.example.com   Ready    <none>          382d   v1.26.9
workernode2.example.com   Ready    <none>          382d   v1.26.9

```
### First, we will upgrade the kubeadm. Thus, list the version available for us.
```
[root@master1 ~]# yum list --showduplicates kubeadm --disableexcludes=kubernetes
Last metadata expiration check: 0:11:20 ago on Monday 25 December 2023 04:30:00 PM.
Installed Packages
kubeadm.x86_64                           1.26.9-0                            @kubernetes
Available Packages
kubeadm.x86_64                           1.6.0-0                             kubernetes 
kubeadm.x86_64                           1.6.1-0                             kubernetes 
kubeadm.x86_64                           1.6.2-0                             kubernetes
.
.
.
.
.
.
kubeadm.x86_64                           1.26.6-0                            kubernetes 
kubeadm.x86_64                           1.26.7-0                            kubernetes 
kubeadm.x86_64                           1.26.8-0                            kubernetes 
kubeadm.x86_64                           1.26.9-0                            kubernetes 
kubeadm.x86_64                           1.27.0-0                            kubernetes 
kubeadm.x86_64                           1.27.1-0                            kubernetes 
kubeadm.x86_64                           1.27.2-0                            kubernetes 
kubeadm.x86_64                           1.27.3-0                            kubernetes 
kubeadm.x86_64                           1.27.4-0                            kubernetes 
kubeadm.x86_64                           1.27.5-0                            kubernetes 
kubeadm.x86_64                           1.27.6-0                            kubernetes 
kubeadm.x86_64                           1.28.0-0                            kubernetes 
kubeadm.x86_64                           1.28.1-0                            kubernetes 
kubeadm.x86_64                           1.28.2-0                            kubernetes
```
### As per the question, we need to upgrade the components on 1.27.6 version. Hence, install the kubeadm package on the System.
```
[root@master1 ~]# yum install -y kubeadm-1.27.6 --disableexcludes=kubernetes
Last metadata expiration check: 0:20:18 ago on Monday 25 December 2023 04:30:00 PM.
Dependencies resolved.
======================================================================================
 Package            Architecture     Version      Repository       Size
======================================================================================
Downgrading:
 kubeadm           x86_64            1.27.6-0     kubernetes      11 M

Transaction Summary
=======================================================================================
Downgrade  1 Package

Total download size: 11 M
Downloading Packages:
b551bc583f95854dd3f0e3cb8361cc1cef57ada75aef31b709c63f0da37b1fbd-kubeadm-1.27.6-0.x86_64.rpm        1.5 MB/s |  11 MB     00:07    
--------------------------------------------------------------------------------
Total                                                                                               1.5 MB/s |  11 MB     00:07     
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                1/1 
  Downgrading      : kubeadm-1.27.6-0.x86_64                        1/2 
  Cleanup          : kubeadm-1.28.1-0.x86_64                        2/2 
  Running scriptlet: kubeadm-1.28.1-0.x86_64                        2/2 
  Verifying        : kubeadm-1.27.6-0.x86_64                        1/2 
  Verifying        : kubeadm-1.28.1-0.x86_64                        2/2 

Downgraded:
  kubeadm-1.27.6-0.x86_64                
Complete!
[root@master1 ~]# 

```

### We can check the current version.
```
[root@master1 ~]# kubeadm version -o yaml
clientVersion:
  buildDate: "2023-09-13T09:19:54Z"
  compiler: gc
  gitCommit: 741c8db18a52787d734cbe4795f0b4ad860906d6
  gitTreeState: clean
  gitVersion: v1.27.6  >>>>>>>>>>>>>>
  goVersion: go1.20.8
  major: "1"
  minor: "27"
  platform: linux/amd64
```

```
[root@master1 data]# kubeadm upgrade plan
```
### Its time to upgrade the Kubeadm. Chose the version that you want to upgrade.
```
[root@master1 data]# kubeadm upgrade apply v1.27.6
```
### Once the kubeadm is upgraded, then we need to ugprade the kubelet and kubectl. We can upgrade both components in a single command.
```
[root@master1 data]# yum install -y kubelet-1.27.6 kubectl-1.27.6 --disableexcludes=kubernetes
```
### After the upgrade of kubelet and kubectl services. All steps are already given in the Kubernetes.io web page. 
```
[root@master1 data]# sudo systemctl daemon-reload
[root@master1 data]# sudo systemctl restart kubelet
```

### It take some time to update the version. Thus, wait for 2 minutes then execute the below command. 
```
[root@master1 ~]#  kubectl get nodes
NAME                      STATUS   ROLES           AGE    VERSION
master1.example.com       Ready    control-plane   382d   v1.27.6  >>>>>>>>> Its showing new version.
workernode1.example.com   Ready    <none>          382d   v1.26.9
workernode2.example.com   Ready    <none>          382d   v1.26.9
```
