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
    name: simple-webapp
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
Selector:                 name=simple-webapp
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

---

## lab5

### Create basic resources

- Create the specified Namespace
- Create a deployment nginx with 3 replicas
- Expose the deployment on port 80

```bash
controlplane $ k create ns nasr-space
namespace/nasr-space created
controlplane $ vi nasr-deploy.yml
```

```bash
# nasr-deploy.yml file
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  namespace: nasr-space
  labels:
    app: nginx
spec:
  replicas: 3
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
        image: nginx
        ports:
        - containerPort: 80
```

```bash
controlplane $ k apply -f nasr-deploy.yml 
deployment.apps/nginx-deployment created
controlplane $ k get all -n nasr-space                 
NAME                                    READY   STATUS    RESTARTS   AGE
pod/nginx-deployment-7f456874f4-6xchb   1/1     Running   0          51s
pod/nginx-deployment-7f456874f4-gg8nt   1/1     Running   0          51s
pod/nginx-deployment-7f456874f4-tcz9r   1/1     Running   0          51s

NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-deployment   3/3     3            3           52s
```

### Create a CronJob for listing the EndPoints

1. Create a **serviceaccount cronjob???sa

```bash
controlplane $ k create serviceaccount cronjopn-sa
serviceaccount/cronjopn-sa created
```

2. Create a Role that allows listing all the services and endpoints

```bash
# role.yml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: nasr-space
  name: svc-endpoints-list
rules:
- apiGroups: [""] # "" indicates the core API group
  resources: ["services","endpoints"]
  verbs: ["list","get"]                                     
```

```bash
controlplane $ k apply -f role.yml 
role.rbac.authorization.k8s.io/svc-endpoints-list created
```

3. Link the Role with the created SA

```bash
# rolebinding.yml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: my-rolebinding
  namespace: nasr-space
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: svc-endpoints-list
subjects:
- kind: ServiceAccount
  name: cronjob-sa
  namespace: nasr-space

```

```bash
controlplane $ k apply -f rolebinding.yml 
rolebinding.rbac.authorization.k8s.io/my-rolebinding created
```

4. Create a CronJob that lists the endpoints in that namespace every minute and paste the output for the first pod created

```bash
# my-cronjob.yml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: list-endpoints-in-nasr-space
  namespace: nasr-space
spec:
  schedule: "* * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          serviceAccountName: cronjob-sa
          containers:
          - name: bitnami-container
            image: bitnami/kubectl:latest
            imagePullPolicy: IfNotPresent
            command:
            - sh
            - -c
            - "kubectl get endpoints" # "sleep 4000" # to keep the pod running
          restartPolicy: OnFailure

```

5. After listing try to delete the 3 nginx pods ? again try to view the logs for the newly created pod for that cronJob what do you think happened ?

```bash
k delete deployment nginx-deployment
k logs -n nasr-space list-endpoints-in-nasr-space-27896516-c5p8w 
```

## lab6

1. create pod from the below yaml file

    ```yaml
        apiVersion: v1
        kind: Pod
        metadata:
          name: webapp
        spec:
          containers:
          - env:
            - name: LOG_HANDLERS
              value: file
            image: kodekloud/event-simulator
            imagePullPolicy: Always
            name: event-simulator
    ```

  ```bash
    controlplane $ vi lab6-pod.yml
    controlplane $ k apply -f lab6-pod.yml 
    pod/webapp created
    controlplane $ k get po
    NAME     READY   STATUS    RESTARTS   AGE
    webapp   1/1     Running   0          12s
    
  ```

2. Configure a volume to store these logs at /var/log/webapp on the host.

    Use the spec provided below.

    ```yaml
      Name: webapp

      Image Name: kodekloud/event-simulator

      Volume HostPath: /var/log/webapp

      Volume Mount: /log
    ```

    updated yaml

    ```yaml
        apiVersion: v1
        kind: Pod
        metadata:
          name: webapp
        spec:
          volumes:
          - name: webapp
            hostPath:
              path: /var/log/webapp
          containers:
          - volumeMounts:
            - name: webapp
              mountPath: /log
            env:
            - name: LOG_HANDLERS
              value: file
            image: kodekloud/event-simulator
            imagePullPolicy: Always
            name: event-simulator
    ```

    ```bash
      controlplane $ k delete pod webapp 
      pod "webapp" deleted
      controlplane $ k apply -f lab6-pod.yml 
      pod/webapp created
      controlplane $ k get po 
      NAME     READY   STATUS    RESTARTS   AGE
      webapp   1/1     Running   0          19s
    ```

3. Create a Persistent Volume with the given specification.

  ```yaml
    Volume Name: pv-log

    Storage: 100Mi

    Access Modes: ReadWriteMany

    Host Path: /pv/log

    Reclaim Policy: Retain
  ```
  
  persistent volume yaml file: my-pv.yml

  ```yaml
    apiVersion: v1
    kind: PersistentVolume
    metadata:
      name: pv-log
    spec:
      capacity:
        storage: 100Mi
      accessModes:
        - ReadWriteMany
      persistentVolumeReclaimPolicy: Retain
      hostPath:
        path: "/pv/log"
  ```

  In terminal

  ```bash
    controlplane $ vi my-pv.yml
    controlplane $ k apply -f my-pv.yml 
    persistentvolume/pv-log created
  ```

4. Let us claim some of that storage for our application. Create a Persistent

    Volume Claim with the given specification.

    ```yaml
      Volume Name: claim-log-1

      Storage Request: 50Mi

      Access Modes: ReadWriteOnce
    ```

    persistent volume claim yaml file: my-pvc.yml

    ```yaml
      apiVersion: v1
      kind: PersistentVolumeClaim
      metadata:
        name: claim-log-1
      spec:
        accessModes:
          - ReadWriteOnce
        resources:
          requests:
            storage: 50Mi
    ```

    In terminal

    ```bash
      controlplane $ vi my-pvc.yml
      controlplane $ k apply -f my-pvc.yml 
      persistentvolumeclaim/claim-log-1 created
    ```


5. What is the state of the Persistent Volume Claim?

    status: Pending

  ```bash
    controlplane $ k get pvc
    NAME          STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
    claim-log-1   Pending                                                     3m10s
  ```

6. What is the state of the Persistent Volume?

    status: Available

    ```bash
      controlplane $ k get pv 
      NAME     CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE
      pv-log   100Mi      RWX            Retain           Available                                   11m
    ```

7. Why is the claim not bound to the available Persistent Volume?

    Due to different access modes.

8. Update the Access Mode on the claim to bind it to the PV?

    updated pvc file

    ```yaml
      apiVersion: v1
      kind: PersistentVolumeClaim
      metadata:
        name: claim-log-1
      spec:
        accessModes:
          - ReadWriteMany
        resources:
          requests:
            storage: 50Mi
    ```

    In terminal

    ```bash
      controlplane $ k delete pvc claim-log-1 
      persistentvolumeclaim "claim-log-1" deleted
      controlplane $ k apply -f my-pvc.yml 
      persistentvolumeclaim/claim-log-1 created
      controlplane $ k get pvc
      NAME          STATUS   VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
      claim-log-1   Bound    pv-log   100Mi      RWX                           23s
    ```

    status: Bound

9. create pv, pvc, and mount a pod using pvc

  pv yaml file - nasr-pv.yml

  ```yaml
    apiVersion: v1
    kind: PersistentVolume
    metadata:
      name: pv-log
    spec:
      capacity:
        storage: 100Mi
      accessModes:
        - ReadWriteOnce
      persistentVolumeReclaimPolicy: Retain
      hostPath:
        path: "/pv/log"
  ```

  pvc yaml file - nasr-pvc.yml

  ```yaml
    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      name: claim-log-1
    spec:
      accessModes:
        - ReadWriteOnce
      resources:
        requests:
          storage: 100Mi
  ```

  pod yaml file - nasr-pod.yaml

  ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: webapp2
    spec:
      volumes:
      - name: nasr-pvc
        persistentVolumeClaim:
          claimName: claim-log-1
      containers:
      - volumeMounts:
        - name: nasr-pvc
          mountPath: /log
        env:
        - name: LOG_HANDLERS
          value: file
        image: kodekloud/event-simulator
        imagePullPolicy: Always
        name: event-simulator
  ```

  In terminal

  ```bash
    controlplane $ vi nasr-pv.yml 
    controlplane $ vi nasr-pvc.yml 
    controlplane $ vi nasr-pod.yml 
    controlplane $ k apply -f nasr-pv.yml 
    persistentvolume/pv-log created
    controlplane $ k apply -f nasr-pvc.yml 
    persistentvolumeclaim/claim-log-1 created
    controlplane $ k apply -f nasr-pod.yml 
    pod/webapp2 created
    controlplane $ k get pv
    NAME     CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                 STORAGECLASS   REASON   AGE
    pv-log   100Mi      RWO            Retain           Bound    default/claim-log-1                           19s
    controlplane $ k get pvc
    NAME          STATUS   VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
    claim-log-1   Bound    pv-log   100Mi      RWO                           19s
    controlplane $ k get po
    NAME      READY   STATUS    RESTARTS   AGE
    webapp2   1/1     Running   0          18s
    controlplane $ k exec -it webapp2 -- /bin/sh
    / # cd log/
    /log # ls
    app.log   nasr.txt
    /log # exit
  ```

## lab7

1. The DevOps team would like to get the list of all Namespaces in the cluster. Get the list and save it to /opt/namespaces.

    ```bash
      controlplane $ k get ns > /opt/namespaces
      controlplane $ cat /opt/namespaces 
      NAME              STATUS   AGE
      default           Active   30d
      kube-node-lease   Active   30d
      kube-public       Active   30d
      kube-system       Active   30d
    ```

2. create  ServiceAccount named neptune-sa-v2 in Namespace neptune.

    ```bash
      controlplane $ k create namespace neptune
      namespace/neptune created
      
      controlplane $ k create serviceaccount neptune-sa-v2 -n neptune 
      serviceaccount/neptune-sa-v2 created
    ```

3. Create a new ConfigMap named cm-3392845. Use the spec given on the below.

ConfigName Name: cm-3392845

Data: DB_NAME=SQL3322

Data: DB_HOST=sql322.mycompany.com

Data: DB_PORT=3306

```bash
    controlplane $ vi configmap_data.txt # add data in a file

    controlplane $ cat configmap_data.txt 
    DB_NAME=SQL3322
    DB_HOST=sql322.mycompany.com
    DB_PORT=3306

    controlplane $ k create configmap cm-3392845 --from-env-file=configmap_data.txt                        
    configmap/cm-3392845 created
```

4. Team Pluto needs a new cluster internal Service. Create a ClusterIP Service named project-plt-6cc-svc in Namespace pluto. This Service should expose a single Pod named project-plt-6cc-api of image nginx:1.17.3-alpine, create that Pod as well. The Pod should be identified by label project: plt-6cc-api. The Service should use tcp port redirection of 3333:80.

```bash
    controlplane $ k create ns pluto
    namespace/pluto created

    controlplane $ k run project-plt-6cc-api --image=nginx:1.17.3-alpine --namespace pluto --labels project="plt-6cc-api"                         
    pod/project-plt-6cc-api created

    controlplane $ k expose --namespace pluto pod project-plt-6cc-api --type ClusterIP --port 3333 --target-port 80 --name project-plt-6cc-svc
    service/project-plt-6cc-api exposed
```

5. Create a new PersistentVolume named earth-project-earthflower-pv. It should have a capacity of 2Gi, accessMode ReadWriteOnce, hostPath /Volumes/Data and no storageClassName defined.

    ```yaml
        apiVersion: v1
        kind: PersistentVolume
        metadata:
          name: earth-project-earthflower-pv
        spec:
          capacity:
            storage: 2Gi
          accessModes:
            - ReadWriteOnce
          persistentVolumeReclaimPolicy: Recycle
          hostPath:
            path: /Volumes/Data
    ```

Next create a new PersistentVolumeClaim in Namespace earth named earth-project-earthflower-pvc . It should request 2Gi storage, accessMode ReadWriteOnce and should not define a storageClassName. The PVC should bound to the PV correctly.

  ```yaml
    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      name: earth-project-earthflower-pvc
    spec:
      accessModes:
        - ReadWriteOnce
      resources:
        requests:
          storage: 2Gi
  ```

    ```bash
      controlplane $ vi my-pv.yml
      controlplane $ k apply -f my-pv.yml 
      persistentvolume/earth-project-earthflower-pv created
      controlplane $ vi my-pvc.yml 
      controlplane $ k create ns earth                   
      namespace/earth created
      controlplane $ k apply -f my-pvc.yml --namespace earth 
      persistentvolumeclaim/earth-project-earthflower-pvc created
      controlplane $ k get pvc -n earth 
      NAME                            STATUS   VOLUME                         CAPACITY   ACCESS MODES   STORAGECLASS   AGE
      earth-project-earthflower-pvc   Bound    earth-project-earthflower-pv   2Gi        RWO                           13s
      controlplane $ k get pv  
      NAME                           CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                                 STORAGECLASS   REASON   AGE
      earth-project-earthflower-pv   2Gi        RWO            Recycle          Bound    earth/earth-project-earthflower-pvc                           5m1s
    ```

Finally create a new Deployment project-earthflower in Namespace earth which mounts that volume at /tmp/project-data. The Pods of that Deployment should be of image httpd:2.4.41-alpine.

```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      creationTimestamp: null
      labels:
        app: project-earthflower
      name: project-earthflower
      namespace: earth
    spec:
      replicas: 3
      selector:
        matchLabels:
          app: project-earthflower
      strategy: {}
      template:
        metadata:
          creationTimestamp: null
          labels:
            app: project-earthflower
        spec:
          volumes:
          - name: my-claim
            persistentVolumeClaim:
              claimName: earth-project-earthflower-pvc
          
          containers:
          - image: httpd:2.4.41-alpine
            name: httpd
            resources: {}
            volumeMounts:
            - mountPath: "/tmp/project-data"
              name: my-claim
```

```bash
    controlplane $ k create deployment project-earthflower -n earth --image httpd:2.4.41-alpine -oyaml --dry-run=client > my-deployment.yml
    
    controlplane $ vi my-deployment.yml

    controlplane $ k apply -f my-deployment.yml
    deployment.apps/project-earthflower created

    controlplane $ k get -n earth deployments.apps
    NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
    project-earthflower   3/3     3            3           20s

    controlplane $ k get -n earth po
    NAME                                   READY   STATUS    RESTARTS   AGE
    project-earthflower-57f97f68c9-c9p6c   1/1     Running   0          34s
    project-earthflower-57f97f68c9-n664p   1/1     Running   0          34s
    project-earthflower-57f97f68c9-vngmz   1/1     Running   0          34s

    controlplane $ k exec -n earth project-earthflower-57f97f68c9-c9p6c -it -- /bin/sh
    /usr/local/apache2 # cd /tmp/project-data/
    /tmp/project-data # touch nasr.txt
    /tmp/project-data # exit

    controlplane $ k exec -n earth project-earthflower-57f97f68c9-n664p -it -- /bin/sh
    /usr/local/apache2 # cd /tmp/project-data/
    /tmp/project-data # ls
    nasr.txt
```
