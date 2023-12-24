 
# Workloads & Scheduling 15%

### - Understand deployments and how to perform rolling update and rollbacks
### - Use ConfigMaps and Secrets to configure applications
### - Know how to scale applications
### - Understand the primitives used to create robust, self-healing, application deployments
### - Understand how resource limits can affect Pod scheduling
### - Awareness of manifest management and common templating tools
     
     
     
###     1. Understand deployments and how to perform rolling update and rollbacks
     
##     Question: Create a new deployment called web-prod-268, with image nginx:1.16 and 1 replica. Next upgrade the deployment to version 1.17 using rolling update.
##     Make sure that the version upgrade is recorded in the resource annotation. 
##     Use context: kubectl config use-context k8s-c1-H

## Solution: First, we need to create a deployment "web-prod-268" with image "nginx:1.16" and must have replicaa =1
```
kubectl create deployment web-prod-268 --image=nginx:1.16
```

## We can check the rollout history of this deployment.
```
kubectl rollout history deployment web-prod-268
```

## For reference purpose:
```
[root@master1 ~]# kubectl rollout history deployment web-prod-268 
deployment.apps/web-prod-268 
REVISION  CHANGE-CAUSE
1         <none>
```

## In question, it is asked us to upgrade the deployment and use new image "1.17". For  upgrade we can use "set image" sub-command. For record the upgrade, we can use option "--record"
## Syntax of command would be "kubectl set image deployment deployment_name containername=newimagename --record"
## How to find the container name? , we can use describe command. 

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

## It's time for post checks. Again, execute the rollout history command to see the output.
```
kubectl rollout history deployment web-prod-268
```
## For reference purpose:
```
[root@master1 ~]# kubectl rollout history deployment web-prod-268 
deployment.apps/web-prod-268 
REVISION  CHANGE-CAUSE
1         <none>
2         kubectl set image deployment web-prod-268 nginx=nginx:1.17 --record=true
```

## Also, we should check the image version.

```
kubectl describe deployments.apps web-prod-268 | grep -i "Image:"
```

## For reference purpose:
```
[root@master1 ~]# kubectl describe deployments.apps web-prod-268 | grep -i "Image:"
    Image:        nginx:1.17
[root@master1 ~]#
```



