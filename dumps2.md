## Qestion 4: Reconfigure the existing deployment front-end-var and add a port specifiction named http exposing port 80/tcp of the existing container nginx. 
## Create a new service named front-end-var-svc-var exposing the container port http.
## Configure the new service to also expose individual Pods via a NodePort on the nodes on which they are scheduled.



### Solution: 
### What we have deployment name "front-end-var".
### What is asked us, expose this deployment on port 80, protocol TCP. 
### Then, ask to bind the NodePort service "front-end-var-svc-var" to this deployment. 

```
kubectl expose deployment front-end-var --name=front-end-var-svc-var --port=80 --target-port=80 --protocol=TCP --type=NodePort
```
```
[root@master1 ~]# kubectl get service front-end-var-svc-var 
NAME            TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
front-end-var-svc-var   NodePort   10.101.245.122   <none>        80:30521/TCP   12s
[root@master1 ~]# curl http://10.101.245.122:80 | head
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   615  100   615    0     0   600k      0 --:--:-- --:--:-- --:--:--  600k
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
[root@master1 ~]#
```


## Question 5: kubectl config use-context k8s-c1-H
## Scale the deployment web-server-var to 6 pods
### Solution: 
```
kubectl config use-context k8s-c1-H
```
```
kubectl get deployments/web-server-var
```
```
kubectl scale deployment web-server-var --replicas=6
```



## Question 6: use context : k8s-c1-H 

##   Schedule a pod as follows:

## · Name: nginx-kusc00401-var
## · Image: nginx
## · Node selector: disktype=ssd

### Solution: 
```
kubectl config use-context ek8s
```
```
kubectl config current-context
```
```
kubectl run nginx-kusc00401-var --image=nginx --dry-run=client -o yaml > 6-pod.yaml
```
```
vi  6-pod.yaml 
		apiVersion: v1
		kind: Pod
		metadata:
		  creationTimestamp: null
		  labels:
			run: nginx-kusc00401-var
		  name: nginx-kusc00401-var
		spec:
		  containers:
		  - image: nginx
			name: nginx-kusc00401-var
			resources: {}
		  dnsPolicy: ClusterFirst
		  nodeSelector:       ## NodeSelector syntax
			disktype: ssd       ## This pod will schedule only this node
		  restartPolicy: Always
		status: {}
```
```
kubectl create -f 4.pod.yaml
```
```
kubectl get pod
```


## Question 7: use context : k8s-c1-H 

##  - Create a new PersistentVolume named tata-pv. It should have a capacity of 2Gi, accessMode ReadWriteOnce, hostPath /srv/app-config-var.
##  - Next create a new PersistentVolumeClaim in Namespace project-tiger named tata-pvc . It should request 2Gi storage, accessMode ReadWriteOnce and should not define a storageClassName. The PVC should bound to the PV correctly.
##  - Finally create a new pod tata in Namespace project-tiger which mounts that volume at /tmp/tata-data. The Pods should be of image httpd:2.4.41-alpine.

### Solution: In exam, namespace "project-tige" already created. 
### To complete this question, we need to create PV, PVC and POD.
### 1. Create PV:  
### name=tata-pv,  
### Capacity=2Gi, 
### accessMode=ReadWriteOnce, 
### hostPath=/srv/app-config-var , 
### Remember: PV is not namespaced object.
 
### 2. Create PVC: 
### name=tata-pvc, 
### Capacity=2Gi,
### accessMode=ReadWriteOnce, 
### Namespace=project-tiger, 
### should not define a storageClassName.
 
### 3. Create POD: 
### name=tata, 
### namespace=project-tiger, 
### volume=/tmp/tata-data, 
### image=httpd:2.4.41-alpine


### 1. Create PV:  name=tata-pv,  Capacity=2Gi, accessMode=ReadWriteOnce, hostPath=/srv/app-config-var , Remember: PV is not namespaced object.
 
### Open kubernetes.io => Documentation => Search "pv hostPath"

### Copy content like below from the website.

### Copy the below content

<table>
<tr>
<th>Argument</th>
<th>Description</th>
</tr>
<tr>
<td>
apiVersion: v1            
kind: PersistentVolume    
metadata:                 
  name: task-pv-volume    
  labels:                 
    type: local           
spec:                     
  storageClassName: manual
  capacity:               
    storage: 10Gi         
  accessModes:            
    - ReadWriteOnce       
  hostPath:               
    path: "/mnt/data"     </td>
<td>
apiVersion: v1                
kind: PersistentVolume        
metadata:                     
  name: tata-pv               
  labels:                     
    type: local               
spec:                         
  storageClassName: manual    
  capacity:                   
    storage: 2Gi              
  accessModes:                
   - ReadWriteOnce            
  hostPath:                   
   path: "/srv/app-config-var"</td>
</tr>
</table>
