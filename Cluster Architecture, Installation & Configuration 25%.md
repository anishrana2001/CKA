# Cluster Architecture, Installation & Configuration 25%
##  1. Manage role based access control (RBAC)
##  2. Use Kubeadm to install a basic cluster
##  3. Manage a highly-available Kubernetes cluster
##  4. Provision underlying infrastructure to deploy a Kubernetes cluster
##  5. Perform a version upgrade on a Kubernetes cluster using Kubeadm
##  6. Implement etcd backup and restore 

###                                                                                                                                                   .
###                                                                                                                                                   .

###  Question 1:  Use context: kubectl config use-context k8s-c1-H
###  You have been asked to create a new ClusterRole for a deployment pipeline and bind it to a specific ServiceAccount scoped to a specific namespace.
###  Task -
###  Create a new ClusterRole named deployment-clusterrole-var, which only allows to create the following resource types:
###  - Deployment
###  - Stateful Set
###  - DaemonSet
###  Create a new ServiceAccount named cicd-token-var in the existing namespace app-team-var.
###  Bind new ClusterRole deployment-clusterrole-var to the new ServiceAccount cicd-token-var, limited to the namespace app-team-var.
###  Solution: What we have given in the question?
###  ClusterRole name = deployment-clusterrole-var
###  ServiceAccount name = cicd-token-var
###  namespace = app-team-var
###  verb= create
###  resources= Deployment,StatefulSet,DaemonSet
###  What we need to create, clusterrole, serviceaccount, and roleBinding.
###   .
###  Solution 
###   Remember : ClusterRole and clusterrolebinding are not a namespaced object. But in question it is asked us "Bind new ClusterRole deployment-clusterrole-var to the new ServiceAccount cicd-token-var, limited to the namespace app-team-var."
 
###  It means that we need to create a ClusterRole and bind this ClusterRole with Rolebinding because rolebinding is a namespaced object.
 
 
###  Change the context
```
kubectl config use-context k8s-c1-H
```

###  Create a ClusterRole which should have verbs "create" and resources must be "Deployment,StatefulSet,DaemonSet"
```
kubectl create clusterrole deployment-clusterrole-var --verb=create --resource=Deployment,StatefulSet,DaemonSet
```
###  Once we created ClusterRole, we can create system serviceaccount under namespace "app-team-var"
```
kubectl create serviceaccount cicd-token-var -n app-team-var
```
###  So far, we created clusterrole and serviceaccount. Now, we can create rolebinding. Please note the serviceaccount syntax "--serviceaccount=namespace:serviceaccount-name" and don't forget to add namespace  because rolebinding is a namespaced object.
```
kubectl create rolebinding deploy-b --clusterrole=deployment-clusterrole-var --serviceaccount=app-team-var:cici-token --namespace=app-team-var 
```
 

###   How we can verify it? 
### Let's check, if system user can create deployment under namespace app-team-var. The answer should be yes.  
```
kubectl auth can-i create  Deployment --as system:serviceaccount:app-team-var:cici-token --namespace=app-team-var
```
### For your references.
```
[root@master1 ~]# kubectl describe clusterrole deployment-clusterrole-var
Name:         deployment-clusterrole-var
Labels:       <none>
Annotations:  <none>
PolicyRule:
  Resources          Non-Resource URLs  Resource Names  Verbs
  ---------          -----------------  --------------  -----
  daemonsets.apps    []                 []              [create]
  deployments.apps   []                 []              [create]
  statefulsets.apps  []                 []              [create]
 
[root@master1 ~]# kubectl -n app-team-var describe rolebindings.rbac.authorization.k8s.io deploy-b
Name:         deploy-b
Labels:       <none>
Annotations:  <none>
Role:
  Kind:  ClusterRole
  Name:  deployment-clusterrole-var
Subjects:
  Kind            Name        Namespace
  ----            ----        ---------
  ServiceAccount  cici-token  app-team-var
[root@master1 ~]#
 
[root@master1 ~]# kubectl auth can-i create  Deployment --as system:serviceaccount:app-team-var:cici-token --namespace=app-team-var
yes

You should see "Yes" in your output.
```

##  Detailed information is being shared on this video : https://youtu.be/_MmrGe1_l3c
 
 

## 5. Perform a version upgrade on a Kubernetes cluster using Kubeadm
###                                                                                                                                                   .
### Question: Given an existing kubernetes cluster running version 1.26.9, upgrade all of the Kubernetes control plan and node components on the master node only to version 1.27.6. You are also expected to upgrade kubelet and kubectl on the master node.


### Open the "Kubernetes.io" page and Click on "Documentation" at the middle of the page and then search "kubeadm upgrade" at the left hand side of the page. Open the first link and click on tab as per your Operating System (OS). My OS is CentOS, so I am following CentOS tab. In exam, it would be Ubuntu or Debian

![image](https://github.com/anishrana2001/CKA/assets/93471182/19fde512-e86b-49fa-b4d3-5b9b11ef8ef3)

### Check the curent version.
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
### Once the kubeadm is upgraded, then we need to upgrade the kubelet and kubectl. We can upgrade both components in a single command.
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

### 🚴
### 🏉
##  6. Implement etcd backup and restore 

### Question : First, create a snapshot of the existing etcd instance running at https://127.0.0.1:2379, saving the snapshot to /var/lib/etcd-snapshot123.db
### After that, you need to restore an existing / previous snapshot located at /var/lib/etcd-snapshot-previous.db.


### solution: 
### First, we need to identify the etcd pods. Below command we can use.
```
kubectl -n kube-system get pod | grep etcd
```
### Now, we need to identify the CA Cert, Cert and Server key. Below command, we can use. 
```
kubectl -n kube-system describe pod etcd-master1.example.com
```
### Once, we have all details, we can take the snapshot. 
```
etcdctl --endpoints=https://127.0.0.1:2379 --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key snapshot save  /var/lib/etcd-snapshot123.db 
```

### We can also verify the new file. 
```
ls -l /opt/etcd-backup.db
```
### How to restore from backup file (/var/lib/from-backup) ?

### Restore the backup in "/var/lib/from-backup" directory. Make sure to use sudo before running command otherwise it will throw permission issue

```
sudo etcdctl snapshot restore --data-dir /var/lib/from-backup  /var/lib/etcd-snapshot-previous.db
```
### Or we can use below command.
```
sudo etcdctl snapshot restore --data-dir /var/lib/from-backup  --endpoints=https://127.0.0.1:2379 --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key /var/lib/etcd-snapshot-previous.db
```
```
sudo chown -R etcd:etcd /var/lib/etcd
```
```
sudo systemctl start etcd
```
 

## All explanation is being done on this video : https://youtu.be/0gkKak8ERQM
