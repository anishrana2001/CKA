## Qestion 4: Reconfigure the existing deployment front-end-var and add a port specifiction named http exposing port 80/tcp of the existing container nginx. 
## Create a new service named front-end-var-svc-var exposing the container port http.
## Configure the new service to also expose individual Pods via a NodePort on the nodes on which they are scheduled.



### Solution: 
### What we have, deployment name "front-end-var", whcih is already running.
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
### If you want to understand in deep then you may watch my "What is service in Kubernetes" video on Youtube.
[What is service in Kubernete](https://youtu.be/4AFLbbfr9UM)

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
 
### Open kubernetes.io => Documentation => Search "pv hostPath". Open the first link and scron down.

### Copy content like below from the website.

### Copy the below content

```yaml
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
    path: "/mnt/data"     
```

### Modify the content as per our question, like below

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: tata-pv
spec:
  capacity:
    storage: 2Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/srv/app-config-var"
```
### create one new file and paste the above yaml file, like below.
```
cat <<EOF>> question7-pv.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: tata-pv
spec:
  capacity:
    storage: 2Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/srv/app-config-var"
EOF
```

### 2. Create PVC: name=tata-pvc, Capacity=2Gi, accessMode=ReadWriteOnce, Namespace=project-tiger, should not define a storageClassName.
 
### Open the kubernetes.io => Documentation => Search "pvc accessModes"
###  Open the first link and Copy the below content

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: restore-pvc
spec:
  storageClassName: csi-hostpath-sc
  dataSource:
    name: new-snapshot-test
    kind: VolumeSnapshot
    apiGroup: snapshot.storage.k8s.io
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
```
### Modify the content as per our question, like below
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: tata-pvc
  namespace: project-tiger
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 2Gi
```

### create one new file and paste the above yaml file, like below.
```
cat <<EOF>> question7-pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: tata-pvc
  namespace: project-tiger
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 2Gi
EOF
```

### 3. Create pod: name=tata, namespace=project-tiger, volume=/tmp/tata-data, image=httpd:2.4.41-alpine
 
###  kubernetes.io => Documentation => Search "pod pvc"
 
 

### Copy the below content
	

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: task-pv-pod
spec:
  volumes:
    - name: task-pv-storage
      persistentVolumeClaim:
        claimName: task-pv-claim
  containers:
    - name: task-pv-container
      image: nginx
      ports:
        - containerPort: 80
          name: "http-server"
      volumeMounts:
        - mountPath: "/usr/share/nginx/html"
          name: task-pv-storage
```

### Modify the content as per our question, like below
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: tata
  namespace: project-tiger
spec:
  volumes:
    - name: data
      persistentVolumeClaim:
        claimName: tata-pvc
  containers:
    - name: task-pv-container
      image: httpd:2.4.41-alpine
      volumeMounts:
        - mountPath: "/tmp/tata-data"
          name: data
```

### create one new file and paste the above yaml file, like below.
```
cat <<EOF>> question7-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: tata
  namespace: project-tiger
spec:
  volumes:
    - name: data
      persistentVolumeClaim:
        claimName: tata-pvc
  containers:
    - name: task-pv-container
      image: httpd:2.4.41-alpine
      volumeMounts:
        - mountPath: "/tmp/tata-data"
          name: data
EOF
```
### Now, its time to create PV, PVC and POD from the yaml files.
```
kubectl apply -f  question7-pv.yaml
kubectl apply -f  question7-pvc.yaml
kubectl apply -f  question7-pod.yaml
```

### If everything is OK, then our pod must be in running state.
```
[root@master1 ~]# kubectl -n project-tiger get pods
NAME   READY   STATUS    RESTARTS   AGE
tata   1/1     Running   0          50s
```
### We can also check if directory is correctly mounted on our POD
```
[root@master1 ~]# kubectl -n project-tiger exec -it tata -- /bin/bash

bash-5.0# df -h
Filesystem                Size      Used Available Use% Mounted on
overlay                  14.0G      8.3G      5.6G  60% /
tmpfs                    64.0M         0     64.0M   0% /dev
/dev/sda2                14.0G      8.3G      5.6G  60% /tmp/tata-data  >>>>>>
/dev/sda2                14.0G      8.3G      5.6G  60% /etc/hosts
/dev/sda2                14.0G      8.3G      5.6G  60% /dev/termination-log
/dev/sda2                14.0G      8.3G      5.6G  60% /etc/hostname
/dev/sda2                14.0G      8.3G      5.6G  60% /etc/resolv.conf
shm                      64.0M         0     64.0M   0% /dev/shm
tmpfs                     2.1G     12.0K      2.1G   0% /run/secrets/kubernetes.io/serviceaccount
tmpfs                     1.1G         0      1.1G   0% /proc/acpi
tmpfs                    64.0M         0     64.0M   0% /proc/kcore
tmpfs                    64.0M         0     64.0M   0% /proc/keys
tmpfs                    64.0M         0     64.0M   0% /proc/timer_list
tmpfs                     1.1G         0      1.1G   0% /proc/scsi
tmpfs                     1.1G         0      1.1G   0% /sys/firmware
bash-5.0#
 
bash-5.0# ls -ltr /tmp/tata-data
total 0
```
### ### If you want to deep dive on Volume topic then you may watch my "What is Volume in Kubernetes" video on Youtube.
[What is Volume in Kubernetes](https://youtu.be/VIlrF9DoTIM)



## Question 8: Create a persistent volume with name app-motor, of capacity 2Gi and access mode ReadWriteMany. The type of volume is hostPath location is /srv/app-tata

### Solution: kubectl config use-context ek8s
### Open URL : https://kubernetes.io --> Documentation --> Search => PV Accessmode ==> Open frist link and search for "accessmode"
### Copy the content link below:

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv0003
spec:
  capacity:
    storage: 5Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Recycle
  storageClassName: slow
  mountOptions:
    - hard
    - nfsvers=4.1
  nfs:
    path: /tmp
    server: 172.17.0.2
```

### Now, modify the yaml file as per the question
```yaml
cat <<EOF>> question8-pv.yaml 
apiVersion: v1
kind: PersistentVolume
metadata:
  name: app-motor
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 2Gi
  accessModes:
    - ReadWriteMany
  hostPath:
    path: "/srv/app-tata"
EOF
```

```
kubectl apply -f question8-pv.yaml 
```
### [root@master1 ~]# kubectl apply -f question8-pv.yaml 
### persistentvolume/app-motor created


```
kubectl get pv/app-motor 
```

#### [root@master1 ~]# kubectl get pv/app-motor 
#### NAME        CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE
#### app-motor   2Gi        RWX            Retain           Available           manual                  14m

```
kubectl describe pv/app-motor 
```
```
For the reference purpose:
[root@master1 ~]# kubectl describe pv/app-motor 
Name:            app-motor
Labels:          type=local
Annotations:     <none>
Finalizers:      [kubernetes.io/pv-protection]
StorageClass:    manual
Status:          Available
Claim:           
Reclaim Policy:  Retain
Access Modes:    RWX
VolumeMode:      Filesystem
Capacity:        2Gi
Node Affinity:   <none>
Message:         
Source:
    Type:          HostPath (bare host directory volume)
    Path:          /srv/app-tata
    HostPathType:  
Events:            <none>
[root@master1 ~]# 
```



## If you want to learn hostPath volume then you may watch my Youtube video.
[What is hostPath Volume in Kubernetes](https://youtu.be/oRDBzoorybI)

