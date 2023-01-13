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
replicasets                       rs           apps/v1                                true         ReplicaSet
```
