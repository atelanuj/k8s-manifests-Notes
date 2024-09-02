# Kubernetes Important Commands

- kubectl get `nodes`/`pods`/`services`/`deployments`/`replicasets`/`namespaces`
- kubectl get all -o `wide`/`yaml`/`json`
- kubectl get `events`
- kubectl get `endpoints`
- kubectl get pods --namespace=`dev`
- Kubectl get pods -n `namespace`
- kubectl get pods --namespace=`dev` --show-labels
- kubectl config set-context --current --namespace=`namespace-name`
- kubectl config set-context `context-name` --cluster=`new-cluster-name` --user=new-`user-name` --namespace=`new-namespace`
- kubectl describe `pod`/`service`/`deployment`/`replicaset`
- kubectl run `name` --image=`image` --command -- `cmd` `args1` `args2`
- kubectl run `name` --image=`image` --dry-run=client -o yaml > `name`.yaml
- kubectl run `name` --image=`image` -- `args1` ... `argsN` # default commands and custom arguments
- kubectl replace --force -f `name`.yaml # replace
- kubectl edit `pod`/`service`/`deployment`/`replicaset` `resource-name` # edit
- kubectl set image deployment `deployment-name` `name`=`image-name`:`version`
- kubectl delete pod `name`
- kubectl delete pod `name` --grace-period=`0` --force # force delete
- kubectl delete pod `name` --grace-period=`0` --force --namespace=`namespace`
- kubectl delete pod `name` --grace-period=`0` --force --namespace=`namespace`
- kubectl logs -f `pod`/`service`/`deployment`/`replicaset` # get logs
- kubectl logs -f `pod` -c `container_name`
- kubectl logs --tail=20 `pod-name`
- kubectl logs --since=1h `pod-name`
- Kubectl exec `pod_name` -it -c `container_name` -- /bin/bash # execute command into the pod or perticular container
- kubectl create `pod`/`service`/`deployment`/`replicaset`/`namespace` -f `name`.yaml
- kubectl create -f `name`.yaml
- kubectl create -f `name`.yaml --record # record the history
- kubectl create configmap `configmap-name` --from-literal=`key=value` --from-literal=`key2=value2`
- kubectl create configmap `configmap-name` --from-file=`path-to-file`
- kubectl create configmap `configmap-name` --from-file=`app.config.properties`
- kubectl create secret generic `secret-name` --from-literal=`key`=`value`
- kubectl create secret generic `secret-name` --from-literal=`key`=`value` --from-literal=`key1`=`value1`
- kubectl create secret generic `secret-name` --from-file=`path-to-file`
- kubectl create secret generic `app-secret` --from-file=`app-secret.properties`
- kubectl apply -f `config1.yaml` -f `config2.yaml` # Applies multiple configuration files.
- kubectl apply -f `https://url.com/config.yaml` # Applies a configuration from a URL.
- kubectl scale deployment `deployment-name` --replicas=`5` -f `deploy.yml` --record # Scales a deployment to 5 replicas and records the change in the revision history.
- kubectl scale replicaset `rs-name` --replicas=`3` -f `replica.yml`
- kubectl rollout history deployment/`deployment-name` # Shows the history of deployments
- kubectl rollout pause deployment/`deployment-name` # Pauses the rollout of a deployment
- kubectl rollout resume deployment/`deployment-name` # Resumes a paused deployment rollout.
- Kubectl rollout status deployment `deployment-name`
- kubectl expose deployment `deployment-name` --port=80 --target-port=8080 # Exposes a deployment as a new service on port 80, targeting the container port 8080.
- kubectl label pods `pod-name` `app=myapp`
- kubectl label nodes `node-name` `disk=ssd`
- kubectl patch deployment deployment-name -p `'{"spec":{"replicas":5}}'` # Updates the number of replicas for a deployment to 5.
- kubectl patch service service-name -p `'{"spec":{"type":"LoadBalancer"}}'` # Updates the service type to LoadBalancer.
- kubectl autoscale deployment `deployment-name` --min=2 --max=10 --cpu-percent=80 # Configures auto-scaling for a deployment based on CPU utilization.
- kubectl autoscale replicaset `rs-name` --min=3 --max=6 --cpu-percent=70 # Configures auto-scaling for a ReplicaSet based on CPU utilization.
- kubectl top `nodes` # Shows CPU and memory usage for nodes in the cluster.
- kubectl top `pods` --containers # Shows CPU and memory usage for containers in pods.
- kubectl drain `node-name` # Safely drains a node in preparation for maintenance.
- kubectl drain `node-name` `--ignore-daemonsets` # Drains a node while ignoring DaemonSet pods.
- kubectl taint nodes `node-name` key=value:NoSchedule
- kubectl taint nodes `node-name` key=value:NoSchedule- # remove taint
- kubectl taint nodes `node-name` key=value:NoExecute
- kubectl taint nodes `node-name` key=value:NoExecute-  # remove taint
- kubectl cordon `node-name` # Marks a node as unschedulable, preventing new pods from being scheduled on it.
- kubectl uncordon `node-name` # Marks a node as schedulable, allowing new pods to be scheduled on it.
- kubectl cp `file.txt` `pod-name`:`/path/to/dir` # Copies a file from the local machine to a pod.
- kubectl cp `pod-name`:`/path/to/file.log` `file.log` # Copies a file from a pod to the local machine.
- `kubectl api-resources` | tail +2 | awk ' { print $1 }'`; do kubectl explain $kind; done | grep -e "KIND:" -e "VERSION:"
- kubectl drain `node_name` # this will make the node unshedulable and make the pods spawn on other nodes
- kubectl uncordon `node_name` # this will make the node schedulable
- kubectl cordon `node_name` # this will make the node unschedulable

---

# [Labels and Selectors](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/)

- Labels are the way by which you can filter and sort the number pods in a cluster
- selector tells the replicaset/controller/deployment which pods to watch/belongs to
- The label selector is the core grouping primitive in Kubernetes.
- Labels are usedful for filtering the environment specific pods
- You can assign multiple labels to a pod, node, cluster objects.
- By giving multiple labels you can filter out exact pods that you required.
- Labels are key=value pair without any predefined meaning
- They are similar to tags in AWS and Git
- You are free to choose labels of any name.
- You can provide label through both imperative and declarative methods.
- Define it under the **metadata**
- **Annotations** are extra details for the pod

---

```
---
apiVersion: apps/v1
kind: Deployment
metadata:
    name: myDeployment

    # This are the Deployment labels 
    labels:
        app: myApp
        env: dev
spec:
   replicas: 3

   # the Selector selects the pod with specific labels
   selector: 
      matchLabels:
      tier: front-end
   
   # template  defines the PodSpec for the new pods being created and what Labels it contains
   template:
      metadata: 
      name: testpod
      labels:
          tier: front-end
      spec:
      containers:
      - name: c00
          image: nginx:latest
```

---

# [StatefulSets](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/)

- StatefulSet is a workload API object used to manage stateful applications.
- A StatefulSet runs a group of Pods, and maintains a sticky identity for each of those Pods. This is useful for managing applications that need persistent storage or a stable, **unique network identity**
- Manages the deployment and scaling of a set of Pods, and provides guarantees about the ordering and uniqueness of these Pods
- StatefulSet maintains a sticky identity for each of its Pods.
- Although individual Pods in a StatefulSet are susceptible to failure, the persistent Pod identifiers make it easier to match existing volumes to the new Pods that replace any that have failed.
- **StatefulSets are valuable for applications that require one or more of the following.**
  - Stable, unique network identifiers.
  - Stable, persistent storage.
  - Ordered, graceful deployment and scaling.
  - Ordered, automated rolling updates
- The storage for a given Pod must either be provisioned by a PersistentVolume Provisioner ([examples here](https://github.com/kubernetes/examples/blob/master/staging/persistent-volume-provisioning/README.md)) based on the requested storage class, or pre-provisioned by an admin.
- Deleting and/or scaling a StatefulSet down will not delete the volumes associated with the StatefulSet.
- StatefulSets currently require a Headless Service to be responsible for the network identity of the Pods
- StatefulSets do not provide any guarantees on the termination of pods when a StatefulSet is deleted. To achieve ordered and graceful termination of the pods in the StatefulSet, it is possible to scale the StatefulSet down to 0 prior to deletion.

```
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None
  selector:
    app: nginx
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  selector:
    matchLabels:
      app: nginx # has to match .spec.template.metadata.labels
  serviceName: "nginx"
  replicas: 3 # by default is 1
  minReadySeconds: 10 # by default is 0
  template:
    metadata:
      labels:
        app: nginx # has to match .spec.selector.matchLabels
    spec:
      terminationGracePeriodSeconds: 10
      containers:
      - name: nginx
        image: registry.k8s.io/nginx-slim:0.24
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "my-storage-class"
      resources:
        requests:
          storage: 1Gi

```

## Volume Claim Templates

- You can set the `.spec.volumeClaimTemplates` field to create a `PersistentVolumeClaim`. 
- StatefulSet Pods have a **unique identity** that consists of an ordinal, a stable network identity, and stable storage. The identity sticks to the Pod, regardless of which node it's (re)scheduled on.

## StatefulSets Diagrams

![alt text](statefulset_pvc.png)
![alt text](image-19.png)

## StatefulSets vs Deployment

# [PersistentVolume (PV) Provisioning](https://github.com/kubernetes/examples/blob/master/staging/persistent-volume-provisioning/README.md)

- A PersistentVolume is a piece of storage in the Kubernetes cluster that has been *provisioned by an administrator* or *dynamically using StorageClasses*.
- The admin must define StorageClass objects that describe named "classes" of storage offered in a cluster.
- When configuring a StorageClass object for persistent volume provisioning, the admin will need to describe the type of **provisioner** to use and the **parameters** that will be used by the provisioner when it provisions a PersistentVolume belonging to the class.
- The provisioner field must be specified as it determines what `volume plugin` is used for provisioning PVs.
- The parameters field contains the parameters that describe volumes belonging to the storage class. Different parameters may be accepted *depending on the provisioner*.

**Key Attributes:**
  - **Capacity**: The size of the storage.
  - **Access Modes**: Defines how the volume can be mounted (e.g., ReadWriteOnce, ReadOnlyMany, ReadWriteMany).
  - **Reclaim Policy**: Determines what happens to the PV after the PVC is deleted (e.g., Retain, Recycle, Delete).
  - **StorageClass**: Links the PV to a StorageClass for dynamic provisioning.

AWS EBS PV
```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: ebs-sc
provisioner: ebs.csi.aws.com
volumeBindingMode: WaitForFirstConsumer
parameters:
  csi.storage.k8s.io/fstype: xfs
  type: io1
  iopsPerGB: "50"
  encrypted: "true"
allowedTopologies:
- matchLabelExpressions:
  - key: topology.ebs.csi.aws.com/zone
    values:
    - us-east-2c
```
```
---
apiVersion: v1
kind: PersistantVolume
metadata:
  name: example-volume
  labels:
    type: local
spec:
  storageClassName: ebs-sc
  capacity: 
    storage: 50Gi
  accessModes:
    - ReadWriteOnce
  hostpath:
    path: "/mnt/data"
```

# PersistentVolumeClaim (PVC)

- A PersistentVolumeClaim is a request for storage by a user. It specifies size, access modes, and optionally, a StorageClass. Kubernetes matches PVCs to PVs based on these specifications

**Key Attributes**:
1. **Requested Storage**: The size of the storage needed.
2. **Access Modes**: How the storage should be accessed.
3. **StorageClass**: (Optional) Specifies the desired StorageClass for dynamic provisioning.

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-example
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
  storageClassName: ebs-sc # Optinal
```

# [StorageClass](https://kubernetes.io/docs/concepts/storage/storage-classes/)

- A StorageClass provides a way to describe the "classes" of storage available in a Kubernetes cluster. 
- If you are leveraging dynamic provisioning, you should create the StorageClass first. This is because the StorageClass is responsible for dynamically provisioning the PersistentVolumes when a PersistentVolumeClaim (PVC) is created.
- A StorageClass provides a way for administrators to describe the classes of storage they offer
- It abstracts the underlying storage provider (like AWS EBS, GCE PD, NFS, etc.) and allows dynamic provisioning of PVs.

**Key Attributes**:
  1. **Provisioner**: Specifies the volume plugin (e.g., kubernetes.io/aws-ebs, kubernetes.io/gce-pd, kubernetes.io/nfs).
  2. **Parameters**: Configuration options specific to the provisioner.
  3. **Reclaim Policy**: Defaults for dynamically provisioned PVs.

```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: ebs-sc
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp2
  zones: us-east-1a,us-east-1b
reclaimPolicy: Retain
```

**Workflow**:

  - **Create the StorageClass**: Define the storage characteristics and provisioner.
  - **Create the PVC**: Reference the StorageClass in the PVC specification.
  - **Kubernetes Automatically Creates the PV**: Kubernetes uses the provisioner to create the storge.

### Static vs Dynamic provisioning of PV in K8s

1. **Dynamic Provisioning**
- **StorageClass First**: If you're using dynamic provisioning, you should create the StorageClass first. This is because when a PersistentVolumeClaim (PVC) is created with a storageClassName, Kubernetes uses the StorageClass to dynamically provision a PersistentVolume that matches the PVC's requirements.
- **PVC Creation Triggers PV Creation**: After the StorageClass is defined, the PVC can be created. Kubernetes will then automatically create a PV based on the specifications in the StorageClass and bind it to the PVC.

**Order**:
  1. StorageClass
  2. PersistentVolumeClaim (PVC)

2. **Static Provisioning**
- **PersistentVolume First**: In the case of static provisioning, you manually create the PVs first. These PVs are pre-provisioned by an administrator and are available in the cluster for any PVC to claim.
- **PVC Creation Binds to PV**: After the PVs are created, you can create PVCs that match the specifications of the available PVs. Kubernetes will bind the PVC to a matching PV.

**Order**:
  1. PersistentVolume (PV)
  2. PersistentVolumeClaim (PVC)
---

## Imperative Commands to find pods with selected Labels

`kubectl get pods --selector key=value --no-headers | wc -l`

`kubectl get pods --selector env=prod,env=prod,tier=frontend`

# [Taints and Tolerence](https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/)

- Taints are placed on Nodes
- Tolerations is placed on Pods
- Tolerations allow the scheduler to schedule pods with matching taints.
- if Taint are placed on the specific node it will not allow a pod without tolerence tobe scheduled on that Node
- if we place the tolerence on the specific pod then only it will be able to scheduled on the node else not
- Taints are the opposite -- they allow a node to repel a set of pods.
- Taints and tolerations work together to ensure that pods are not scheduled onto **inappropriate** nodes.
- One or more taints are applied to a node; this marks that the node should not accept any pods that do not tolerate the taints
- Taints and Tolerations does not tell pod to go on specific node
- *Taints and Tolerations only tells accept pods on the node with certain tolerations.*
- if you want to restrict a certain pod on a single node then you have to use the node **Affinity**.
- Taint is alredy placed on masterNode which tells the scduler not to schedule pods on the masterNode to view which taint effect is applied on the master node use below command.
  - `kubectl describe node kubemaster | grep Taint`

## Taint-Effects

- **NoExecute**
  - Pods that do not tolerate the taint are evicted immediately
  - Pods that tolerate the taint without specifying `tolerationSeconds` in their toleration specification remain bound forever
  - Pods that tolerate the taint with a specified `tolerationSeconds` remain bound for the specified amount of time. After that time elapses, the node lifecycle controller evicts the Pods from the node
  - Pods currently running on the node are **evicted** if does not match the tolerence.
  - `NoExecute` effect can specify an optional `tolerationSeconds` field that dictates how long the pod will stay bound to the node after the taint is added
- **NoSchedule**
  - No new Pods will be scheduled on the tainted node unless they have a matching toleration. Pods currently running on the node are not evicted.
- **PreferNoSchedule**
  - `PreferNoSchedule` is a "preference" or "soft" version of `NoSchedule`.
  - The control plane will try to avoid placing a Pod that does not tolerate the taint on the node, *but it is not guaranteed*.

---

### Commands: (Applying Tolerence to Node:)

To Apply the Taint on the Node

`kubectl taint nodes node1 key1=value1:NoSchedule`

To  remove the taint from the node:

`kubectl taint nodes node1 key1=value1:NoSchedule-`

```
kubectl taint nodes node1 key1=value1:NoSchedule
kubectl taint nodes node1 key1=value1:NoExecute
kubectl taint nodes node1 key2=value2:NoSchedule
```

### Applying Tolerence to Pod

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
      env: test
spec:
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent

# add tolerence to Pod
  tolerations:
  - key: "key1"
    operator: "Equal"
    value: "value1"
    effect: "NoSchedule"
```

---

# [Node Selectors](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/)

- Node selectors helps in scheduling the pod on specific node based on labels assigned to node and the pods
- suppose we have Three nodes namely Large, Small with labels as size = large/small.
- We can use this label information to schedule our pods accordingly.

Example :

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
      env: test
spec:
  containers:
  - name: nginx
    image: nginx

# Apply node selector to run pod on node with size=large label on it
  nodeSelector:
    size: large
```

## Commands

Add a label to a particular node

`kubectl label nodes <node_name> key=value`

### Limitations
>
> NodeSelector will not be able to help in complex senerios such as
>
> - size=large or size=small
> - size !=medium (not equal)

---

# [Node Affinity](https://kubernetes.io/docs/tasks/configure-pod-container/assign-pods-nodes-using-node-affinity/)

- with the help of Node Affinity , you can control the scheduling of pods  onto nodes more granularly using various criteria like
- It is an extension of node selectors which allows you to specify more flexible rules about where to place your pods by specifying certain requirements.

Example: This manifest describes a Pod that has a `requiredDuringSchedulingIgnoredDuringExecution` node affinity,`size=large or size=small`. This means that the pod will get scheduled only on a node that has a `size=large or size=small`

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: size
            operator: In
            values: # The Pod will scheduled on the nodes that have at least one of these value.
            - large
            - small            
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent
```

## Node Affinity types

1. **`required`DuringScheduling`Ignored`DuringExecution**
     - Pods with this affinity rule will only be scheduled on nodes that meet the specified requirements. However, if a node loses the label that made it eligible after the pod is scheduled, the pod will still continue to run on that node.
2. **`Preferred`DuringScheduling`Ignored`DuringExecution**
     - Pods with this affinity rule will prefer to be scheduled on nodes that meet the specified requirements, but they are not strictly required to. If no nodes match the preferred requirements, the pod can still be scheduled on other nodes.
  
| Syntax | DuringScheduling | DuringExecution |
| ----------- | ----------- | --------------- |
| Type1 | Required | Ignored |
| Type2 | preferred | Ignored |
| Type3 | Required | Required |

### Taints and Tolerence & Node Affinity

- You can use both  taints and tolerations & Node affinity to granular control over the Pod scheduling on the specific nodes.

- **what if we applied taint to specific node and Node Affinity to a pod to be schedule on the specific node only without tolerance set what will happen?**
  - When the scheduler attempts to place the pod onto a node, it evaluates the Node Affinity rules against the labels of each node in the cluster.
  - If there is a node that meets the Node Affinity requirements specified in the pod's configuration and doesn't have a taint that repels the pod (i.e., the node matches the Node Affinity and doesn't have any taints), the pod will be scheduled on that node.
  - However, if the node with the matching Node Affinity also has the taint that the pod lacks tolerations for, the pod won't be scheduled on that node due to the taint-repelling behavior. In this case, the pod won't be scheduled anywhere unless there is another node in the cluster that meets the Node Affinity requirements and doesn't have the taint applied.

---

# [Resource Requests and Limits](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/)

- **Requests**: Requests specify the minimum amount of resources (CPU and memory) that a pod needs to run.
  - When a pod is scheduled onto a node, Kubernetes ensures that the node has enough available resources to satisfy the pod's resource requests.
  - Requests are used by the **scheduler** to determine the best node for placing a pod.

- **Limits**: Limits, on the other hand, specify the maximum amount of resources that a pod can consume.
  - Kubernetes enforces these limits to prevent pods from using more resources than allowed, which helps in maintaining stability and preventing resource contention.
  - If a pod exceeds its specified limits, Kubernetes may take actions such as throttling or terminating the pod.
  - Pod cannot exceed the CPU limit but can exceed the Memory Limit in that case K8s will try to Terminate or kill the Pod when it finds that the pod is using more resources than allowed `OOM` (out of memory (OOM) error) `OOMKilled` means the pod has been killed by Out of Memory.
  - if you donot specify the Request the Limit will be `Request = Limit`

## Use Cases

1. **No Request and No Limit**

- This scenario is suitable for non-critical applications or experiments where resource requirements are flexible, and the pod can utilize whatever resources are available on the node without any constraints. However, it can lead to potential resource contention issues if other pods on the node require resources.

2. **No Request and Limit**

- This scenario is useful when you want to constrain the resource usage of a pod to prevent it from using excessive resources and impacting other pods on the node. However, without a request, Kubernetes scheduler may place the pod on a node without considering its resource needs, potentially leading to inefficient resource allocation.

3. **Request and Limit**

- This is the most common and recommended configuration. By specifying both requests and limits, Kubernetes can ensure that the pod gets scheduled on a node with sufficient resources and enforce resource constraints to maintain stability and fairness within the cluster.

4. **Request and No Limits**

- This scenario is suitable when you want to ensure that the pod gets scheduled on a node with sufficient resources based on its requirements, but you're not concerned about limiting its resource usage. It allows the pod to scale up its resource consumption based on demand without strict constraints, which can be useful for certain workloads with variable resource requirements.

---

```
---
apiVersion: v1
kind: Pod
metadata:
  name: frontend
spec:
  containers:
  - name: app
    image: images.my-company.example/app:v4
    resources:
      requests:
        memory: "64Mi"
        cpu: "1"
      limits:
        memory: "128Mi"
        cpu: "500m"
```

**Note**:
>Kubernetes doesn't allow you to specify CPU resources with a precision finer than `1m or 0.001 CPU`. To avoid accidentally using an invalid CPU quantity, it's useful to specify CPU units using the milliCPU form instead of the decimal form when using less than `1 CPU` unit.
>For example, you have a Pod that uses `5m or 0.005 CPU` and would like to decrease its CPU resources. By using the decimal form, it's harder to spot that `0.0005 CPU` is an invalid value, while by using the milliCPU form, it's easier to spot that `0.5m` is an invalid value.
>Limits and requests for `memory` are measured in bytes. You can express memory as a plain integer or as a fixed-point number using one of these quantity suffixes: E, P, T, G, M, k. You can also use the power-of-two equivalents: Ei, Pi, Ti, Gi, Mi, Ki. For example, the following represent roughly the same value:
  `128974848, 129e6, 129M,  128974848000m, 123Mi`
>Pay attention to the case of the suffixes. If you request `400m` of memory, this is a request for 0.4 bytes. Someone who types that probably meant to ask for 400 mebibytes (`400Mi`) or 400 megabytes (`400M`).
---

# [DeamonSet](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/)

- DeamonSet are simillar to ReplicaSets but inly difference is that DeamonSet ensure that every node has at  least one instance of a pod running.
- when the node gets created DeamonSet automatically creates the pod on the Node.
- Pods created by the DeamonSet are ignored by the Kube-scheduler.
- deamonSets does not support scalling inside the Node
- it supports the Rolling update like Deployments.

## Use Cases

  - running a cluster storage daemon on every node
  - running a *logs collection* daemon on every node
  - running a *node monitoring* daemon on every node
- one DaemonSet, covering all nodes, would be used for each type of daemon.
![alt text](image.png)

```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd-elasticsearch
  namespace: kube-system
  labels:
    k8s-app: fluentd-logging
spec:
  selector:
    matchLabels:
      name: fluentd-elasticsearch
  template:
    metadata:
      labels:
        name: fluentd-elasticsearch
    spec:
      tolerations:
      # these tolerations are to have the daemonset runnable on control plane nodes
      # remove them if your control plane nodes should not run pods
      - key: node-role.kubernetes.io/control-plane
        operator: Exists
        effect: NoSchedule
      - key: node-role.kubernetes.io/master
        operator: Exists
        effect: NoSchedule
      containers:
      - name: fluentd-elasticsearch
        image: quay.io/fluentd_elasticsearch/fluentd:v2.5.2
        resources:
          limits:
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 200Mi
        volumeMounts:
        - name: varlog
          mountPath: /var/log
      # it may be desirable to set a high priority class to ensure that a DaemonSet Pod
      # preempts running Pods
      # priorityClassName: important
      terminationGracePeriodSeconds: 30
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
```

`kubectl apply -f DeamonSet.yml`

## Running Pods on select Nodes
>
>If you specify a `.spec.template.spec.nodeSelector`, then the DaemonSet controller will create Pods on nodes which match that **node selector**. Likewise if you specify a `.spec.template.spec.affinity`, then DaemonSet controller will create Pods on nodes which match that **node affinity**. If you do not specify either, then the DaemonSet controller will create Pods on all nodes.
---

# [Static Pods](https://kubernetes.io/docs/tasks/configure-pod-container/static-pod/)

- Static Pods are managed directly by the **kubelet daemon** on a specific node, without the API server observing them.
- kube-Scheduler ignore the Static pods
- Static pod are used to depoly the Control plane pods
- static pods are only created by kubelet.
- Static Pods are always bound to one Kubelet on a specific node.
- to Identify the Static pods, they always have a nodeName at the end. ex- **etcd-`controlplane`**, **kube-apiserver-`controlplane`**, **kube-controller-manager-`controlplane`**, **kube-scheduler-`controlplane`** or you can refer to the `OwerReference`in the Pod defination file.
- Kubelet does not manage the Deployment and Replicasset it only manages the Pod
- Static pods are visible to the Kube-Apiserver, but does not control from there
- Kubelet also runs on the master node in Kubernetes to provision the master components as static pods.
  
**There are two way to craete a Static pod**

1. **Filesystem-hosted static Pod manifest**
2. **Web-hosted static pod manifest**

- Manifests are standard Pod definitions in JSON or YAML format in a specific directory. Use the `staticPodPath: <the directory>` field in the [kubelet configuration file](https://kubernetes.io/docs/reference/config-api/kubelet-config.v1beta1/), which periodically scans the directory and creates/deletes static Pods as YAML/JSON files **appear/disappear there**.
- to modify the kubelet use the [`KubeletConfiguration`](https://kubernetes.io/docs/tasks/administer-cluster/kubelet-config-file/) YAML file which is the recommended way.

```
apiVersion: kubelet.config.k8s.io/v1beta1
kind: KubeletConfiguration
address: "192.168.0.8"
port: 20250
serializeImagePulls: false
evictionHard:
    memory.available:  "100Mi"
    nodefs.available:  "10%"
    nodefs.inodesFree: "5%"
    imagefs.available: "15%"
    staticPodPath: "/etc/kubernetes/manifests"
```

to apply this file

`kubectl apply -f kubelet-config.yaml`

- `staticPodPath: "/etc/kubernetes/manifests"` is were the static-pod manifest are stored this path is not same always *refer the kubelet config file for the mention path*
  - `cat /var/lib/kubelet/config.yaml` look for `staticPodPath`

  or

- `--pod-manifest-path=/etc/kubernetes/manifests/` **(command line method for creating the static pod)**
  - `systemctl restart kubelet`
- `kubectl run --restart=Never --image=busybox static-busybox --dry-run=client -o yaml --command -- sleep 1000 > /etc/kubernetes/manifests/static-busybox.yaml`

```
controlplane ~ ➜  ls -lrt /etc/kubernetes/manifests/
total 16
-rw------- 1 root root 2406 Apr  7 16:35 etcd.yaml
-rw------- 1 root root 1463 Apr  7 16:35 kube-scheduler.yaml
-rw------- 1 root root 3393 Apr  7 16:35 kube-controller-manager.yaml
-rw------- 1 root root 3882 Apr  7 16:35 kube-apiserver.yaml
```

- to ssh into any node within k8s cluster you can use the command `ssh <Node-name>/<Node-IP>`

# [Multiple Schedulers](https://kubernetes.io/docs/tasks/extend-kubernetes/configure-multiple-schedulers/)

- K8s comes with a single default scheduler but as the k8s is highly extensible, If the default scheduler does not suit your needs you can **implement your own scheduler**.
- you can even run multiple schedulers simultaneously alongside the default scheduler.

---
**STEP 1:**

- Package your scheduler binary into a container image.
- Clone the [Kubernetes](https://github.com/kubernetes/kubernetes.git) source code from GitHub and build the source.
- `Docker Build -t` and `Docker Push` to push the Image to Docker Hub (or another registry). refer k8s documentation

---
**Docker File**

```
FROM busybox
ADD ./_output/local/bin/linux/amd64/kube-scheduler /usr/local/bin/kube-scheduler
```

---
**STEP 2:** Define a Kubernetes Deployment for the scheduler

- you have your scheduler in a container image
- create a pod/Deployment configuration for it and run it in your Kubernetes cluster.
- Save it as [my-scheduler.yaml](https://raw.githubusercontent.com/kubernetes/website/main/content/en/examples/admin/sched/my-scheduler.yaml)
- in the deployment use this

```
---
# **my-scheduler-config.yaml**
apiVersion: kubescheduler.config.k8s.io/v1
kind: KubeSchedulerConfiguration
profiles:
  - schedulerName: my-scheduler
leaderElection:
  leaderElect: false 

---
# **my-scheduler-configmap.yaml**
apiVersion: v1
data:
  my-scheduler-config.yaml: |
    apiVersion: kubescheduler.config.k8s.io/v1
    kind: KubeSchedulerConfiguration
    profiles:
      - schedulerName: my-scheduler
    leaderElection:
      leaderElect: false
kind: ConfigMap
metadata:
  creationTimestamp: null
  name: my-scheduler-config
  namespace: kube-system   

---
# **my-scheduler.yml**
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: my-scheduler
  name: my-scheduler
  namespace: kube-system
spec:
  serviceAccountName: my-scheduler
  containers:
  - command:
    - /usr/local/bin/kube-scheduler
    - --config=/etc/kubernetes/my-scheduler/my-scheduler-config.yaml
    image: registry.k8s.io/kube-scheduler:v1.29.0
    livenessProbe:
      httpGet:
        path: /healthz
        port: 10259
        scheme: HTTPS
      initialDelaySeconds: 15
    name: kube-second-scheduler
    readinessProbe:
      httpGet:
        path: /healthz
        port: 10259
        scheme: HTTPS
    resources:
      requests:
        cpu: '0.1'
    securityContext:
      privileged: false
    volumeMounts:
      - name: config-volume
        mountPath: /etc/kubernetes/my-scheduler
  hostNetwork: false
  hostPID: false
  volumes:
    - name: config-volume
      configMap:
        name: my-scheduler-config
```

---
**STEP 3:** How to use the custom schduler into pod definations.

```
---
apiVersion: v1
kind: Pod
metadata:
  name: frontend
spec:
  containers:
  - name: nginx
    image: nginx
  
  # This is how you can add the custom scheduler to pod.
  schedulerName: my-scheduler
```

**Example**:

```
controlplane ~ ➜  kubectl get pods -A
NAMESPACE      NAME                                   READY   STATUS    RESTARTS   AGE
kube-flannel   kube-flannel-ds-nssdp                  1/1     Running   0          10m
kube-system    coredns-69f9c977-25wxh                 1/1     Running   0          10m
kube-system    coredns-69f9c977-9v84m                 1/1     Running   0          10m
kube-system    etcd-controlplane                      1/1     Running   0          11m
kube-system    kube-apiserver-controlplane            1/1     Running   0          11m
kube-system    kube-controller-manager-controlplane   1/1     Running   0          10m
kube-system    kube-proxy-zhj22                       1/1     Running   0          10m
kube-system    kube-scheduler-controlplane            1/1     Running   0          10m
kube-system    **my-scheduler**                           1/1     Running   0          2m22s
```

---
`kubectl get events -o wide`

`kubectl logs <scheduler_name> -n <name_space>`

---

# [Scheduler Profiles](https://kubernetes.io/docs/concepts/scheduling-eviction/pod-priority-preemption/)

- Suppose if you have multiple schedulers you can combine all those schedulers to use same binary rather than creating multiple binaries, this can be achived with the help of Scheduler Profile in latest k8s versions.
- this is a efficient way to use and configure mutiple schedulers in k8s cluster.
- How a Pod gets scheduled on the nodes
  - **Sceduling Queue**
  - **Filtering**
  - **Scoring**
  - **Binding**

![alt text](<Screenshot 2024-04-20 at 13-52-52 Certified Kubernetes Administrator (CKA) Practice Exam Tests.png>)

## Scheduling Queue

### PriorityClass

- you can set the priority of pod tobe scheduled

```
---
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority
value: 1000000
globalDefault: false
description: "This priority class should be used for XYZ service pods only."

---
apiVersion: v1
kind: Pod
metadata:
  name: frontend
spec:
  priorityClass: high-priority
  containers:
  - name: nginx
    image: nginx
  resources:
    requests:
      memory: 1Gi
      cpu: 10
```

## [Filtering](https://kubernetes.io/docs/concepts/scheduling-eviction/kube-scheduler/#kube-scheduler-implementation)

- Filter which nodes are not  suitable for running the pod, e.g., lack of resource or taints and tolerations

## [Scoring](https://discuss.kubernetes.io/t/kube-pod-scoring-criteria/20743)

- after the filtering phase Nodes which are eligible will go through the scoring phase . The node with the highest score is selected.
- Score means Suitable to run the Pod and have more available free resource after scheduling.
- The scheduler assigns a score to each Node that survived filtering, basing this score on the active scoring rules.

## Binding

- The nodes which have a good score are selected for binding.

### [Sceduler Plugins](https://kubernetes.io/docs/concepts/scheduling-eviction/scheduling-framework/#interfaces)

- **Scheduling Queue**
  - PrioritySort
- **Filtering**
  - NodeResourcesFit
  - NodeName
  - NodeUnschedule
- **Scoring**
  - NodeResourcesFit
  - ImageLocality
- **Binding**
  - DefaultBinder

These Plugins are pluged to **Extention points**

1. queueSort
2. filter
3. score
4. bind

---

# [Logging and Monitoring the Kubernetes Components:](https://kubernetes.io/docs/tasks/debug/)

## Monitoring The k8s Cluster

- `Metric Server` is available on each node to gather logs.
- Metric server can be enabled using `minikube addons enable metrics-server` on minikube only
- for Others to deploy **metric server**
  - `git clone https://github.com/kubernetes-incubator/metrics-server.git`
  - `kubectl create -f deploy/1.8+/`
  - `kubectl top node` to view resource consumption of cluster.

```
controlplane kubernetes-metrics-server on  master ➜  kubectl top node
NAME           CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%   
controlplane   258m         0%     1084Mi          0%        
node01         118m         0%     268Mi           0%        
```

- `kubectl top pod` to view resources consumption of pods.

```
controlplane kubernetes-metrics-server on  master ➜  kubectl top pod
NAME       CPU(cores)   MEMORY(bytes)   
elephant   15m          32Mi            
lion       1m           18Mi            
rabbit     102m         14Mi            
```

## Logging in K8s

- to view logs of a pod
  - `kubectl logs <pod-name>`
- to view logs of a pod in a specific container
  - `kubectl logs <pod-name> -c <container-name>`
- to view logs of a pod in a specific container and follow the logs
- `kubectl describe pod` to find the container name

```
Name:         my-pod
Namespace:    default
Priority:     0
Node:         node1/10.0.0.1
Start Time:   Tue, 01 Mar 2023 10:00:00 +0000
Labels:       app=my-app
Annotations:  <none>
Status:       Running
IP:           10.0.0.2
IPs:
  IP:  10.0.0.2
Containers:
  **my-container:** 
    Container ID:   docker://abcdefg1234567890
    Image:          my-image:latest
    Image ID:       docker-pullable://my-image@sha256:abcdefg1234567890
    Port:           8080/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Tue, 01 Mar 2023 10:00:01 +0000

```

# Application Lifecycle Management in K8s

## Rollout and Versioning

- Users expect applications to be available all the time, and developers are expected to deploy new versions of them several times a day. In Kubernetes this is done with rolling updates
- A **rollinrg update** allows a Deployment update to take place with **zero downtime**
- It does this by incrementally replacing the current Pods with new ones.
- Kubernetes waits for those new Pods to start before removing the old Pods.
- In Kubernetes, updates are versioned and any Deployment update can be reverted to a previous (stable) version.
- the Service will load-balance the traffic only to available Pods during the update.
- when updates happens the new set of replicas are created one by one and the old set of replicas are down to 0 one by one in **rolling update**. when doing the undo old ones are up and new ones are down to zero one by one.

```
> kubectl get replicasets
NAME DESIRED CURRENT READY AGE
myapp-deployment-67c749c58c 0 0 0 22m
myapp-deployment-7d57dbdb8d 5 5 5 20m
```

**Rolling updates allow the following actions:**

1. Promote an application from one environment to another (via container image updates)
2. Rollback to previous versions
3. Continuous Integration and Continuous Delivery of applications with zero downtime

![alt text](image-1.png)

to update the image

- `kubectl set image deployment/<deployment-name> <container-name>=<new-image>`

To view the status of application upgrade

- `kubectl rollout status deployment/<deployment-name>`

```
> kubectl rollout status deployment/myapp-deployment
Waiting for rollout to finish: 0 of 10 updated replicas are available...
Waiting for rollout to finish: 1 of 10 updated replicas are available...
Waiting for rollout to finish: 2 of 10 updated replicas are available...
Waiting for rollout to finish: 3 of 10 updated replicas are available...
Waiting for rollout to finish: 4 of 10 updated replicas are available...
Waiting for rollout to finish: 5 of 10 updated replicas are available...
Waiting for rollout to finish: 6 of 10 updated replicas are available...
Waiting for rollout to finish: 7 of 10 updated replicas are available...
Waiting for rollout to finish: 8 of 10 updated replicas are available...
Waiting for rollout to finish: 9 of 10 updated replicas are available...
deployment "myapp-deployment" successfully rolled out
```

To view the history of application upgrade

- `kubectl rollout history deployment/<deployment-name>`

```
> kubectl rollout history deployment/myapp-deployment
deployments "myapp-deployment"
REVISION CHANGE-CAUSE
1        <none>
2        kubectl apply --filename=deployment-definition.yml --record=true
```

To rollback to a previous version

- `kubectl rollout undo deployment/<deployment-name>`

## Deployment Strategy

- The Deployment controller supports two different strategies for rolling updates:

1. **Recreate**
   - In this strategy, all existing Pods are killed one at a time, and new ones are created to replace them.
   - In the strategy user will faces the downtime of application during the update.

2. **Rolling Update**
   - In this strategy, new Pods are created and the old ones are killed one at a time.
   - In this strategy, user will not faces the downtime of application during the update.

![alt text](image-2.png)

![alt text](image-3.png)  ![alt text](image-4.png)

There are two ways to update the deployment

1. **Manually**
   - The user can manually update the deployment by changing the image version in the deployment manifest. `kubectl edit deployment <deployment-name>`
   - or using commandline `kubectl set image deployment/<deployment-name> <container-name>=<new-image-name>`

2. **Automatically**
   - The user can also update the deployment automatically by using the image update policy.

## Application Commands, EntryPoint and Arguments in Pod defination

### Docker

Difference Between `ENTRYPOINT` and `CMD` in Docker and use case.

- **`CMD`** is used to run the command when the container is started.
  - The `CMD` instruction specifies the default command or arguments that will be executed by the ENTRYPOINT if no other command is provided when running the container.
  - If you use both `ENTRYPOINT` and `CMD` in a Dockerfile, the CMD instruction will provide the default arguments for the `ENTRYPOINT`.
- **`ENTRYPOINT`** is used to run the command when the container is started.
  - It is the preferred way to define the main command for a Docker container.
  - If you provide additional arguments when running the container, they will be appended to the `ENTRYPOINT` instruction.
  - The ENTRYPOINT cannot be overridden by arguments provided during container runtime unless you use the `--entrypoint` flag.

Command:
`docker run ubuntu [COMMAND]`

![alt text](image-5.png)

**Example:**

1.

```
# Example 1: Using ENTRYPOINT
FROM ubuntu
ENTRYPOINT ["sleep"]

# If you run: docker run my-image 10
# It will execute: sleep 10

# If you run: docker run my-image sleep2.0 10
# It willnot execute: sleep 10 (cannot replace sleep the default ENTRYPOINT) need to use `--entrypoint sleep2.0` to replace it in commandline

```

`docker run --entrypoint sleep2.0 my-image 10`

2.

```
# Example 2: Using CMD
FROM ubuntu
CMD ["sleep", "5"]

# If you run: docker run my-image
# It will execute: sleep 5

# If you run: docker run my-image 10
# It will execute: sleep 10 (overriding the default CMD)

# If you run: docker run my-image sleep2.0 10
# It will execute: sleep2.0 10 (replace the default CMD)

```

3.

```
# Example 3: Using ENTRYPOINT and CMD together
FROM ubuntu
ENTRYPOINT ["sleep"]
CMD ["5"]

# If you run: docker run my-image
# It will execute: sleep 5 (CMD provides the default argument)

# If you run: docker run my-image 10
# It will execute: sleep 10 (additional arguments are appended)
```

### POD

- we can use `command` to override the `ENTRYPOINT` from pod defination,works like `--entrypoint`
- we can use `args` to override the `CMD` from pod defination

![alt text](image-6.png)

Example 1 of `Command` and `args`

```
---
apiVersion: v1
kind: Pod
metadata:
  name: frontend
spec:
  priorityClass: high-priority
  containers:
  - name: nginx
    image: nginx
    command: ["sleep", "5000"]
# OR
    command:
    - "sleep"
    - "5000"
```

Example 2 of `Command` and `args`

```
---
apiVersion: v1
kind: Pod
metadata:
  name: frontend
spec:
  priorityClass: high-priority
  containers:
  - name: nginx
    image: nginx
    command: ["sleep"]
    args: ["5000"]
```

>**Note**:
> Both the `command` and `args` need to string not number

# [Config Map](ConfigMaps)

## What is a ConfigMap

- ConfigMap can pass the key value pair to the pod
- ConfigMap can be used to pass the configuration to the pod
- it helps in managing the environment variables in the pod defination centrally
- A ConfigMap is an API object used to store non-confidential data in key-value pairs
- Pods can consume ConfigMaps as environment variables, command-line arguments, or as configuration files in a volume.
- The Pod and the ConfigMap must be in the same namespace.
- ConfigMap does not provide secrecy or encryption
- A ConfigMap allows you to decouple environment-specific configuration from your container images, so that your applications are easily portable.'
- a ConfigMap has `data` and `binaryData` fields rather than spec.
- The data field is designed to contain `UTF-8` strings while the binaryData field is designed to contain binary data as `base64-encoded` strings.
- **The Kubernetes feature `Immutable` Secrets and ConfigMaps provides an option to set individual Secrets and ConfigMaps as immutable**.
  - protects you from **accidental** (or unwanted) updates that could cause applications outages
  - **improves performance** of your cluster by significantly reducing load on kube-apiserver, by closing watches for ConfigMaps marked as immutable.

```
apiVersion: v1
kind: ConfigMap
metadata:
  ...
data:
  ...
immutable: true
```

>**Note**: Once a ConfigMap is marked as `immutable`, it is not possible to revert this change nor to mutate the contents of the data or the binaryData field. You can only delete and recreate the ConfigMap. Because existing Pods maintain a mount point to the deleted ConfigMap, it is recommended to recreate these pods.

>**Note**: A ConfigMap is not designed to hold large chunks of data. The data stored in a ConfigMap cannot exceed `1 MiB`. If you need to store settings that are larger than this limit, you may want to consider **mounting a volume** or use a separate database or file service.

### Creating a ConfigMap

- Imperative
  - Adding config map through command line
    - kubectl create configmap `configmap-name` --from-literal=`key=value` --from-literal=`key2=value2`
    - kubectl create configmap `configmap-name` --from-file=`path-to-file`
    - kubectl create configmap `configmap-name` --from-file=`app.config.properties`
- Diclarative
  - Adding through the pod defination

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: game-demo
data:
  # property-like keys; each key maps to a simple value
  player_initial_lives: "3"
  ui_properties_file_name: "user-interface.properties"

  # file-like keys
  game.properties: |
    enemy.types=aliens,monsters
    player.maximum-lives=5    
  user-interface.properties: |
    color.good=purple
    color.bad=yellow
    allow.textmode=true    
```

command :

- `kubectl create -f configmap.yml`
- `kubectl describe configmaps`
- `kubectl get configmaps`

There are four different ways that you can use a ConfigMap to configure a container inside a Pod:

    1. Inside a container command and args
    2. Environment variables for a container
    3. Add a file in read-only volume, for the application to read
    4. Write code to run inside the Pod that uses the Kubernetes API to read ConfigMap

1. Using `env` in pod defination

```
---
apiVersion: v1
kind: Pod
metadata:
  name: frontend
spec:
  priorityClass: high-priority
  containers:
  - name: nginx
    image: nginx
    env:
      - name: USERNAME
        value: admin
      - name: APP_COLOUR
        value: Pink
```

- adding env varibles as `USERNAME = admin` and `APP_COLOUR = pink`in pod defination directly.

1. **Using `valueFrom` in pod defination**

```
---
apiVersion: v1
kind: Pod
metadata:
  name: frontend
spec:
  priorityClass: high-priority
  containers:
  - name: nginx
    image: nginx
    env:                      # Define the environment variable
      - name: APP_COLOUR      # Notice that the case is different here 
                              # from the key name in the ConfigMap.
        valueFrom:            
          configMapKeyRef:
            name: APP         # The ConfigMap this value comes from.
            key: frontend     # The key to fetch.
      - name: UI_PROPERTIES_FILE_NAME
        valueFrom:
          configMapKeyRef:
            name: game-demo                 # different configmap
            key: ui_properties_file_name    # key to fetch the value of
```

- The value of `frontend` will be assigned to `APP_COLOUR` env variable from `APP` configmap
- The value of `ui_properties_file_name` will be assigned to `UI_PROPERTIES_FILE_NAME` env variable from `game-demo` configmap

2. **Using `envFrom` in pod defination**

```
---
apiVersion: v1
kind: Pod
metadata:
  name: frontend
spec:
  priorityClass: high-priority
  containers:
  - name: nginx
    image: nginx
    envFrom:
      - configMapKeyRef:
          name: app-config
```

Use `envFrom` to define all of the ConfigMap's data as container environment variables. The key from the ConfigMap becomes the environment variable name in the Pod.

3. **Mounting the configMap through the `Volumes`.
To consume a ConfigMap in a volume in a Pod:**

- Create a ConfigMap or use an existing one. Multiple Pods can reference the same ConfigMap.
- Modify your Pod definition to add a volume under `.spec.volumes[]`. Name the volume anything, and have a `.spec.volumes[].configMap.name` field set to reference your ConfigMap object.
- Add a `.spec.containers[].volumeMounts[]` to each container that needs the ConfigMap. Specify `.spec.containers[].volumeMounts[].readOnly = true` and `.spec.containers[].volumeMounts[].mountPath` to an unused directory name where you would like the ConfigMap to appear inside the container.
  - If there are multiple containers in the Pod, then each container needs its own `volumeMounts` block, but only **one** `.spec.volumes` is needed per ConfigMap
- Modify your image or command line so that the program looks for files in **that directory**. Each key in the ConfigMap data map becomes the filename under mountPath.
- These volumes can be of various types, such as `emptyDir`, `hostPath`, `persistentVolumeClaim`, `configMap`, or `secret`.
- The `volumeMounts` section allows you to connect the volumes defined in the Pod specification to specific paths within the container's filesystem
- This enables containers to access and store data persistently, share data between containers in the same Pod, or access configuration files and secrets securely.
- Mounted ConfigMaps are updated automatically

```
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - name: mypod
    image: redis
    volumeMounts:
    - name: foo
      mountPath: "/etc/foo"
      readOnly: true
  volumes:
  - name: foo
    configMap:
      name: myconfigmap
```

# [Secrets](https://kubernetes.io/docs/concepts/configuration/secret/)

## What is a Secret

- A Secret is an object that contains a small amount of sensitive data such as a password, a token, or a key.
- Using a Secret means that you don't need to include confidential data in your application code.
- Secrets are similar to ConfigMaps but are specifically intended to hold confidential data.
- A secret is only sent to a node if a pod on that node requires it.
- Kubelet stores the secret into a `tmpfs` so that the secret is not written to disk storage.
- Once the Pod that depends on the secret is deleted, kubelet will delete its local copy of the secret data as well.

## Use Cases

- [Set environment variables for a container.](https://kubernetes.io/docs/tasks/inject-data-application/distribute-credentials-secure/#define-container-environment-variables-using-secret-data)
- [Provide credentials such as SSH keys or passwords to Pods.](https://kubernetes.io/docs/tasks/inject-data-application/distribute-credentials-secure/#provide-prod-test-creds)
- [Allow the kubelet to pull container images from private registries.](https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/)

# How to create a Secret

- Imperative
  - commands
    - kubectl create secret generic `secret-name` --from-literal=`key`=`value`
    - kubectl create secret generic `secret-name` --from-literal=`key`=`value` --from-literal=`key1`=`value1`
    - kubectl create secret generic `secret-name` --from-file=`path-to-file`
    - kubectl create secret generic `app-secret` --from-file=`app-secret.properties`
- Diclarative

```
apiVersion: v1
kind: Secret
metadata:
  name: dotfile-secret
data:
  DB_HOST: dmFsdWUtMg0KDQo= # this is a encoded value
  DB_USER: dmFsdWUtMQ0KDQo= # this is a encoded value
  DB_PASS: dmFsdWUtMl0KDQo= # this is a encoded value
```

To encode a values in a secret use:

```
echo -n 'my-value-to-be-encoded' | base64
```

to decode the values in a secret use:

```
echo -n 'c29tZS12YWx1ZQ==' | base64 --decode
```

1. `envFrom` injecting secret as a Environment variables

```
---
apiVersion: v1
kind: Pod
metadata:
  name: secret-dotfiles-pod
spec:
  containers:
    - name: dotfile-test-container
      image: registry.k8s.io/busybox
      envFrom:
        - secretRef:
            name: dotfile-secret
```

1. `valueFrom` injecting secret as a `Single` Environment variables

```
---
apiVersion: v1
kind: Pod
metadata:
  name: secret-dotfiles-pod
spec:
  containers:
    - name: dotfile-test-container
      image: registry.k8s.io/busybox
      env:
        - name: SECRET_USERNAME
          valueFrom:
            secretKeyRef:
              name: dotfile-secret
              key: username
```

2. Mount secret from a `volume`

```
---
apiVersion: v1
kind: Pod
metadata:
  name: secret-dotfiles-pod
spec:
  containers:
    - name: dotfile-test-container
      image: registry.k8s.io/busybox
      volumeMounts:
        - name: secret-volume
          readOnly: true
          mountPath: "/etc/secret-volume"
  volumes:
    - name: secret-volume
      secret:
        secretName: dotfile-secret
```

> **Notes**:
>
> - Secrets are not encrypted they only encoded.
> - dont push secrets to code repos
> - secrets are not encrypted in `etcd
>   - enable the encryption at REST
> - Anyone able to create pods/deploys in the same namespace can access the secrets
>   - configure less privileged access to secrets - `RBAC`
> - Consider third party secrets providers
> - AWS provider
> - AZURE provider
> - GCP provider
> - valut provider

# Multicontainer pods

- The containers in a Pod are automatically co-located and co-scheduled on the same physical or virtual machine in the cluster.
- The containers can share resources and dependencies, communicate with one another, and coordinate when and how they are terminated.

## Multi Container Pods design and Pattern

1. SIDECAR
2. ADAPTER
3. AMBASSADOR

![alt text](image-7.png)

---

#### SideCar Container

#### ADAPTER Container

#### AMBASSADOR Container

## Pods in a Kubernetes cluster are used in two main ways

- Pods that run a single container.
  - The "one-container-per-Pod" model is the most common Kubernetes use case; in this case, you can think of a Pod as a wrapper around a single container; Kubernetes manages Pods rather than managing the containers directly.
- Pods that run multiple containers that need to work together.
  - A Pod can encapsulate an application composed of multiple co-located containers that are tightly coupled and need to share resources. These co-located containers form a single cohesive unit of service—for example, one container serving data stored in a shared volume to the public, while a separate **sidecar** container refreshes or updates those files. The Pod wraps these containers, storage resources, and an ephemeral network identity together as a single unit.

### [Init Containers](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/)

- Init containers can contain **utilities or setup scripts** not present in an app image.
- You can specify init containers in the Pod specification alongside the containers array (which describes app containers).
- containers that run to completion during Pod initialization.
- init containers, which are run before the app containers are started.
- If a Pod's init container `fails`, the `kubelet` repeatedly restarts that init container until it succeeds.
- However, if the Pod has a `restartPolicy` of `Never`, and an init container fails during startup of that Pod, Kubernetes treats the overall Pod as failed.
- But at times you may want to run a process that runs to completion in a container.
  
**For example** a process that pulls a code or binary from a repository that will be used by the main web application. That is a task that will be run only  one time when the pod is first created. Or a process that waits  for an external service or database to be up before the actual application starts. That's where initContainers comes in.

Example 1:

```
    apiVersion: v1
    kind: Pod
    metadata:
      name: myapp-pod
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp-container
        image: busybox:1.28
        command: ['sh', '-c', 'echo The app is running! && sleep 3600']
      initContainers:
      - name: init-myservice
        image: busybox
        command: ['sh', '-c', 'git clone <some-repository-that-will-be-used-by-application> ; done;']
```

**Init containers are exactly like regular containers, except:**

- Init containers always run to completion.
- Each init container must `complete successfully` before the next one starts.

### [Differences from regular containers](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/#differences-from-regular-containers)

- Init containers support all the fields and features of app containers, including resource limits, volumes, and security settings
- the resource requests and limits for an init container are handled differently
- Regular init containers (in other words: excluding sidecar containers) do not support the lifecycle, livenessProbe, readinessProbe, or startupProbe fields.

### Differences in `init` and `Sidecar` cinatiner

`**Init container**`

- They run to completion before any application containers start within a pod.
- Their primary role is to set up the correct environment for the app to run.
- This may involve tasks like database migrations, configuration file creation, or permission setting.
- Once an init container has successfully completed its task, the app containers in the pod can start.
- init containers do not support lifecycle, livenessProbe, readinessProbe, or startupProbe
- Init containers share the same resources (CPU, memory, network) with the main application containers

`**Sidecar**`

- They run alongside the main application container, providing additional capabilities like logging, monitoring, or data synchronization.
- Unlike init containers, sidecar containers remain running for the life of the pod.
- They are used to enhance or extend the functionality of the primary app container without altering the primary application code.
- For example, a logging sidecar might collect logs from the main container and forward them to a central log store.
- whereas sidecar containers support all these probes to control their lifecycle.

### Mutiple `Init` conatainers

- If you specify multiple init containers for a Pod, kubelet runs each init container sequentially
- Each init container must succeed before the next can run.
- When all of the init containers have run to completion, `kubelet` initializes the application containers for the Pod and runs them as usual.
- If any of the initContainers fail to complete, Kubernetes restarts the Pod repeatedly until the Init Container succeeds.
- if the pod fails to restart after sevral attemps then K8s gives up and pod goes into `crashloopbackoff` error.

Example 2:

```
    apiVersion: v1
    kind: Pod
    metadata:
      name: myapp-pod
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp-container
        image: busybox:1.28
        command: ['sh', '-c', 'echo The app is running! && sleep 3600']
      initContainers:
      - name: init-myservice
        image: busybox:1.28
        command: ['sh', '-c', 'until nslookup myservice; do echo waiting for myservice; sleep 2; done;']
      - name: init-mydb
        image: busybox:1.28
        command: ['sh', '-c', 'until nslookup mydb; do echo waiting for mydb; sleep 2; done;']
```

```
NAME        READY     STATUS     RESTARTS   AGE
myapp-pod   0/1       Init:0/2   0          6m
```

# Cluster Maintenance

## OS upgrades and Patching of cluster

- We can drain the node on K8s cluster for which we want to patch
- this will make the node unschedulable and whatever pods present on the node will be terminated and created on some other node which part of the replica set.
- if the pods on the node which is supposed tobe drain is not a part of replicaset of replication controller or deamonSet then that node will note be able to drain because that pod will not created on other node. So for this we need to use the `--force` option.

**Command**

```
kubectl drain `node_name`
kubectl drain controlplane --ignore-daemonsets
kubectl drain controlplane --force --ignore-daemonsets # when the pod is present and not a part of replicaSet
```

- with drain you can **empty** node and make the node **unschedulable**
- after the pathing is done you can make the node normal by making it uncorden.

**Command**

```
kubectl uncordon `node_name`
```

- this will make the node schedulable. and the new pods will be spawned when the pods on the other node will be terminated or crashed.

**Command**

```
kubectl cordon 'node_name'
```

- This will cordon the node means this make the node unshedulable, But the pods which are already presnet on the node will not be evicted.
- **Taints** offer more granular control compared to cordoning

## Cluster Upgrade Process

- always the `kube-apiserver` will be one or two versions higher than other cluster components
- but the `kubectl` can be higher than `kube-apiserver`
- ![alt text](image-8.png)
- whenever the new version of k8s is releases then the `X-3` version will be unsupported.
- recommended way of upgrading is upgrading minor versios
  - ex - `1.10 --> 1.11 --> 1.12 --> 1.13`
- you can easily upgrade the k8s version on managed k8s on cloud provider with just few clicks
- for `kubeadm` you can use the
  - `kubeadm upgrade plan`
  - `kubeadm upgrade apply`
    - in kubeadm upgrade the kublets are not upgraded you have to manually upgrade the kubelets
    - when you do `kubectl get node` it will only shows the versions of the kubelet on the master and worker nodes
- if you have deployed the k8s cluster from scratch then you need to upgrade each compomnents manually.

### Cluster upgrade strategy

- **Strategy-1**
  - upgrade the mastet node first this will not give downtime as the pods in the nodes are still serving the traffic
    - as the master is still upgrading the new pods on the Worker nodes will not be created as a part of `replicaSet`
  - upgrading all node at once this will have a downtime
- **Strategy-2**
  - Upgarding the master node fist this will not give downtime as the pods in the nodes are still serving the traffi
    - as the master is still upgrading the new pods on the Worker nodes will not be created as a part of `replicaSet`
  - Upgarding the node one by one, when a node is been taken down for upgrade the pods on the node will be creted on the different node which is running.

---

# Backup and Restore Methods

- **etcd** is the key-value store for kubernetes cluster
- **There are two way you can backup the cluster configurations**
  - querying the `kube-apiserver`
  - taking a snapshot.db of `etcd`
- You cannot backup the etcd in **managed** K8s cluster

## Backup `ETCD`

- `etcd` stores the state of the cluster
- rather than taking a backup of each individual resource, best way to take the backup of `ETCD database`
- for etcd the data is stored in the `data-dir=/var/lib/etcd`
- etcd comes with a built in snapshot solution
- to take the `etcd` backup first stop the kube-apiserver using `Service kube-apiserver stopped`

```
ETCDCTL_API=3 etcdctl \
    snapshot save snapshot.db
```

- to check the snapshot status

```
ETCDCTL_API=3 etcdctl \
    snapshot status snapshot.db
```

```
ETCDCTL_API=3 etcdctl \
    snapshot restore snapshot.db \
        --data-dir /var/lib/etcd-from-backup
```

- `systemctl daemon-reload`
- `service etcd restart`
- for authentication remember to provide the certs for `etcd`

```
ETCDCTL_API=3 etcdctl --endpoints=https://[127.0.0.1]:2379 \
--cacert=/etc/kubernetes/pki/etcd/ca.crt \
--cert=/etc/kubernetes/pki/etcd/server.crt \
--key=/etc/kubernetes/pki/etcd/server.key \
snapshot save /opt/snapshot-pre-boot.db
```

- if are using a managed k8s you wont be able to access or backup the `etcd`
- best way to query the `api-server`
- to query the `api-server` use the below command

```
kubectl get all --all-namespaces -o yaml > all-deploy-services.yml
```

### what is a `etcdctl`?

- `etcdctl` is a command-line tool for interacting with etcd, a distributed key-value store.
- `etcdctl` is used to:
  - Put, get, and delete keys from etcd
  - Watch for changes to keys
  - Manage etcd cluster membership
  - Perform maintenance tasks, such as defragmentation and compaction

```
etcdctl put <key> <value>
etcdctl get <key>
etcdctl del <key>
# Watches for changes to a key in etcd and prints the new value when it changes.
etcdctl watch <key>
# Manages etcd cluster membership
etcdctl member <subcommand>
```

# [Disaster Recovery for K8s](https://www.youtube.com/watch?v=qRPNuT080Hk)

# Security

- controlling the access to `kube-apiserver` you can control the authentication
  - **who can acess** and **what can they do**

**Who can access?**
`Authentication mechanism`

- Username and passwords in a file
- Username and Token in a file
- Certificates
- LDAP
- Service Accounts

**Authorization in K8s**

1. RBAC Authorization (Role based)
2. ABAC Authorization (Attibute based)
3. Node Authorization
4. Webhook Mode

All communication between varius master components are secure using the `TLS certificate`
![alt text](image-10.png)

you can restricthe the communication between pods with `network policies`

## User Authentication in k8s cluster

- User access the k8s cluster:
  - Admins
  - Developers
  - End users
  - bots

- K8s connot manage the Users natively
  - hence you cannot create or list user in k8s
- k8s relies on external user managment service like `LDAP`
- you can only manage the k8s service account
- all user access is managed by the `kube-apiserver`
- ![alt text](image-11.png)
- Auth Mechanisam for `kube-apiserver`
  - Username and passwords in a csv file

```
password123,username1,user_id1
password456,username2,user_id2
password789,username3,user_id3
```

    - `--basic-auth-file=user-details.csv`
![alt text](image-12.png)

- Username and Token in a csv file
- Certificates
- LDAP
- Service Accounts

**Communication between Pods:**

- `Pods` are isolated from each other
- by default pod on any node can talk with any pod within a cluster.
- you can restrict this by setting up a network policy.

## TLS Certificate Authentication: (https://)

- **TLS Certification**is a cryptographic protocol designed to provide secure communication over a computer network. It ensures data privacy, integrity, and authentication between clients and servers.
- with the help of `https` there is no intervention between sender and recevier.
- `https` is an extension of `http`
- `RSA` is no longer support as a method for key exchange, it is replaced by [Diffie-Hellman algorithm](https://www.techtarget.com/searchsecurity/definition/Diffie-Hellman-key-exchange) for more secure key exchange method.

### Process

1. **Handshake Process:** `TLS Handshake`
   - **Client Hello:** The client sends a "Client Hello" message to the server, which includes the TLS version, cipher suites, and a randomly generated number.
   - **Server Hello:** The server responds with a "Server Hello" message, selecting the TLS version and cipher suite from the client's list, and provides a randomly generated number.
2. **Server Authentication and Pre-Master Secret:**
     - **Server Certificate:** The server sends its digital certificate to the client, which includes the server’s public key and is issued by a trusted `Certificate Authority (CA).`
     - **Pre-Master Secret:** The client generates a pre-master secret *encryption session key*, *encrypts it with the server’s public key*, and sends it to the server.
3. **Session Keys Creation:**
   - Both the client and the server use the`pre-master secret` along with the previously exchanged random numbers to generate the same session keys, which are symmetric keys used for the session’s encryption and decryption.
4. **Secure Communication:**
   - **Change Cipher Spec:** Both client and server send a message to indicate that future messages will be encrypted using the session keys.
   - **Encrypted Messages:** The client and server communicate securely using symmetric encryption.
![alt text](image-13.png)

> The combination of TLS and Asystemtec provides a `robust` security framework, where TLS secures the communication channel and Asystemtec verifies user identities and manages sessions.

- **Symmentric Encryption**
   1. it is a secure way of encryption
   2. In symmetric key encryption, the same key is used for both encryption and decryption of data.
   3. encrypted data and key travells through same network
   4. hacker can intersept that key while sending and decrypt it.
   5. **faster** way of authentication

- **Asymmtric Encryption**
   1. Asymmetric encryption uses two mathematically connected keys: a **public** key and a **private** key.
      1. `ssh-keygen` to generate the private and public keys
   2. The **public** key is used for encryption of user data, while the **private** key is used for decryption of user data of the same server itself.
   3. Server send the public key to client
   4. client uses the public key to encrypt the private key of client
   5. client sends the encrypted private key to server
   6. server decrypts the encrypted key with its private key
   7. client encrypts the data with the clients public key
   8. client sends the encrypted data to server
   9. server decrypts the data with the clients private key which server recived earlier.
   10. Reads the data.
   11. Hacker can place its own proxy server in place of actual server
   12. then client communicates with that hacker server and hacks the data.s
  
- **TLS Handshake**
   1. `SSL/TLS` certs are created by CA authorities.
   2. Application server sends its public key to CA authority.
   3. CA authority issues a TLS certificate
   4. CA authority enclose the application server key init and give **signature** with CA authority server public key.
      1. `SIGNATURE = Application server public key + CA authority server public key`
      2. signature is then added in the certificate.
   5. Now that certificate from CA will be transfered to the application server with application server public key init.
   6. now the application server sends that signed certificate to the client server
   7. client server then recives the certificate from application server with application server public key init
   8. client server then goes to the CA authority of the application server and brings its public key.
   9. client then verify the SIGNATURE with public key if application server and Public key of CA authority.
   10. if the SIGNATURE matches then Client establish the connection with that application server, if not then connection is not been established.
   11. client uses that public key to encrypt the private key of client
   12. client send that encrypted private key of the client server to application server which was been encrypted by the public key of the application server.
   13. application server then drypt that client private key with application server private key, and stores that Client private key for that session.
   14. client encrypts the data with its public key and send to the application server
   15. application server recives that ecrypted data and drypt it with clients private key and reads the data.

## TLS certificate in Kubernetes Cluster

### Certificate and Key extensions

![alt text](image-14.png)

- TLS certificates are used to establish secure connection between the Master Node and Workers nodes or a System Admins is accessing the `kube-apiserver` with the help of `Kubectl`.
- kube-apiserver is itself act as server so need to be secure with `https` connection by the help of `TLS certificate` so it has its own `apiserver.crt` and `apiserver.key` .
- similarly `etcd-server` also has `etcdserver.crt` and `etcdserver.key`.
- Admin also requires the `admin.crt` and `admin.key` to access the `kube-apiserver`.
- similarly the `kube-scheduler` also talks with the `kube-apiserver` to schedule the pods on the worker nodes. So it also requires the TLS certificate and the key `scheduler.crt` and `scheduler.key`
- Even the Kube-apiserver requires the `apiserver.crt` and `apiserver.key` to talk to `etcd-server`
![alt text](image-15.png)

### How to generate the TLS certificate for k8s cluster

1. **Cluster Certificates**
   - **Generate the Private Key**
     - command = `openssl genrsa -out ca.key 2048`
   - **Generate the Certificate Signing Request (CSR)**
     - command = `openssl req -new -key ca.key -subj "/CN=KUBERNETES-CA" -out ca.csr`
   - **Sign Certificates**
     - command = `openssl x509 -req -in ca.csr -signkey ca.key -out ca.crt`
     - `ca.csr` is the cert with no sign
     - `ca.key` is used to sign the cert
![Cerificate](image-16.png)
2. **Client Certificates**
   - **Generate the Private Key**
     - command = `openssl genrsa -out admin.key 2048`
   - **Generate the Certificate Signing Request (CSR)**
     - command = `openssl req -new -key admin.key -subj "/CN=KUBE-admin" -out admin.csr`
   - **Sign Certificates**
     - command = `openssl x509 -req -in admin.csr -signkey admin.key -out admin.crt`
     - `admin.csr` is the cert with no sign
     - `admin.key` is used to sign the cert
     - `admin.crt` is the final signed certificate

**To view all the certificates**

```
cat /etc/kubernetes/manifests/apiserver.yaml
```
