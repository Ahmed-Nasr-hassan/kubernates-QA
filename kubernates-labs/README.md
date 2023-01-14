# kubernates-QA

## Lab2

1-Create a ReplicaSet using the below yaml

apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: new-replica-set
  namespace: default
spec:
  replicas: 4
  selector:
    matchLabels:
      name: busybox-pod
  template:
    metadata:
      labels:
        name: busybox-pod
    spec:
      containers:
      - command:
        - sh
        - -c
        - echo Hello Kubernetes! && sleep 3600
        image: busybox777
        imagePullPolicy: Always
        name: busybox-container

``` bash
controlplane $ kubectl apply -f busy-box.yaml 
replicaset.apps/new-replica-set created
controlplane $ k get rs
NAME              DESIRED   CURRENT   READY   AGE
new-replica-set   4         4         0       6s
```


2-How many PODs are DESIRED in the new-replica-set?

``` bash
controlplane $ k describe rs new-replica-set | grep "Replicas"
Replicas:     4 current / 4 desired
#another solution
controlplane $ k get po | grep "new-replica-set" | wc -l
4
```

3-What is the image used to create the pods in the new-replica-set?

```bash
controlplane $ k describe rs new-replica-set | grep "Image"
    Image:      busybox777
```

4-How many PODs are READY in the new-replica-set?

0

```bash
controlplane $ k describe rs new-replica-set | grep "Pods Status"
Pods Status:  0 Running / 4 Waiting / 0 Succeeded / 0 Failed
#another solution
controlplane $ k get rs new-replica-set                
NAME              DESIRED   CURRENT   READY   AGE
new-replica-set   4         4         0       11m

```

5-Why do you think the PODs are not ready?

There is image called "busybox777"

```bash
controlplane $ k describe pods new-replica-set-llbmd | grep "Failed"
  Warning  Failed     3m28s (x4 over 4m53s)  kubelet            Failed to pull image "busybox777": rpc error: code = Unknown desc = failed to pull and unpack image "docker.io/library/busybox777:latest": failed to resolve reference "docker.io/library/busybox777:latest": pull access denied, repository does not exist or may require authorization: server message: insufficient_scope: authorization failed
  Warning  Failed     3m28s (x4 over 4m53s)  kubelet            Error: ErrImagePull
  Warning  Failed     3m1s (x6 over 4m53s)   kubelet            Error: ImagePullBackOff
```

6-Delete any one of the 4 PODs
How many pods now

4

```bash
controlplane $ k delete pod new-replica-set-79cxt 
pod "new-replica-set-79cxt" deleted
controlplane $ k get pods 
NAME                    READY   STATUS             RESTARTS   AGE
new-replica-set-brm9l   0/1     ImagePullBackOff   0          15m
new-replica-set-gcch6   0/1     ImagePullBackOff   0          15m
new-replica-set-llbmd   0/1     ErrImagePull       0          7s
new-replica-set-wfmp7   0/1     ImagePullBackOff   0          15m

```

7-Why are there still 4 PODs, even after you deleted one?

As the desired replicas equals to 4,
so it maintain the number of pods to be 4


8-Create a ReplicaSet using the below yaml

There is an issue with the file, so try to fix it.

apiVersion: v1
kind: ReplicaSet
metadata:
  name: replicaset-1
spec:
  replicas: 2
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
      - name: nginx
        image: nginx

## set >> apiVersion: apps/v1

```bash
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: replicaset-1
spec:
  replicas: 2
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
      - name: nginx
        image: nginx
```
In Kubernetes, API versions are used to indicate which version of the Kubernetes API is being used to create or manage resources. The "v1" API version is the core Kubernetes API, and includes basic resources such as Pods and Services. However, additional resources and features, such as ReplicaSets and Deployments, are not included in the core API and are instead part of the "apps/v1" API version.

This is because ReplicaSets and Deployments are higher level abstractions that are built on top of the core resources like Pods. They provide additional functionality, such as replication and self-healing, that is not included in the core API. By grouping these additional resources into separate API versions, it allows for backwards compatibility and enables users to more easily adopt new features as they are added to Kubernetes.

```bash
controlplane $ k api-resources | grep "replicaset"
replicasets                       rs           apps/v1                                true         
```

---

## Lab3

1-Create a deployment called my-first-deployment of image nginx:alpine in the default namespace.
Check to make sure the deployment is healthy.

```bash
#deployment file
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-first-deployment
  labels:
    app: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:alpine
        ports:
        - containerPort: 80
~                           
```
```bash
controlplane $ vi my-deploy.yml
controlplane $ k apply -f my-deploy.yml  
deployment.apps/my-first-deployment created
controlplane $ k get deployments.apps 
NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
my-first-deployment   1/1     1            1           8s
controlplane $ k get pods
NAME                                   READY   STATUS    RESTARTS   AGE
my-first-deployment-774f96d4d9-rwrqx   1/1     Running   0          2m42s
```

2-Scale my-first-deployment up to run 3 replicas.
Check to make sure all 3 replicas are ready.

```bash
controlplane $ k scale deployment my-first-deployment --replicas=3
deployment.apps/my-first-deployment scaled

NAME                             DESIRED   CURRENT   READY   AGE
my-first-deployment-774f96d4d9   3         3         3       4m57s

```

3-Scale my-first-deployment down to run 2 replicas.

```bash
controlplane $ k scale deployment my-first-deployment --replicas=2
deployment.apps/my-first-deployment scaled
controlplane $ k get rs
NAME                             DESIRED   CURRENT   READY   AGE
my-first-deployment-774f96d4d9   2         2         2       12m
```

4-Change the image my-first-deployment runs from nginx:alpine to httpd:alpine .

```bash
controlplane $ vi my-deploy.yml # edit image field in yml file
controlplane $ k apply -f my-deploy.yml 
deployment.apps/my-first-deployment configured

```

5-Delete the deployment my-first-deployment

```bash
controlplane $ k delete deployments.apps my-first-deployment 
deployment.apps "my-first-deployment" deleted
controlplane $ k get deployments.apps 
No resources found in default namespace.
```


6-Create deployment from the below yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend-deployment
  namespace: default
spec:
  replicas: 4
  selector:
    matchLabels:
      name: busybox-pod
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      labels:
        name: busybox-pod
    spec:
      containers:
      - command:
        - sh
        - -c
        - echo Hello Kubernetes! && sleep 3600
        image: busybox888
        imagePullPolicy: Always
        name: busybox-container
        

```bash
controlplane $ vi q6-deploy.yml # deployment definition file
controlplane $ k apply -f q6-deploy.yml 
deployment.apps/frontend-deployment created
```

7-How many ReplicaSets exist on the system now?
1
```bash
controlplane $ k get rs
NAME                             DESIRED   CURRENT   READY   AGE
frontend-deployment-7fbf4f5cd9   4         4         0       5m16s
```


8-How many PODs exist on the system now?
4

```bash
controlplane $ k get po 
NAME                                   READY   STATUS             RESTARTS   AGE
frontend-deployment-7fbf4f5cd9-2zgst   0/1     ImagePullBackOff   0          8m40s
frontend-deployment-7fbf4f5cd9-llntp   0/1     ImagePullBackOff   0          8m40s
frontend-deployment-7fbf4f5cd9-rtk4d   0/1     ImagePullBackOff   0          8m40s
frontend-deployment-7fbf4f5cd9-z6ckk   0/1     ImagePullBackOff   0          8m40s
controlplane $ k get po | grep "front" | wc -l
4
```

9-Out of all the existing PODs, how many are ready?
zero

```bash
controlplane $ k describe deployments.apps frontend-deployment | grep "Replica"
Replicas:               4 desired | 4 updated | 4 total | 0 available | 4 unavailable
```

10-What is the image used to create the pods in the new deployment?

```bash
controlplane $ k describe deployments.apps frontend-deployment | grep "Image"
    Image:      busybox888
```

11-Why do you think the deployment is not ready?
Cannot pull image busybox888

```bash
controlplane $ k describe pod frontend-deployment-7fbf4f5cd9-rtk4d | grep "Failed"
  Warning  FailedMount  13m                   kubelet            MountVolume.SetUp failed for volume "kube-api-access-nzlj7" : failed to sync configmap cache: timed out waiting for the condition
  Warning  Failed       11m (x4 over 13m)     kubelet            Failed to pull image "busybox888": rpc error: code = Unknown desc = failed to pull and unpack image "docker.io/library/busybox888:latest": failed to resolve reference "docker.io/library/busybox888:latest": pull access denied, repository does not exist or may require authorization: server message: insufficient_scope: authorization failed
  Warning  Failed       11m (x4 over 13m)     kubelet            Error: ErrImagePull
  Warning  Failed       11m (x6 over 13m)     kubelet            Error: ImagePullBackOff
```

12-Create a new Deployment using the below yaml 

No yml file exist

13-There is an issue with the file, so try to fix it.
and correct the value of kind.

No yml file exist

---
## Lab4

1-How many Services exist on the system?
1

```bash
controlplane $ k get svc
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   22d
```

2-What is the type of the default kubernetes service?
ClusterIP

3-What is the targetPort configured on the kubernetes service?

```bash
controlplane $ k describe svc kubernetes | grep "TargetPort"
TargetPort:        6443/TCP
```

4-How many labels are configured on the kubernetes service?
2

```bash
controlplane $ k describe svc kubernetes                
Name:              kubernetes
Namespace:         default
Labels:            component=apiserver
                   provider=kubernetes
```

5-How many Endpoints are attached on the kubernetes service?
1

```bash
controlplane $ k describe svc kubernetes | grep "Endpoints"
Endpoints:         172.30.1.2:6443
```

6-Create a Deployment using the below yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: simple-webapp-deployment
  namespace: default
spec:
  replicas: 4
  selector:
    matchLabels:
      name: simple-webapp
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      creationTimestamp: null
      labels:
        name: simple-webapp
    spec:
      containers:
      - image: kodekloud/simple-webapp:red
        imagePullPolicy: IfNotPresent
        name: simple-webapp
        ports:
        - containerPort: 8080
          protocol: TCP

```bash
controlplane $ vi q6-lab4-deploy.yml # deployment definition file
controlplane $ k apply -f q6-lab4-deploy.yml 
deployment.apps/simple-webapp-deployment created

```

7-What is the image used to create the pods in the deployment?

```bash
controlplane $ k describe pod simple-webapp-deployment-c7c68b6f4-llnx6 | grep "Image"
    Image:          kodekloud/simple-webapp:red
```

8-Create a new service to access the web application using the the below 

Name: webapp-service
Type: NodePort
targetPort: 8080
port: 8080
nodePort: 30080
selector:
  name: simple-webapp

```bash
# service yaml file
apiVersion: v1
kind: Service
metadata:
  name: webapp-service
spec:
  type: NodePort
  selector:
    app.kubernetes.io/name: simple-webapp
  ports:
      # By default and for convenience, the `targetPort` is set to the same value as the `port` field.
    - port: 8080
      targetPort: 8080
      # Optional field
      # By default and for convenience, the Kubernetes control plane will allocate a port from a range (default: 30000-32767)
      nodePort: 30080
```

```bash
controlplane $ k apply -f my-service.yml 
service/webapp-service created
controlplane $ k get svc     
NAME             TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
kubernetes       ClusterIP   10.96.0.1      <none>        443/TCP          22d
webapp-service   NodePort    10.97.217.52   <none>        8080:30080/TCP   20s
controlplane $ k describe svc webapp-service 
Name:                     webapp-service
Namespace:                default
Labels:                   <none>
Annotations:              <none>
Selector:                 app.kubernetes.io/name=simple-webapp
Type:                     NodePort
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.97.217.52
IPs:                      10.97.217.52
Port:                     <unset>  8080/TCP
TargetPort:               8080/TCP
NodePort:                 <unset>  30080/TCP
Endpoints:                <none>
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```



