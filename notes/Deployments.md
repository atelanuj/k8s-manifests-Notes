# [Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)

- A `Deployment` manages a set of Pods to run an application workload, typically for stateless applications.
- When a `Deployment` is created, a `ReplicaSet` is automatically created with it.
- Deployments are essential for rolling out and rolling back applications to specific versions.
- During a rollout, the old `ReplicaSet` scales to zero while a new `ReplicaSet` is created. During a rollback, the new `ReplicaSet` scales to zero while the old one becomes active.

```bash
kubectl create deployment <deployment-name> --image=<container-image>

kubectl create deployment nginx-deployment --image=nginx --replicas=3

kubectl create deployment nginx-deployment --image=nginx --dry-run=client -o yaml

kubectl create deployment nginx-deployment --image=nginx --dry-run=client -o yaml > deployment.yaml
```

 

```yaml
apiVersion: apps/v1 # Specifies the API version for the Deployment object
kind: Deployment # Declares that this manifest is for a Deployment
metadata:
  name: nginx-deployment # Name of the Deployment
  labels:
    app: nginx # Labels for categorizing or filtering the Deployment
spec:
  replicas: 3 # Number of desired pod instances
  selector:
    matchLabels:
      app: nginx # Label selector for identifying the pods managed by this Deployment
  template:
    metadata:
      labels:
        app: nginx # Labels to be applied to each pod created by this Deployment
    spec:
      containers:
      - name: nginx # Name of the container
        image: nginx:1.14.2 # Docker image to use for the container
        ports:
        - containerPort: 80 # Port on which the container listens for traffic
```

```bash
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   0/3     0            0           1s
```

- **Output**
    - `NAME` lists the names of the Deployments in the namespace.
    - `READY` displays how many replicas of the application are available to your users. It follows the pattern ready/desired.
    - `UP-TO-DATE` displays the number of replicas that have been updated to achieve the desired state.
    - `AVAILABLE` displays how many replicas of the application are available to your users.
    - `AGE` displays the amount of time that the application has been running.
- To see the Deployment rollout status, run `kubectl rollout status deployment/nginx-deployment`.

The output is similar to:

```bash
Waiting for rollout to finish: 2 out of 3 new replicas have been updated...
deployment "nginx-deployment" successfully rolled out
```

- Run the `kubectl get deployments` again a few seconds later. The output is similar to this:

```bash
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3/3     3            3           18s
```

- To see the ReplicaSet (`rs`) created by the Deployment, run `kubectl get rs`. The output is similar to this:

```bash
NAME                          DESIRED   CURRENT   READY   AGE
nginx-deployment-75675f5897   3         3         3       18s
```

- To see the labels automatically generated for each Pod, run `kubectl get pods --show-labels`. The output is similar to:

```bash
NAME                                READY     STATUS    RESTARTS   AGE       LABELS
nginx-deployment-75675f5897-7ci7o   1/1       Running   0          18s       app=nginx,pod-template-hash=75675f5897
nginx-deployment-75675f5897-kzszj   1/1       Running   0          18s       app=nginx,pod-template-hash=75675f5897
nginx-deployment-75675f5897-qqcnn   1/1       Running   0          18s       app=nginx,pod-template-hash=75675f5897
```

---

## **Updating a Deployment**

- Let's update the nginx Pods to use the `nginx:1.16.1` image instead of the `nginx:1.14.2` image.
    
    `kubectl set image deployment.v1.apps/nginx-deployment nginx=nginx:1.16.1`
    

The output is similar to:

```bash
deployment.apps/nginx-deployment image updated
```

- Alternatively, you can `edit` the Deployment and change `.spec.template.spec.containers[0].image` from `nginx:1.14.2` to `nginx:1.16.1`:
    
    `kubectl edit deployment/nginx-deployment`
    
- After the rollout succeeds, you can view the Deployment by running `kubectl get deployments`. The output is similar to this:
    
    ```bash
    NAME               READY   UP-TO-DATE   AVAILABLE   AGE
    nginx-deployment   3/3     3            3           36s
    ```
    
- Run `kubectl get rs` to see that the Deployment updated the Pods by creating a new ReplicaSet and scaling it up to 3 replicas, as well as scaling down the old ReplicaSet to 0 replicas.

```bash
NAME                          DESIRED   CURRENT   READY   AGE
nginx-deployment-1564180365   3         3         3       6s
nginx-deployment-2035384211   0         0         0       36
```

- Deployment ensures that only a certain number of Pods are down while they are being updated. By default, it ensures that at least 75% of the desired number of Pods are up (25% max unavailable).
- Deployment also ensures that only a certain number of Pods are created above the desired number of Pods. By default, it ensures that at most 125% of the desired number of Pods are up (25% max surge).
- **For example**, if you look at the above Deployment closely, you will see that it first creates a new Pod, *then deletes an old Pod, and creates another new one. It does not kill old Pods until a sufficient number of new Pods have come up*, and does not create new Pods until a sufficient number of old Pods have been killed. It makes sure that at least 3 Pods are available and that at max 4 Pods in total are available. In case of a Deployment with 4 replicas, the number of Pods would be between 3 and 5.
- Get details of your Deployment: `kubectl describe deployments`

```bash
Name:                   nginx-deployment
Namespace:              default
CreationTimestamp:      Thu, 30 Nov 2017 10:56:25 +0000
Labels:                 app=nginx
Annotations:            deployment.kubernetes.io/revision=2
Selector:               app=nginx
Replicas:               3 desired | 3 updated | 3 total | 3 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=nginx
   Containers:
    nginx:
      Image:        nginx:1.16.1
      Port:         80/TCP
      Environment:  <none>
      Mounts:       <none>
    Volumes:        <none>
  Conditions:
    Type           Status  Reason
    ----           ------  ------
    Available      True    MinimumReplicasAvailable
    Progressing    True    NewReplicaSetAvailable
  OldReplicaSets:  <none>
  NewReplicaSet:   nginx-deployment-1564180365 (3/3 replicas created)
  Events:
    Type    Reason             Age   From                   Message
    ----    ------             ----  ----                   -------
    Normal  ScalingReplicaSet  2m    deployment-controller  Scaled up replica set nginx-deployment-2035384211 to 3
    Normal  ScalingReplicaSet  24s   deployment-controller  Scaled up replica set nginx-deployment-1564180365 to 1
    Normal  ScalingReplicaSet  22s   deployment-controller  Scaled down replica set nginx-deployment-2035384211 to 2
    Normal  ScalingReplicaSet  22s   deployment-controller  Scaled up replica set nginx-deployment-1564180365 to 2
    Normal  ScalingReplicaSet  19s   deployment-controller  Scaled down replica set nginx-deployment-2035384211 to 1
    Normal  ScalingReplicaSet  19s   deployment-controller  Scaled up replica set nginx-deployment-1564180365 to 3
    Normal  ScalingReplicaSet  14s   deployment-controller  Scaled down replica set nginx-deployment-2035384211 to 0
```

- If you update a Deployment *while an existing rollout is in progress*, the Deployment creates a new ReplicaSet as per the update and start scaling that up, and rolls over the ReplicaSet that it was scaling up previously -- it will add it to its list of old ReplicaSets and start scaling it down.
- **For example**, suppose you create a Deployment to create 5 replicas of `nginx:1.14.2`, but then update the Deployment to create 5 replicas of `nginx:1.16.1`, when only 3 replicas of `nginx:1.14.2` had been created. In that case, the Deployment immediately starts killing the 3 `nginx:1.14.2` Pods that it had created, and starts creating `nginx:1.16.1` Pods. It does not wait for the 5 replicas of `nginx:1.14.2` to be created before changing course.

---

## **Rolling Back a Deployment**

- Sometimes, you may want to rollback a Deployment; for example, when the Deployment is not stable, such as crash looping. By default, all of the Deployment's rollout history is kept in the system so that you can rollback anytime you want (you can change that by modifying revision history limit).
- Suppose that you made a **typo** while updating the Deployment, by putting the image name as `nginx:1.161` instead of `nginx:1.16.1`
- You see that the number of old replicas (adding the replica count from `nginx-deployment-1564180365` and `nginx-deployment-2035384211`) is 3, and the number of new replicas (from `nginx-deployment-3066724191`) is 1. `kubectl get rs`

```bash
NAME                          DESIRED   CURRENT   READY   AGE
nginx-deployment-1564180365   3         3         3       25s
nginx-deployment-2035384211   0         0         0       36s
nginx-deployment-3066724191   1         1         0       6s
```

- Looking at the Pods created, you see that 1 Pod created by new ReplicaSet is stuck in an image pull loop. `kubectl get pods`

```bash
NAME                                READY     STATUS             RESTARTS   AGE
nginx-deployment-1564180365-70iae   1/1       Running            0          25s
nginx-deployment-1564180365-jbqqo   1/1       Running            0          25s
nginx-deployment-1564180365-hysrc   1/1       Running            0          25s
nginx-deployment-3066724191-08mng   0/1       ImagePullBackOff   0          6s
```

- `kubectl describe deployment`

```bash
Name:           nginx-deployment
Namespace:      default
CreationTimestamp:  Tue, 15 Mar 2016 14:48:04 -0700
Labels:         app=nginx
Selector:       app=nginx
Replicas:       3 desired | 1 updated | 4 total | 3 available | 1 unavailable
StrategyType:       RollingUpdate
MinReadySeconds:    0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=nginx
  Containers:
   nginx:
    Image:        nginx:1.161
    Port:         80/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    ReplicaSetUpdated
OldReplicaSets:     nginx-deployment-1564180365 (3/3 replicas created)
NewReplicaSet:      nginx-deployment-3066724191 (1/1 replicas created)
Events:
  FirstSeen LastSeen    Count   From                    SubObjectPath   Type        Reason              Message
  --------- --------    -----   ----                    -------------   --------    ------              -------
  1m        1m          1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled up replica set nginx-deployment-2035384211 to 3
  22s       22s         1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled up replica set nginx-deployment-1564180365 to 1
  22s       22s         1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled down replica set nginx-deployment-2035384211 to 2
  22s       22s         1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled up replica set nginx-deployment-1564180365 to 2
  21s       21s         1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled down replica set nginx-deployment-2035384211 to 1
  21s       21s         1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled up replica set nginx-deployment-1564180365 to 3
  13s       13s         1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled down replica set nginx-deployment-2035384211 to 0
  13s       13s         1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled up replica set nginx-deployment-3066724191 to 1
```

- To fix this, you need to rollback to a previous revision of Deployment that is stable.

---

### **Checking Rollout History of a Deployment**

- `kubectl rollout history deployment/nginx-deployment`

```bash
deployments "nginx-deployment"
REVISION    CHANGE-CAUSE
1           kubectl apply --filename=https://k8s.io/examples/controllers/nginx-deployment.yaml
2           kubectl set image deployment/nginx-deployment nginx=nginx:1.16.1
3           kubectl set image deployment/nginx-deployment nginx=nginx:1.161
```

- To see the details of each revision, run: `kubectl rollout history deployment/nginx-deployment --revision=2`

```bash
deployments "nginx-deployment" revision 2
  Labels:       app=nginx
          pod-template-hash=1159050644
  Annotations:  kubernetes.io/change-cause=kubectl set image deployment/nginx-deployment nginx=nginx:1.16.1
  Containers:
   nginx:
    Image:      nginx:1.16.1
    Port:       80/TCP
     QoS Tier:
        cpu:      BestEffort
        memory:   BestEffort
    Environment Variables:      <none>
  No volumes.
```

---

### **Rolling Back to a Previous Revision**

- `kubectl rollout undo deployment/nginx-deployment`
- Alternatively, you can rollback to a specific revision by specifying it with `--to-revision`:

```bash
kubectl rollout undo deployment/nginx-deployment --to-revision=2
```

---

## **Scaling a Deployment**

- `kubectl scale deployment/nginx-deployment --replicas=10`

---

## **Pausing and Resuming a rollout of a Deployment**

- Pause by running the following command: `kubectl rollout pause deployment/nginx-deployment`
- Eventually, resume the Deployment rollout and observe a new ReplicaSet coming up with all the new updates: `kubectl rollout resume deployment/nginx-deployment`

---

## **Deployment status**

- **Progressing Deployment**
    - The Deployment creates a new `ReplicaSet`.
    - The Deployment is scaling up its newest `ReplicaSet`.
    - The Deployment is scaling down its older `ReplicaSet`(s).
    - New Pods become ready or available (ready for at least [MinReadySeconds](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#min-ready-seconds)).
- **Complete Deployment [](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#complete-deployment)**
    - All of the replicas associated with the Deployment have been updated to the latest version you've specified, meaning any updates you've requested have been completed.
    - All of the replicas associated with the Deployment are available.
    - No old replicas for the Deployment are running.
- **Failed Deployment**
    - Insufficient quota
    - Readiness probe failures
    - Image pull errors
    - Insufficient permissions
    - Limit ranges
    - Application runtime misconfiguration

---

## Deployment **Strategy**

- **Recreate Deployment**
    - All existing Pods are killed before new ones are created when `.spec.strategy.type==Recreate`
- **Rolling Update Deployment**
    - "**RollingUpdate**" is the default value
    - Minimizes downtime as both versions may run simultaneously for a short period.
    - The Deployment updates Pods in a rolling update fashion when `.spec.strategy.type==RollingUpdate`. You can specify `maxUnavailable` and `maxSurge` to control the rolling update process.
    - **Max Unavailable**
        - The default value is `25%`.
        - optional field that specifies the maximum number of Pods that can be unavailable during the update process.
    - **Max Surge**
        - The default value is `25%.`
        - optional field that specifies the maximum number of Pods that can be created over the desired number of Pods
    - Example:
    
    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
     name: nginx-deployment
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
           image: nginx:1.14.2
           ports:
           - containerPort: 80
     strategy:
       type: RollingUpdate
       rollingUpdate:
         maxSurge: 1
         maxUnavailable: 1
    ```
    
- Key Considerations
    - Use `RollingUpdate` for high availability services.
    - Use `Recreate` for cases where you need a clean state or when compatibility issues prevent versions from coexisting.
    - Tweak `maxUnavailable` and `maxSurge` to balance speed and reliability of updates.
    - `Blue Green` and `canary` Deployment strategy is not supported by K8s. you can achieve this with the help of `ingress controller`.