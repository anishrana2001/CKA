## Question 1: 

## Use context: kubectl config use-context k8s-c1-H

## You have been asked to create a new ClusterRole for a deployment pipeline and bind it to a specific ServiceAccount scoped to a specific namespace.

## Task -
## Create a new ClusterRole named deployment-clusterrole-var, which only allows to create the following resource types:
##  - Deployment
##  - Stateful Set
##  - DaemonSet
## Create a new ServiceAccount named cicd-token-var in the existing namespace app-team-var.
## Bind the new ClusterRole deployment-clusterrole-var to the new ServiceAccount cicd-token-var, limited to the namespace app-team-var.

## Solution: What we have given in the question
## ClusterRole = deployment-clusterrole-var
## ServiceAccount = cicd-token-var
## namespace = app-team-var
## What we need to create, clusterrole,serviceaccount, and roleBinding.
## Detailed information is being shared on this video : https://youtu.be/_MmrGe1_l3c
##  Remember : ClusterRole and clusterrolebinding are not a namespaced object.
## But in question it is asked us "Bind new ClusterRole "deployment-clusterrole-var" to the new ServiceAccount cicd-token-var, limited to the namespace app-team-var."
## It means that we need to create a ClusterRole and bind this ClusterRole with Rolebinding because rolebinding is a namespaced object.

### Change the context
```
kubectl config use-context k8s-c1-H
```
### Create a ClusterRole which should have verbs "create" and resources "Deployment,StatefulSet,DaemonSet"
```
kubectl create clusterrole deployment-clusterrole-var --verb=create --resource=Deployment,StatefulSet,DaemonSet

```
### Once we created ClusterRole, we can create system serviceaccount under namespace "app-team-var"
```
kubectl create serviceaccount cicd-token-var -n app-team-var

```
### We have created clusterrole and serviceaccount, and now, we can create rolebinding. Please note the serviceaccount syntax "--serviceaccount=namespace:serviceaccount-name" and don't forget to add namespace
```
kubectl create rolebinding deploy-b --clusterrole=deployment-clusterrole-var --serviceaccount=app-team-var:cici-token --namespace=app-team-var

```
```
kubectl describe rolebindings.rbac.authorization.k8s.io/deploy-b

```
```
kubectl describe clusterrole deployment-clusterrole-var

```
```
kubectl auth can-i create  Deployment --as system:serviceaccount:app-team-var:cici-token --namespace=app-team-var

```


## Question 2: context: ek8s

## Set the node named workernode1.example.com as unavailable and reschedule all the pods running on it.


### Solution: In this question, it is asked indirectly to put this node on maintenance mode. 
### Change the context
```
kubectl config use-context ek8s
```
### Check the context
```
kubectl config current-context
```
### Change the running pods on all nodes. Just check the workernode1, how many pods are working. 
```
kubectl get pods -o wide
```
### Check the node name.
```
kubectl get nodes
```
### Put the node on maintenance mode, cordon is the sub command and the followed by nodename.
```
kubectl cordon workernode1.example.com
```
### Move all the pods to another nodes. 
```
kubectl drain workernode1.example.com --ignore-daemonsets --force
```

### Check all pods moved from workernode1 to another node. In exam node name must be changed.
```
kubectl get pods -o wide
```

## Question 3: First, create a snapshot of the existing etcd instance running at https://127.0.0.1:2379, saving the snapshot to /var/lib/etcd-snapshot.db.
## Next, restore an existing, previous snapshot located at /var/lib/etcd-snapshot-previous.db.

```
kubectl -n kube-system get pod | grep etcd
```
```
kubectl -n kube-system describe pod etcd-master1.example.com

```
```
etcdctl --endpoints=https://127.0.0.1:2379 --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key snapshot save  /var/lib/etcd-snapshot.db

```
```
ls -l /opt/etcd-backup.db

```
```
sudo chown -R etcd:etcd  /var/lib/etcd-snapshot.db
```

```
### How to restore from backup file (/var/lib/from-backup) ?

```
```
etcdctl snapshot restore --data-dir /var/lib/from-backup  /opt/etcd-backup.db 

```

All explanation is being done on this video : https://youtu.be/0gkKak8ERQM
