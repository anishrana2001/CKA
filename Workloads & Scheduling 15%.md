 
# Workloads & Scheduling 15%

## 1. Understand deployments and how to perform rolling update and rollbacks
## 2. Use ConfigMaps and Secrets to configure applications
## 3. Know how to scale applications
## 4. Understand the primitives used to create robust, self-healing, application deployments
## 5. Understand how resource limits can affect Pod scheduling
## 6. Awareness of manifest management and common templating tools
##
##
##
##
## 1. Understand deployments and how to perform rolling update and rollbacks
     
### Question: Create a new deployment called web-prod-268, with image nginx:1.16 and 1 replica. Next upgrade the deployment to version 1.17 using rolling update.
### Make sure that the version upgrade is recorded in the resource annotation. 
### Use context: kubectl config use-context k8s-c1-H

### Solution: First, we need to create a deployment "web-prod-268" with image "nginx:1.16" and must have replicaa =1
```
kubectl create deployment web-prod-268 --image=nginx:1.16
```

### We can check the rollout history of this deployment.
```
kubectl rollout history deployment web-prod-268
```

### For reference purpose:
```
[root@master1 ~]# kubectl rollout history deployment web-prod-268 
deployment.apps/web-prod-268 
REVISION  CHANGE-CAUSE
1         <none>
```

### In question, it is asked us to upgrade the deployment and use new image "1.17". For  upgrade we can use "set image" sub-command. For record the upgrade, we can use option "--record"
### Syntax of command would be "kubectl set image deployment deployment_name containername=newimagename --record"
### How to find the container name? , we can use describe command. 

```
[root@master1 ~]# kubectl get deployments.apps web-prod-268 -o yaml | grep -A 10 container 
      containers:
      - image: nginx:1.17
        imagePullPolicy: IfNotPresent
        name: nginx    >>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
[root@master1 ~]#
```

```
kubectl set image deployment web-prod-268 nginx=nginx:1.17 --record
```

### It's time for post checks. Again, execute the rollout history command to see the output.
```
kubectl rollout history deployment web-prod-268
```
### For reference purpose:
```
[root@master1 ~]# kubectl rollout history deployment web-prod-268 
deployment.apps/web-prod-268 
REVISION  CHANGE-CAUSE
1         <none>
2         kubectl set image deployment web-prod-268 nginx=nginx:1.17 --record=true
```

### Also, we should check the image version.

```
kubectl describe deployments.apps web-prod-268 | grep -i "Image:"
```

### For reference purpose:
```
[root@master1 ~]# kubectl describe deployments.apps web-prod-268 | grep -i "Image:"
    Image:        nginx:1.17
[root@master1 ~]#
```
### Congratulation!! You have completed the question successfully.


## 3. Know how to scale applications

### Question: Scale the deployment web-app to 6 pods
### kubectl config use-context k8s-c1-H

### Solution: First, we should check where this Deployment is running. In the question, no namespace is defined. Thus, this deployment must be running on default namespace under user-context k8s-c1-H.

```
kubectl config use-context k8s-c1-H
```
### Check the deployment and identify how many replicas are defined, it means how many pods are there.
```
kubectl get deployments.apps web-app
```

### For references, my deployment has only 2 replicas.
```
[root@master1 ~]# kubectl get deployments.apps web-app 
NAME      READY   UP-TO-DATE   AVAILABLE   AGE
web-app   2/2     2            2           54s
```

Now, we can update the existing deployment "web-app" to replicas 6
```
kubectl scale deployment web-app --replicas=6
```
### For references
```
[root@master1 ~]# kubectl get deployments.apps web-app 
NAME      READY   UP-TO-DATE   AVAILABLE   AGE
web-app   6/6     6            6           2m54s

Or you can use get sub-command.

[root@master1 ~]# kubectl get deployments.apps web-app -o yaml | grep -i replicas
  replicas: 6    >>>>>>>>>>>>>>>>>>>>
  availableReplicas: 6
    message: ReplicaSet "web-app-66f7dd595" has successfully progressed.
    reason: NewReplicaSetAvailable
    reason: MinimumReplicasAvailable
  readyReplicas: 6
  replicas: 6
  updatedReplicas: 6
[root@master1 ~]# 
```

### Congratulation!! You have completed the question successfully.


## 4. Understand the primitives used to create robust, self-healing, application deployments



## 6. Awareness of manifest management and common templating tools

### Use context: kubectl config use-context k8s-c1-H

### There are two Pods named oodb-* in Namespace project-lab. Lab management asked you to scale the Pods down to one replica to save resources.

### Solution: First, we need to locate the pods under namespace "project-lab". 

### Select the correct context.

```
kubectl config use-context k8s-c1-H
```
```
kubectl get pods -n project-lab | grep oo
```

### For reference purpose only.
```
[root@master1 ~]# kubectl get pods -n project-lab| grep oodb
oodb-0   1/1     Running   0          12s
oodb-1   1/1     Running   0          11s
oodb-2   1/1     Running   0          9s
```

### From the above output, it is not clear that these pods are belongs to which deployment / statefulset / daemonset. This is the reason, we can search on deployment, statefulset and daemonset. See the reference for more options.

```
kubectl get deployments.apps,statefulsets.apps,ds -n project-lab 
```

### For reference purpose only.
```
[root@master1 ~]# kubectl get deployments.apps,statefulsets.apps,ds -n project-lab 
NAME                    READY   AGE
statefulset.apps/oodb   3/3     2m23s
[root@master1 ~]# 

OR You can check from POD labels too.

[root@master1 ~]# kubectl get pods -n project-lab --show-labels  | grep oo
oodb-0   1/1     Running   0          64s   app=nginx,controller-revision-hash=oodb-578cfc4b46,statefulset.kubernetes.io/pod-name=oodb-0
oodb-1   1/1     Running   0          63s   app=nginx,controller-revision-hash=oodb-578cfc4b46,statefulset.kubernetes.io/pod-name=oodb-1
oodb-2   1/1     Running   0          61s   app=nginx,controller-revision-hash=oodb-578cfc4b46,statefulset.kubernetes.io/pod-name=oodb-2

OR you can use "get all"

[root@master1 ~]# kubectl get all -n project-lab 
NAME         READY   STATUS    RESTARTS   AGE
pod/oodb-0   1/1     Running   0          8s
pod/oodb-1   1/1     Running   0          6s
pod/oodb-2   1/1     Running   0          5s

NAME                    READY   AGE
statefulset.apps/oodb   3/3     8s

```

### Now, it is clear that these pods are belongs to statefulsets. To fulfil the task we simply run:

```
kubectl -n project-lab scale statefulset oodb --replicas=1
```

### For reference purpose only.
```
[root@master1 ~]# kubectl -n project-lab scale statefulset oodb --replicas=1
statefulset.apps/oodb scaled
```

### Post checks / How to verify it?

```
kubectl get deployments.apps,statefulsets.apps,ds -n project-lab 
```
### For reference purpose only.
[root@master1 ~]# kubectl get deployments.apps,statefulsets.apps,ds -n project-lab 
NAME                    READY   AGE
statefulset.apps/oodb   1/1     3m46s
[root@master1 ~]# 
### Congratulation!! You have completed the question successfully.
