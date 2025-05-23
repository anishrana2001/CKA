
# Prepare the lab for this question:
```
mkdir -p /data/lab/1/minio
cd /data

curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
helm completion bash > /etc/bash_completion.d/helm
/usr/local/bin/helm repo add minio https://operator.min.io/
curl -o /data/lab/1/minio/tenant.yaml https://raw.githubusercontent.com/anishrana2001/CKA/refs/heads/main/2025/helm-tenant.yaml
```

## Please make sure, helm is installed.

#  Question: Solve this question on: ssh cka4628
## Your tasks are:
- 1. You need to create a namespace called `minio`
- 2. Install Helm chart `minio/operator` in the `minio` namespace. The Helm Release should be called `minio-operator`
- 3. Update the Tenant resource in `/data/lab/1/minio/tenant.yaml` to include `enableSFTP: true` under `features`.
- 4. Create the Tenant resource from `/data/lab/1/minio/tenant.yaml`
--- 
## Solution: 
### helm package is already installed & helm chart "minio/operator" is also installed.
### Note: 
- Helm chart name = `minio/operator`
- Helm release name = `minio-operator`
- Tenant resource yaml file = `/data/lab/1/minio/tenant.yaml`

### Let's start.

### - 1. Create namespace `minio`
```
kubectl create namespace minio
```
### - 2. Install Helm chart `minio/operator` in the `minio` namespace. The Helm Release should be called `minio-operator`
### From the above task, we get to know that 
- Helm chart name = `minio/operator`
- Helm release name = `minio-operator`
- we need to use `minio` namespace 
### Identify the Repo name. You must observe one repo added. In my case repo name is `minio`
```
helm repo list
```

### Check the Chart name. As per the question it must be "minio/operator"
```
helm search repo
```

```
helm -n minio install minio-operator minio/operator
```

### Post check for task 2 !!
```
helm -n minio ls
```
```
kubectl get pods -n minio 
```

### - 3. Update the Tenant resource in `/data/lab/1/minio/tenant.yaml` to include `enableSFTP: true` under features.
### Open the file "/data/lab/1/minio/tenant.yaml" and search for `features` and then add one line "enableSFTP: true". Please bear in mind the indentation. 
```
vi /data/lab/1/minio/tenant.yaml
```

```
spec:
  features:
    bucketDNS: false
    enableSFTP: true                     ## ADD 👈👈👈
  image: quay.io/minio/minio:latest
```
![image](https://github.com/user-attachments/assets/7a4a671e-6919-473b-9248-92511e6bcf0b)


### - 4. Create the Tenant resource from `/data/lab/1/minio/tenant.yaml`
```
kubectl apply -f /data/lab/1/minio/tenant.yaml 
```

### Post checks
```
kubectl -n minio get tenants.minio.min.io 
```






## How to remove the lab ?
- First, we will delete the Tenant.
- Second, we will uninstall the release.
- In last, we will remove the repo
```
kubectl delete -f  /data/lab/1/minio/tenant.yaml
rm -rf /data/lab/1/minio/tenant.yaml 
helm uninstall minio-operator -n minio 
helm repo remove minio
```




### For your references.
```

[root@master1 ~]# helm repo list
NAME    URL                     
minio   https://operator.min.io/

[root@master1 ~]# helm search repo
NAME                    CHART VERSION   APP VERSION     DESCRIPTION                    
minio/minio-operator    4.3.7           v4.3.7          A Helm chart for MinIO Operator
minio/operator          7.1.1           v7.1.1          A Helm chart for MinIO Operator
minio/tenant            7.1.1           v7.1.1          A Helm chart for MinIO Operator

[root@master1 ~]# helm -n minio install minio-operator minio/operator
NAME: minio-operator
LAST DEPLOYED: Fri May 23 11:28:36 2025
NAMESPACE: minio
STATUS: deployed
REVISION: 1
TEST SUITE: None

[root@master1 ~]# helm -n minio ls
NAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART           APP VERSION
minio-operator  minio           1               2025-05-23 11:28:36.238589736 +0530 IST deployed        operator-7.1.1  v7.1.1     

[root@master1 ~]# kubectl get pods -n minio 
NAME                              READY   STATUS         RESTARTS   AGE
minio-operator-67677d7db8-dtkjs   1/1     Running        0          28s
minio-operator-67677d7db8-fv9nm   0/1     ErrImagePull   0          28s
[root@master1 ~]# 

[root@master1 ~]# vi /data/lab/1/minio/tenant.yaml
[root@master1 ~]# kubectl apply -f /data/lab/1/minio/tenant.yaml 
tenant.minio.min.io/myminio created

[root@master1 ~]# kubectl -n minio get tenants.minio.min.io 
NAME      STATE   HEALTH   AGE
myminio                    14s
[root@master1 ~]# 


[root@master1 data]# kubectl delete -f  https://raw.githubusercontent.com/anishrana2001/CKA/refs/heads/main/2025/helm-tenant.yaml
tenant.minio.min.io "myminio" deleted
[root@master1 data]#
[root@master1 data]# kubectl -n minio get tenants.minio.min.io 
error: the server doesn't have a resource type "tenants"

[root@master1 ~]# helm -n minio ls
NAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART           APP VERSION
minio-operator  minio           1               2025-05-23 11:28:36.238589736 +0530 IST deployed        operator-7.1.1  v7.1.1     
[root@master1 ~]# helm uninstall minio-operator -n minio 
release "minio-operator" uninstalled
[root@master1 ~]# helm -n minio ls
NAME    NAMESPACE       REVISION        UPDATED STATUS  CHART   APP VERSION
[root@master1 ~]# 


[root@master1 ~]# helm search repo 
NAME                    CHART VERSION   APP VERSION     DESCRIPTION                    
minio/minio-operator    4.3.7           v4.3.7          A Helm chart for MinIO Operator
minio/operator          7.1.1           v7.1.1          A Helm chart for MinIO Operator
minio/tenant            7.1.1           v7.1.1          A Helm chart for MinIO Operator
[root@master1 ~]# helm repo list 
NAME    URL                     
minio   https://operator.min.io/

[root@master1 ~]# helm repo remove minio
"minio" has been removed from your repositories
[root@master1 ~]# helm repo list 
Error: no repositories to show
[root@master1 ~]# helm search repo 
Error: no repositories configured
[root@master1 ~]# 
```
