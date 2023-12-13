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
