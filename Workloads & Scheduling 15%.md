 
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

## 5. Understand how resource limits can affect Pod scheduling

### Question: Use context: kubectl config use-context k8s-c1-s
### Schedule a pod as follows:
###    · Name: nginx-kusc00401
###    · Image: nginx
###    · Node selector: disktype=ssd
### Solution: In this question, it is asked us to use nodeselector parameter.

### Use the correct context.
```
kubectl config use-context k8s-c1-s
```

### Open the URL : https://kubernetes.io
### Click on Documentation
### Search "nodeSelector disktype"  This is what given in the question. See the below print screen.

![image](https://github.com/anishrana2001/CKA/assets/93471182/afcd9177-deb2-4536-831f-7bc78acbd37a)

copy the below yaml into one file disktype.yaml
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-kusc00401              #updated
  labels:
    env: test
spec:
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent
  nodeSelector:
    disktype: ssd
```
```
kubectl apply -f disktype.yaml
```

### Post checks / How to verify it ? / Pod must be in running state.
```
kubectl get pods nginx-kusc00401 -o wide
```

### For your references.
```
First check the label of nodes
[root@master1 ~]# kubectl describe nodes workernode1.example.com | grep -i -A 2 label
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
                    disktype=ssd

[root@master1 ~]# kubectl apply -f disktype.yaml 
pod/nginx-kusc00401 created

[root@master1 ~]# kubectl get pods nginx-kusc00401 -o wide
NAME              READY   STATUS    RESTARTS   AGE   IP               NODE                      NOMINATED NODE   READINESS GATES
nginx-kusc00401   1/1     Running   0          26s   172.16.133.157   workernode1.example.com   <none>           <none>
[root@master1 ~]#

------------------------------------------------------------------------------------
How to add the lable on nodes.
[root@master1 ~]# kubectl label nodes workernode1.example.com disktype=ssd
node/workernode1.example.com labeled

How to check the lables.
[root@master1 ~]# kubectl describe nodes workernode1.example.com | grep -i -A 6 label
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
                    disktype=ssd

How to remove the lables.
[root@master1 ~]# kubectl label nodes workernode1.example.com disktype-
node/workernode1.example.com unlabeled
[root@master1 ~]# 
---------------------------------------------------------------------------------------
```








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

### Post checks / How to verify it? From the output, pod must be 1

```
kubectl get deployments.apps,statefulsets.apps,ds -n project-lab 
```
### For reference purpose only.
```
[root@master1 ~]# kubectl get deployments.apps,statefulsets.apps,ds -n project-lab 
NAME                    READY   AGE
statefulset.apps/oodb   1/1     3m46s
[root@master1 ~]# 
```
### Congratulation!! You have completed the question successfully.

## 
## 
## 

### Qeustion:  Use context: kubectl config use-context k8s-c1-t
### There are numbers of pods in all namespaces. Write a command into /var/log/find_pods_age.sh which lists all Pods sorted by their AGE (metadata.creationTimestamp).
### Write a second command into /var/log/find_pods_uid.sh which lists all Pods sorted by field metadata.uid. Use kubectl sorting for both commands.
### Solution: If you not remember the command then you may refer the kubernetes.io website and search for cheetsheet.

### use the correct context.
```
kubectl config use-context k8s-c1-t
```
### For part 1.
### Most probably, in exam the sort option "metadata.creationTimestamp" must be given. You just need to use "--sort-by". Or you can type "--sort" and then press the tab button twice.
```
kubectl get pod -A --sort-by=.metadata.creationTimestamp
```

### If the above command executed well then save the command in one file "/var/log/find_pods_age.sh". Please bear in mind that in the question it is mentioned that you need to write a command in the file. Also, try to execute this file, you should see the same output.

```
echo "kubectl get pod -A --sort-by=.metadata.creationTimestamp" > /var/log/find_pods_age.sh
```
```
sh /var/log/find_pods_age.sh
```

### For part 2.

### One can sort the pods list from UID. 
```
kubectl get pod -A --sort-by=.metadata.uid
```
### As per the demand of question, we need to write this command in one file "/var/log/find_pods_uid.sh"
```
echo "kubectl get pod -A --sort-by=.metadata.uid" > /var/log/find_pods_uid.sh
```


### For your references.
```
For part 1.
[root@master1 ~]# kubectl get pod -A --sort-by=.metadata.creationTimestamp
NAMESPACE     NAME                                          READY   STATUS    RESTARTS       AGE
kube-system   calico-node-672fx                             1/1     Running   37 (10h ago)   381d
kube-system   calico-node-tszjn                             1/1     Running   51 (10h ago)   381d
kube-system   calico-node-c5kn4                             1/1     Running   53 (10h ago)   381d
kube-system   kube-proxy-dwr6t                              1/1     Running   5 (10h ago)    29d
kube-system   kube-proxy-g8mq4                              1/1     Running   5 (10h ago)    29d
kube-system   kube-proxy-q9gtm                              1/1     Running   5 (10h ago)    29d
kube-system   kube-scheduler-master1.example.com            1/1     Running   10 (10h ago)   29d
kube-system   kube-controller-manager-master1.example.com   1/1     Running   12 (10h ago)   29d
kube-system   etcd-master1.example.com                      1/1     Running   6 (10h ago)    29d
kube-system   kube-apiserver-master1.example.com            1/1     Running   7 (10h ago)    29d
kube-system   coredns-787d4945fb-rqm58                      1/1     Running   5 (10h ago)    29d
kube-system   coredns-787d4945fb-mtjv5                      1/1     Running   5 (10h ago)    29d
kube-system   calico-kube-controllers-798cc86c47-2bcq4      1/1     Running   5 (10h ago)    29d
default       cpu-pod1                                      1/1     Running   1 (10h ago)    21h
default       cpu-pod2                                      1/1     Running   1 (10h ago)    21h

[root@master1 ~]# 
[root@master1 ~]# echo "kubectl get pod -A --sort-by=.metadata.creationTimestamp" > /var/log/find_pods_age.sh
[root@master1 ~]# sh /var/log/find_pods_age.sh
NAMESPACE     NAME                                          READY   STATUS    RESTARTS       AGE
kube-system   calico-node-672fx                             1/1     Running   37 (10h ago)   381d
kube-system   calico-node-tszjn                             1/1     Running   51 (10h ago)   381d
kube-system   calico-node-c5kn4                             1/1     Running   53 (10h ago)   381d
kube-system   kube-proxy-dwr6t                              1/1     Running   5 (10h ago)    29d
kube-system   kube-proxy-g8mq4                              1/1     Running   5 (10h ago)    29d
kube-system   kube-proxy-q9gtm                              1/1     Running   5 (10h ago)    29d
kube-system   kube-scheduler-master1.example.com            1/1     Running   10 (10h ago)   29d
kube-system   kube-controller-manager-master1.example.com   1/1     Running   12 (10h ago)   29d
kube-system   etcd-master1.example.com                      1/1     Running   6 (10h ago)    29d
kube-system   kube-apiserver-master1.example.com            1/1     Running   7 (10h ago)    29d
kube-system   coredns-787d4945fb-rqm58                      1/1     Running   5 (10h ago)    29d
kube-system   coredns-787d4945fb-mtjv5                      1/1     Running   5 (10h ago)    29d
kube-system   calico-kube-controllers-798cc86c47-2bcq4      1/1     Running   5 (10h ago)    29d
default       cpu-pod1                                      1/1     Running   1 (10h ago)    22h
default       cpu-pod2                                      1/1     Running   1 (10h ago)    22h
[root@master1 ~]# 


For part 2nd.

[root@master1 ~]# kubectl get pod -A --sort-by=.metadata.uid
NAMESPACE     NAME                                          READY   STATUS    RESTARTS       AGE
kube-system   kube-proxy-q9gtm                              1/1     Running   5 (11h ago)    29d
default       web-app-66f7dd595-pm6vh                       1/1     Running   0              5h7m
default       web-app-66f7dd595-xlmvj                       1/1     Running   0              5h7m
default       web-app-66f7dd595-wc44q                       1/1     Running   0              5h7m
kube-system   coredns-787d4945fb-mtjv5                      1/1     Running   5 (11h ago)    29d
default       cpu-pod2                                      1/1     Running   1 (11h ago)    22h
kube-system   kube-controller-manager-master1.example.com   1/1     Running   12 (11h ago)   29d
default       web-app-66f7dd595-5b88b                       1/1     Running   0              5h8m
default       web-prod-268-645746b847-74572                 1/1     Running   0              6h57m
kube-system   etcd-master1.example.com                      1/1     Running   6 (11h ago)    29d
project-lab   oodb-2                                        1/1     Running   0              39m
kube-system   kube-proxy-dwr6t                              1/1     Running   5 (11h ago)    29d
project-lab   oodb-0                                        1/1     Running   0              39m
kube-system   kube-scheduler-master1.example.com            1/1     Running   10 (10h ago)   29d
kube-system   kube-apiserver-master1.example.com            1/1     Running   7 (11h ago)    29d
kube-system   calico-node-672fx                             1/1     Running   37 (11h ago)   381d
kube-system   metrics-server-65bd7dd65d-gznz4               0/1     Pending   0              21h
project-lab   oodb-1                                        1/1     Running   0              39m
kube-system   calico-node-c5kn4                             1/1     Running   53 (11h ago)   381d
default       max-pod1                                      1/1     Running   1 (11h ago)    22h
kube-system   metrics-server-6775944cc6-zwggs               1/1     Running   4 (10h ago)    19h
kube-system   coredns-787d4945fb-rqm58                      1/1     Running   5 (11h ago)    29d
kube-system   metrics-server-65bd7dd65d-qt2gr               0/1     Running   1 (11h ago)    21h
kube-system   calico-node-tszjn                             1/1     Running   51 (10h ago)   381d
default       web-app-66f7dd595-c6q4z                       1/1     Running   0              5h8m
default       web-app-66f7dd595-cb8mn                       1/1     Running   0              5h7m
kube-system   calico-kube-controllers-798cc86c47-2bcq4      1/1     Running   5 (11h ago)    29d
kube-system   kube-proxy-g8mq4                              1/1     Running   5 (10h ago)    29d
default       cpu-pod1                                      1/1     Running   1 (11h ago)    22h

[root@master1 ~]# echo "kubectl get pod -A --sort-by=.metadata.uid" > /var/log/find_pods_uid.sh

[root@master1 ~]# sh /var/log/find_pods_uid.sh
NAMESPACE     NAME                                          READY   STATUS    RESTARTS       AGE
kube-system   kube-proxy-q9gtm                              1/1     Running   5 (11h ago)    29d
default       web-app-66f7dd595-pm6vh                       1/1     Running   0              5h8m
default       web-app-66f7dd595-xlmvj                       1/1     Running   0              5h8m
default       web-app-66f7dd595-wc44q                       1/1     Running   0              5h8m
kube-system   coredns-787d4945fb-mtjv5                      1/1     Running   5 (11h ago)    29d
default       cpu-pod2                                      1/1     Running   1 (11h ago)    22h
kube-system   kube-controller-manager-master1.example.com   1/1     Running   12 (11h ago)   29d
default       web-app-66f7dd595-5b88b                       1/1     Running   0              5h9m
default       web-prod-268-645746b847-74572                 1/1     Running   0              6h58m
kube-system   etcd-master1.example.com                      1/1     Running   6 (11h ago)    29d
project-lab   oodb-2                                        1/1     Running   0              40m
kube-system   kube-proxy-dwr6t                              1/1     Running   5 (11h ago)    29d
project-lab   oodb-0                                        1/1     Running   0              40m
kube-system   kube-scheduler-master1.example.com            1/1     Running   10 (10h ago)   29d
kube-system   kube-apiserver-master1.example.com            1/1     Running   7 (11h ago)    29d
kube-system   calico-node-672fx                             1/1     Running   37 (11h ago)   381d
kube-system   metrics-server-65bd7dd65d-gznz4               0/1     Pending   0              21h
project-lab   oodb-1                                        1/1     Running   0              40m
kube-system   calico-node-c5kn4                             1/1     Running   53 (11h ago)   381d
default       max-pod1                                      1/1     Running   1 (11h ago)    22h
kube-system   metrics-server-6775944cc6-zwggs               1/1     Running   4 (10h ago)    19h
kube-system   coredns-787d4945fb-rqm58                      1/1     Running   5 (11h ago)    29d
kube-system   metrics-server-65bd7dd65d-qt2gr               0/1     Running   1 (11h ago)    21h
kube-system   calico-node-tszjn                             1/1     Running   51 (11h ago)   381d
default       web-app-66f7dd595-c6q4z                       1/1     Running   0              5h9m
default       web-app-66f7dd595-cb8mn                       1/1     Running   0              5h8m
kube-system   calico-kube-controllers-798cc86c47-2bcq4      1/1     Running   5 (11h ago)    29d
kube-system   kube-proxy-g8mq4                              1/1     Running   5 (11h ago)    29d
default       cpu-pod1                                      1/1     Running   1 (11h ago)    22h
[root@master1 ~]# 


If you want to see the UID of pods then you need to specify the custom column.

[a@master1 ~]# kubectl get pods -A -o custom-columns=PodName:.metadata.name,PodUID:.metadata.uid --sort-by=.metadata.uid
PodName                                       PodUID
kube-proxy-q9gtm                              2a9d3417-06e9-41d9-b3c0-1f91a513743e
web-app-66f7dd595-pm6vh                       2ae2e338-8011-4210-958d-6313f2a6cc51
web-app-66f7dd595-xlmvj                       3a965bcc-1077-4702-b7e2-ae0ada9c4e69
web-app-66f7dd595-wc44q                       6a645ffd-648d-4b7f-813e-c189008a91bd
coredns-787d4945fb-mtjv5                      78a4faa4-1ee7-4aec-a9d7-a46f4921e6ec
cpu-pod2                                      79d72aeb-43a4-45f3-835a-6cde0e05ac01
kube-controller-manager-master1.example.com   083c8c23-519d-4354-8511-48f57ae7d3cb
web-app-66f7dd595-5b88b                       681aab8b-3c61-44f9-b210-bd9a70f42bdc
web-prod-268-645746b847-74572                 774f4a6d-3ae1-4939-b374-01641bc76f58
etcd-master1.example.com                      960e66e0-534e-44cc-881d-234c67b91ee4
oodb-2                                        6302c51c-180b-4790-94df-0517d8ff8892
kube-proxy-dwr6t                              94973f3e-f86e-459a-bb74-57def4c90d77
oodb-0                                        800079e1-d734-4a98-8792-7bc1262ceae5
kube-scheduler-master1.example.com            a57556b6-a08a-4f2b-b3d2-18cb4815c6eb
kube-apiserver-master1.example.com            b7c72252-da79-483b-8919-386904eacc80
calico-node-672fx                             b46cf0a5-2588-419a-9550-5807664ef79b
metrics-server-65bd7dd65d-gznz4               b54e0f29-9a53-45ba-a137-90096f050cbd
oodb-1                                        b83b8038-64f0-45da-a7e3-bac6902af7c8

```
