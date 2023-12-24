 
Workloads & Scheduling 15%

### Understand deployments and how to perform rolling update and rollbacks
### Use ConfigMaps and Secrets to configure applications
### Know how to scale applications
### Understand the primitives used to create robust, self-healing, application deployments
### Understand how resource limits can affect Pod scheduling
### Awareness of manifest management and common templating tools
     
     
     
###     1. Understand deployments and how to perform rolling update and rollbacks
     
###     Question: Create a new deployment called web-prod-268, with image nginx:1.16 and 1 replica. Next upgrade the deployment to version 1.17 using rolling update.
###     Make sure that the version upgrade is recorded in the resource annotation. 
###     Use context: kubectl config use-context k8s-c1-H

### Solution: First, we need to create a deployment "web-prod-268" with image "nginx:1.16" and must have replicaa =1
```
kubectl create deployment web-prod-268 --image=nginx:1.16
```

