# Prepare the lab for this question:
```
mkdir -p /data/lab/1/tiger
cd /data

curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
helm completion bash > /etc/bash_completion.d/helm
/usr/local/bin/helm repo add minio https://operator.min.io/
curl -o /data/lab/1/tiger/tenant.yaml https://raw.githubusercontent.com/anishrana2001/CKA/refs/heads/main/2025/helm-tenant.yaml
```

## Please make sure, helm is installed.

#  Question: Solve this question on: ssh cka4628
## Your tasks are:
- 1. You need to create a namespace called `tiger`
- 2. Install Helm chart `minio/operator` in the `tiger` namespace. The Helm Release should be called `minio-prod-operator`
- 3. Update the Tenant resource in `/data/lab/1/tiger/tenant.yaml` to include `enableSFTP: true` under `features`.
- 4. Create the Tenant resource from `/data/lab/1/tiger/tenant.yaml`
--- 
## Solution: 
### helm command package is already installed & `minio repo` will also be added
### Note: 
- Helm chart name = `minio/operator`
- Helm release name = `minio-prod-operator`
- Tenant resource yaml file = `/data/lab/1/tiger/tenant.yaml`

### Let's start.

### - 1. Create namespace `tiger`
```
kubectl create namespace tiger
```
### - 2. Install Helm chart `minio/operator` in the `tiger` namespace. The Helm Release should be called `minio-prod-operator`
### From the above task, we get to know that 
- Helm chart name = `minio/operator`
- Helm release name = `minio-prod-operator`
- we need to use `tiger` namespace 
### Identify the Repo name. You must observe one repo added. In my case repo name is `minio`
```
helm repo list
```

### Check the Chart name. As per the question it must be "minio/operator"
```
helm search repo
```

```
helm -n tiger install minio-prod-operator minio/operator
```

### Post check for task 2 !!
```
helm -n tiger ls
```
```
kubectl get pods -n tiger 
```

### - 3. Update the Tenant resource in `/data/lab/1/tiger/tenant.yaml` to include `enableSFTP: true` under features.
### Open the file "/data/lab/1/tiger/tenant.yaml" and search for `features` and then add one line "enableSFTP: true". Please bear in mind the indentation. 
```
vi /data/lab/1/tiger/tenant.yaml
```

```
spec:
  features:
    bucketDNS: false
    enableSFTP: true                     ## ADD 👈👈👈
  image: quay.io/minio/minio:latest
```
![image](https://github.com/user-attachments/assets/7a4a671e-6919-473b-9248-92511e6bcf0b)


### - 4. Create the Tenant resource from `/data/lab/1/tiger/tenant.yaml`
```
kubectl apply -f /data/lab/1/tiger/tenant.yaml 
```

### Post checks
```
kubectl -n tiger get tenants.minio.min.io 
```






## How to remove the lab ?
- First, we will delete the Tenant.
- Second, we will uninstall the release.
- In last, we will remove the repo and delete the namespace
```
kubectl delete -f  /data/lab/1/tiger/tenant.yaml
rm -rf /data/lab/1/tiger/tenant.yaml 
helm uninstall minio-prod-operator -n tiger 
helm repo remove minio
kubectl -n tiger delete pods/$( kubectl -n tiger get pods | awk '{print $1}' | grep -v NAME) --force --grace-period=0
kubectl delete namespaces tiger
```




### For your references.
```


[root@master1 ~]# kubectl get nodes
NAME                      STATUS   ROLES           AGE    VERSION
master1.example.com       Ready    control-plane   283d   v1.29.15
workernode1.example.com   Ready    <none>          283d   v1.29.15
workernode2.example.com   Ready    <none>          283d   v1.29.15
[root@master1 ~]# mkdir -p /data/lab/1/tiger
[root@master1 data]# /cd /data
[root@master1 data]# /usr/local/bin/helm repo add minio https://operator.min.io/
"minio" has been added to your repositories

[root@master1 data]#
[root@master1 data]# ssh cka4628

[root@master1 data]#
[root@master1 data]# kubectl create namespace tiger
namespace/tiger created

[root@master1 data]# helm repo list
NAME    URL
minio   https://operator.min.io/

[root@master1 data]# helm search repo
NAME                    CHART VERSION   APP VERSION     DESCRIPTION
minio/minio-operator    4.3.7           v4.3.7          A Helm chart for MinIO Operator
minio/operator          7.1.1           v7.1.1          A Helm chart for MinIO Operator
minio/tenant            7.1.1           v7.1.1          A Helm chart for MinIO Operator
[root@master1 data]# helm search repo | grep minio
minio/minio-operator    4.3.7           v4.3.7          A Helm chart for MinIO Operator
minio/operator          7.1.1           v7.1.1          A Helm chart for MinIO Operator
minio/tenant            7.1.1           v7.1.1          A Helm chart for MinIO Operator

[root@master1 data]# helm -n tiger install minio-prod-operator minio/operator
NAME: minio-prod-operator
LAST DEPLOYED: Mon May 26 23:03:50 2025
NAMESPACE: tiger
STATUS: deployed
REVISION: 1
TEST SUITE: None

[root@master1 data]# helm -n tiger ls
NAME                    NAMESPACE       REVISION        UPDATED                                 STATUS     CHART           APP VERSION
minio-prod-operator     tiger           1               2025-05-26 23:03:50.340489177 +0530 IST deployed   operator-7.1.1  v7.1.1
[root@master1 data]#

[root@master1 data]# kubectl get pods -n tiger
NAME                              READY   STATUS    RESTARTS   AGE
minio-operator-766455fdcd-fnkqf   1/1     Running   0          50s
minio-operator-766455fdcd-jqzcx   1/1     Running   0          50s

[root@master1 data]# vi /data/lab/1/tiger/tenant.yaml


[root@master1 data]# kubectl -n tiger get tenants.minio.min.io
No resources found in tiger namespace.

[root@master1 data]# kubectl apply -f /data/lab/1/tiger/tenant.yaml
tenant.minio.min.io/myminio created

[root@master1 data]# kubectl -n tiger get tenants.minio.min.io
NAME      STATE   HEALTH   AGE
myminio                    5s


[root@master1 data]# kubectl delete -f  /data/lab/1/tiger/tenant.yaml
rm -rf /data/lab/1/tiger/tenant.yaml
helm uninstall minio-prod-operator -n tiger
helm repo remove minio
kubectl -n tiger delete pods/$( kubectl -n tiger get pods | awk '{print $1}' | grep -v NAME) --force --grace-period=0
kubectl delete namespaces tiger
tenant.minio.min.io "myminio" deleted
release "minio-prod-operator" uninstalled
"minio" has been removed from your repositories
error: there is no need to specify a resource type as a separate argument when passing arguments in resource/name form (e.g. 'kubectl get resource/<resource_name>' instead of 'kubectl get resource resource/<resource_name>'
namespace "tiger" deleted
[root@master1 data]#


```
