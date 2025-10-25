### References:
- [CloudWithVarJosh](https://www.youtube.com/watch?v=vaW2pwSXdb4&ab_channel=CloudWithVarJosh)
- [Kubernetes Official Documentation: Assigning Pods to Nodes](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/)  
- [Understanding Node Affinity in Kubernetes](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#node-affinity)  
- [Kubernetes Scheduling Preferences](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#types-of-node-affinity)  
- [Taints and Tolerations in Kubernetes](https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/)
- [Day15/40 - Kubernetes node affinity explained](https://youtu.be/5vimzBRnoDk)
- [Day 18: Taints & Tolerations vs. Node Affinity | MASTER Pod Scheduling Control](https://www.youtube.com/watch?v=itEINIqjNfE&ab_channel=CloudWithVarJosh)
- [Day 17: Mastering Node Selector & Node Affinity Rules in Kubernetes](https://www.youtube.com/watch?v=vaW2pwSXdb4&ab_channel=CloudWithVarJosh)
- [Kubernetes Documentation: Taints & Tolerations](https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/)
- [Kubernetes Documentation: Node Affinity](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/)
- [Day 16: Mastering Kubernetes Taints & Tolerations | Essential Scheduling Control](https://www.youtube.com/watch?v=G_Ro0urceF0&ab_channel=CloudWithVarJosh)  
- [Understanding Taints and Tolerations in Kubernetes](https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/)
- [Day14/40 - Taints and Tolerations in Kubernetes](https://youtu.be/nwoS2tK2s6Q)
- https://www.tutorialspoint.com/kubernetes/kubernetes_managing_taints_and_tolerations.htm
- https://medium.com/@muppedaanvesh/kubernetes-taints-tolerations-b0e0ed076cad
- https://medium.com/@prateek.malhotra004/demystifying-taint-and-toleration-in-kubernetes-controlling-the-pod-placement-with-precision-d4549c411c67
- https://overcast.blog/mastering-kubernetes-taints-and-tolerations-08756d5faf55
- https://www.densify.com/kubernetes-autoscaling/kubernetes-taints/
- https://www.geeksforgeeks.org/cloud-computing/kubernetes-taint-and-toleration/
- https://overcast.blog/mastering-kubernetes-taints-and-tolerations-08756d5faf55
- [Best Practices taint-tolerations](https://overcast.blog/mastering-kubernetes-taints-and-tolerations-08756d5faf55)

---
```bash
controlplane:~$ kubectl describe pod nginx | grep -i Node: #to get node of the pod
Node:             node01/172.30.2.2

kubectl describe pod <pod-name>
kubectl taint node <node_name> key=value:effect
kubectl taint node <node_name> --help/-h
kubectl describe node <node-name> | grep -i Taints
kubectl get node <node-name> -o jsonpath='{.spec.taints}'
kubectl get nodes -o custom-columns=NAME:.metadata.name,TAINTS:.spec.taints #Get All Nodes with Their Taints
```
---

* In Kubernetes, scheduling decisions determine where a pod runs within the cluster. While Kubernetes has a built-in **scheduler** that automatically assigns pods to nodes but in production every node is different some has GPUs some have SSDs some have High CPUs and these node can run different kind of workloads, administrators often need to influence **scheduling decisions** to ensure specific workloads run on designated nodes.
* Scheduling can be influence by: 
    * `Request and Limit`
    * `NodeSelector`(basic)
    * `Affinity`(advance)
    * `Taint and Toleration`

* What we can achive using scheduling
    - Workloads are placed on **nodes with required hardware** (e.g., GPUs, SSDs).  
    - Applications are **isolated by environment** (e.g., prod, staging, production).  
    - Compliance with **resource constraints** (e.g., memory, CPU). 

---
---

## Labels
* Labels group nodes based on size, type,env, etc. Unlike taints, labels don't directly affect scheduling but are useful for organizing resources.

```bash
kubectl label nodes <node-name> disktype=ssd #add label

kubectl label nodes <node-name> zone=us-east-1a

kubectl label nodes <node-name> disktype- #remove label

kubectl get nodes --show-labels #get label of all nodes

kubectl get pods --show-labels

kubectl get nodes <node_names...> --show-labels #get label of specified nodes

kubectl get pods -l/--selector key=value

kubectl label pod <pod> --overwrite key=value
```
#### Types of labels
* **User-Defined Labels(custom)**:
  * Created manually by users or automation tools.
  * Help identify or organize resources for management, deployment, or selection.
  * Commonly used for:
    * Environment (env=dev, env=prod)
    * Application (app=nginx, app=frontend)
    * Component (tier=backend, tier=database)
    * Team or owner (team=devops, owner=mahin)
  
* **System (or Built-in) Labels**
  * Automatically added by Kubernetes to identify internal system information.
  * You should not modify these — they’re used by the scheduler and controllers.

    | Label                              | Description                                              |
    | ---------------------------------- | -------------------------------------------------------- |
    | `kubernetes.io/hostname`           | Node’s name (unique per node).                           |
    | `kubernetes.io/os`                 | Operating system of the node (e.g., `linux`, `windows`). |
    | `kubernetes.io/arch`               | CPU architecture (`amd64`, `arm64`, etc.).               |
    | `topology.kubernetes.io/region`    | Region where the node is located.                        |
    | `topology.kubernetes.io/zone`      | Zone (e.g., availability zone in cloud).                 |
    | `node.kubernetes.io/instance-type` | Node’s instance type (like EC2 size).                    |
    | `kubernetes.io/metadata.name`      | Automatically set in namespaces to reflect their name.   |

* **Topology Labels**
  * Used for zone/region-aware scheduling and fault-tolerant deployments.
  * Commonly applied to nodes.
    ```bash
    topology.kubernetes.io/region=us-east1
    topology.kubernetes.io/zone=us-east1-b
    ```
  * Helps distribute Pods across multiple zones for high availability.
  * Used with:
    * Pod topology spread constraints
    * Affinity / anti-affinity rules

* **Controller / Operator Labels**
  * Added automatically by controllers (like Deployments, ReplicaSets, DaemonSets).
  * Used to track resource relationships and ownership.
    ```bash
    app.kubernetes.io/name: nginx
    app.kubernetes.io/instance: my-nginx
    app.kubernetes.io/component: web
    app.kubernetes.io/part-of: myapp
    app.kubernetes.io/managed-by: Helm
    ```
  * These labels are part of the Kubernetes recommended labeling convention (defined in official docs).

* **Cloud Provider / Platform Labels**
  * Automatically added by cloud environments or managed clusters (EKS, GKE, AKS, etc.)
  * Describe node hardware, location, or provider metadata.
    ```bash
    failure-domain.beta.kubernetes.io/zone=us-west-2a
    eks.amazonaws.com/nodegroup=my-node-group
    cloud.google.com/gke-nodepool=default-pool
    ```

* **Custom Automation Labels**
  * Added by CI/CD tools, GitOps pipelines, or configuration management systems.
  * Useful for identifying deployment source, version, or rollout stage.
    ```bash
    pipeline=jenkins
    build=45
    version=1.2.3
    deployed-by=argo
    ```

---
--- 

## Node Selector  

* `nodeSelector` is the **simplest way to schedule Pods onto specific Nodes** in a Kubernetes cluster.
* It works by matching **labels on Nodes** with the values specified in the Pod definition. 
* Useful when you have specialized workloads, e.g., data processing pods needing nodes with SSDs.
* **If no node matches the label**, the pod remains in a **Pending** state.  
* **Only supports exact key-value matching** (no advanced logic).  


#### How It Works  

1. **Assign labels to nodes**  
   ```sh
   kubectl label nodes <node-name> <label-key>=<label-value>
   ```
2. **Define a `nodeSelector` in the pod spec**  
   ```yaml
   nodeSelector:
     key: value
   ```
3. **Pods are scheduled only on nodes matching the label**  

---

#### Limitations of Node Selector  

| Limitation | Explanation |
|------------|------------|
| **Strict Placement** | If no node matches the label, the pod remains in the **Pending** state. |
| **No Preferences** | It does not allow "soft" preferences—either a node matches or it does not. |
| **No OR Condition** | You cannot specify "schedule on nodes with `storage=ssd` OR `storage=hdd`". |

For more **advanced** scheduling needs, **Node Affinity** provides a more flexible alternative.

---

#### Exercise

* Two nodes
* Labels
  ```bash
  kubectl label nodes node01 storage=ssd
  kubectl label nodes node02 storage=hdd
  kubectl label nodes node01 env=prod
  kubectl label nodes node01 memory=high
  ```
* Check labels
  ```sh
  kubectl get nodes --show-labels
  kubectl describe node my-second-cluster-worker
  kubectl describe node controlplane | grep -i -A7 labels
  ```
  This will display all the labels assigned to the nodes.

* Deploy a Pod with `nodeSelector` = `storage=ssd`  
  ```yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: example1 #name in small case
  spec:
    replicas: 1
    selector:
      matchLabels:
        app: app1
    template:
      metadata:
        labels:
          app: app1
      spec:
        nodeSelector:
          storage: ssd
        containers:
          - name: nginx
            image: nginx
  ```
  Pods will only be scheduled on `node1` because it has the label `storage=ssd`.  
  If no node has `storage=ssd`, the pods remain in the `Pending` state.  
<br>

* Deploy a Pod with `nodeSelector` = `storage=hdd` 
  ```yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: example2
  spec:
    replicas: 3
    selector:
      matchLabels:
        app: app1
    template:
      metadata:
        labels:
          app: app1
      spec:
        nodeSelector:
          storage: hdd
        containers:
          - name: nginx
            image: nginx
  ``` 
  Pods will only be scheduled on `node2` because it has the label `storage=hdd`.

* Deploy a Pod with `nodeSelector` = `storage=hdd` and 'env=prod` 
  ```yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: example3
  spec:
    replicas: 1
    selector:
      matchLabels:
        app: app1
    template:
      metadata:
        labels:
          app: app1
      spec:
        nodeSelector:
          storage: hdd
          env: prod
        containers:
          - name: nginx
            image: nginx
  ``` 
  Pods will not scheduled on any node because there is no node with both `storage=hdd` and `env=prod` labels.
  ```bash
  kubectl describe pod example3-67ddf85b54-5qxm7

  Events:
  Type     Reason            Age   From               Message
  ----     ------            ----  ----               -------
  Warning  FailedScheduling  44s   default-scheduler  0/2 nodes are available: 2 node(s) didn't match Pod's node affinity/selector. no new claims to deallocate, preemption: 0/2 nodes are available: 2 Preemption is not helpful for scheduling.
  ```

* Deploy a Pod with `nodeSelector` = `storage=hdd` and `env=prod` 
  ```yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: example4
  spec:
    replicas: 1
    selector:
      matchLabels:
        app: app1
    template:
      metadata:
        labels:
          app: app1
      spec:
        nodeSelector:
          storage: premium-ssd
        containers:
          - name: nginx
            image: nginx
  ``` 
  Pods will not scheduled on any node because there is no node with both `storage=premium-ssd` label.
  ```bash
  kubectl describe pod example4-6474c5664d-85w8x

  Events:
  Type     Reason            Age   From               Message
  ----     ------            ----  ----               -------
  Warning  FailedScheduling  13s   default-scheduler  0/2 nodes are available: 2 node(s) didn't match Pod's node affinity/selector. no new claims to deallocate, preemption: 0/2 nodes are available: 2 Preemption is not helpful for scheduling.
  ```

<br>

* Difference `nodeSelectors`
  Run **database Pods** on nodes with SSDs:
  ```yaml
  nodeSelector:
    disktype: ssd
  ```
  Run **GPU workloads** only on GPU nodes:
  ```yaml
  nodeSelector:
    accelerator: nvidia
  ```
  Separate workloads by environment:
  ```yaml
  nodeSelector:
    env: production
  ```
  built-in labels
  ```bash
  nodeSelector:
  kubernetes.io/os: linux
  ```


* `nodeSelector` applies only during Pod scheduling — that is, when the Pod is first placed on a node.
* Once a Pod is scheduled:
  * If you change or delete the label on the node afterward, the Pod will not be evicted or rescheduled.
  * Kubernetes does not automatically re-evaluate nodeSelector conditions for already running Pods.
  * Once pod deleted it will not recreated on that node if node not have that label.

  ```bash
  controlplane:~$ kubectl get pods -owide              
  NAME                        READY   STATUS    RESTARTS   AGE   IP            NODE     NOMINATED NODE   READINESS GATES
  example1-664f5db69b-bdbv4   1/1     Running   0          51s   192.168.1.4   node01   <none>           <none>

  controlplane:~$ kubectl label nodes node01 storage-   
  node/node01 unlabeled 

  controlplane:~$ kubectl get pods -owide 
  NAME                        READY   STATUS    RESTARTS   AGE    IP            NODE     NOMINATED NODE   READINESS GATES
  example1-664f5db69b-bdbv4   1/1     Running   0          101s   192.168.1.4   node01   <none>           <none>

  controlplane:~$ kubectl delete pod example1-664f5db69b-bdbv4

  controlplane:~$ kubectl get pods -owide 
  NAME                        READY   STATUS    RESTARTS   AGE   IP       NODE     NOMINATED NODE   READINESS GATES
  example1-664f5db69b-vxl64   0/1     Pending   0          30s    <none>   <none>   <none>           <none>
  ```

##### Key Takeaways for `nodeSelector` 
* **Node Selector** is a simple, strict way to assign pods to nodes.  
* **Pods are only scheduled if a node has all required labels**.  
* **Lack of flexibility** makes it unsuitable for complex scheduling needs.  
* **Node Affinity** is the advanced alternative with more features.


---
---
## Affinity
* control where pods are scheduled.
  * Ensure **related pods run on the same node** (for caching, data locality).
  * Ensure **critical pods are spread across nodes** for high availability.
  * Example: Place a frontend pod and its backend pod on the same node, or ensure two replicas of a database never run on the same node.
  * Decision take by set of rules
  * It’s part of the **PodSpec → affinity** field.
  * Similar to `nodeSelector`, but **more expressive and flexible**.
* complex rules
* give good control over scheduling
* It allows **hard constraints** (must match) and **soft preferences** (prefer but not mandatory).  
* It enables complex **AND/OR conditions** for node selection.  

### 2. Types of Affinity

* **Node Affinity** → Schedule pods based on **node labels**.
* **Pod Affinity** → Schedule pods close to **other pods**.
* **Pod Anti-Affinity** → Prevent pods from being scheduled on the **same node** as others.

### Node Affinity in Kubernetes  
- `nodeAffinity` is an advanced version of `nodeSelector` that provides more flexibility in **scheduling** pods onto specific nodes.  
- It supports **set-based expressions** (`In`, `NotIn`, `Exists`, etc.), unlike `nodeSelector`.  
- Uses **nodeSelectorTerms** with matchExpressions.


* Example:- Targeting SSD NodesPod runs only on nodes with label `disktype=ssd`.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: node-affinity-pod
spec:
  containers:
  - name: nginx
    image: nginx
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: disktype
            operator: In
            values:
            - ssd
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: redis
  name: redis-3
spec:
  containers:
  - image: redis
    name: redis
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution: #OR condition
          nodeSelectorTerms:
          - matchExpressions:
              - key: disktype
                operator: In
                values:
                - ssd
                - hdd 
```

#### Types of Node Affinity(condition)

| **Type** | **Behavior** |
|----------|-------------|
| `requiredDuringSchedulingIgnoredDuringExecution` | **Hard rule** – The pod will only be scheduled on matching nodes. If no matching node exists, the pod stays in **Pending state**. |
| `preferredDuringSchedulingIgnoredDuringExecution` | **Soft rule** – The pod will **prefer** matching nodes but can be scheduled elsewhere if needed. |

**IgnoredDuringExecution** means that **if the node’s labels change after scheduling, the pod will continue running** on that node **does not force eviction**(There is no `RequiredDuringExecution` today.)

#### Demo: Node Affinity  

##### Step 1: Label a Node  

```sh
kubectl label nodes my-second-cluster-worker storage=ssd
```

This assigns the label **storage=ssd** to `my-second-cluster-worker`.


##### Step 2: Deploy a Pod with Required Node Affinity  

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: na-deploy
spec:
  replicas: 3
  selector:
    matchLabels:
      app: app1
  template:
    metadata:
      labels:
        app: app1
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: storage
                operator: In
                values:
                  - ssd
      containers:
        - name: nginx
          image: nginx
```

##### Expected Behavior  

- Pods will **only be scheduled on nodes with `storage=ssd`**.  
- If no node has `storage=ssd`, the pod **remains in Pending state**.  

This behaves **similarly to `nodeSelector`**, but **nodeAffinity supports complex conditions**.

📌 **More on Set-Based Selectors:** [Day 10 - ReplicaSet](https://github.com/CloudWithVarJosh/CKA-Certification-Course-2025/tree/main/Day%2010#3-replicaset-rs)


#### OR Condition (Multiple Label Matches)  

If we want to **allow scheduling on nodes with either `storage=ssd` or `storage=hdd`**, we use **OR logic** with `In`.  

##### Step 1: Label a Second Node  

```sh
kubectl label nodes my-second-cluster-worker2 storage=hdd
```

##### Step 2: Update Node Affinity to Allow Either SSD or HDD  

```yaml
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
      - matchExpressions:
        - key: storage
          operator: In
          values:
            - ssd
            - hdd
```

##### Expected Behavior  

- Pods **can be scheduled on**:  
  ✅ `my-second-cluster-worker` (**storage=ssd**)  
  ✅ `my-second-cluster-worker2` (**storage=hdd**)  


#### AND Condition (Multiple Label Requirements)  

To schedule **only on nodes that have BOTH `storage=ssd` AND `env=prod`**, we use multiple `matchExpressions`.  

##### Step 1: Label a Node  

```sh
kubectl label nodes my-second-cluster-worker env=prod storage=ssd
```

##### Step 2: Apply AND Condition  

```yaml
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
      - matchExpressions:
        - key: storage
          operator: In
          values:
            - ssd
        - key: env
          operator: In
          values:
            - prod
```

##### Expected Behavior  

- Pods will **only be scheduled on nodes that have both `storage=ssd` and `env=prod`**.  
- If no node has both labels, **the pod stays in Pending state**.  

#### Node Affinity Operators  

| **Operator** | **Behavior** |
|-------------|-------------|
| `In` | Matches if the node’s label **exists in the provided values**. |
| `NotIn` | Matches if the node’s label **does not exist in the provided values**. |
| `Exists` | Matches if the node **has the specified key**, regardless of its value. |
| `DoesNotExist` | Matches if the node **does not have the specified key**. |
| `Gt` | Matches if the label’s **value is greater than the specified number**. |
| `Lt` | Matches if the label’s **value is less than the specified number**. |

##### Example: Using All Operators  

```yaml
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
      - matchExpressions:
        - key: storage
          operator: In
          values:
            - ssd
            - hdd
        - key: env
          operator: NotIn
          values:
            - dev
        - key: high-memory
          operator: Exists
        - key: dedicated
          operator: DoesNotExist
        - key: cpu
          operator: Gt
          values:
            - "4"
        - key: disk
          operator: Lt
          values:
            - "500"
```
```sh
kubectl label nodes node01 storage=ssd
kubectl label nodes node01 env=prod
kubectl label nodes node01 high-memory=
kubectl label nodes node01 cpu=5
kubectl label nodes node01 disk=499
```

##### Explanation  

| **Condition** | **Effect** |
|--------------|-----------|
| `storage In (ssd, hdd)` | ✅ Pod can be scheduled if the node has **storage=ssd OR storage=hdd**. |
| `env NotIn (dev)` | ✅ Pod is **not scheduled on nodes labeled `env=dev`**. |
| `high-memory Exists` | ✅ Pod can be scheduled **only on nodes that have a `high-memory` label**. |
| `dedicated DoesNotExist` | ✅ Pod **avoids nodes with `dedicated` label**. |
| `cpu Gt 4` | ✅ Pod can be scheduled **only on nodes with CPU > 4**. |
| `disk Lt 500` | ✅ Pod can be scheduled **only on nodes with disk < 500**. |


##### Real Use Cases
* Run GPU workloads only on nodes with `accelerator=nvidia`
* Pin database pods to SSD-backed nodes (`disktype=ssd`)
* Keep dev workloads away from prod nodes (`env=dev`)
* Prefer cheap spot instances but fall back to on-demand

##### Best Practices
* Use **node affinity** instead of `nodeSelector` for flexibility.
* Combine with **podAffinity/podAntiAffinity** for topology-aware placement.
* Combine with **taints & tolerations** for strict isolation.
* Avoid over-restricting pods → may cause scheduling failures.
* Label nodes with **meaningful metadata** (`zone`, `instance-type`, `storage`, etc.).


### Node Anti-Affinity  
* **Node Anti-Affinity** lets you **prevent pods from being scheduled on certain nodes**, based on node **labels**.
* **Alternative:** Use **taints and tolerations** to **repel pods** from certain nodes.  
* Example:
  * "Do not schedule this pod on nodes labeled `env=dev`."
  * "Avoid GPU nodes for lightweight workloads."
* It uses the same **affinity syntax**, but with **negative operators** (`NotIn`, `DoesNotExist`).
    * **NotIn** → Label value must not be in the list
    * **DoesNotExist** → Label key must not exist

| Feature   | Node Affinity           | Node Anti-Affinity        |
| --------- | ----------------------- | ------------------------- |
| Purpose   | Attract pods to nodes   | Repel pods from nodes     |
| Example   | "Run only on SSD nodes" | "Do not run on HDD nodes" |
| Operators | In, Exists, etc.        | NotIn, DoesNotExist       |

* **requiredDuringSchedulingIgnoredDuringExecution** → Hard rule
  * Pod won’t be scheduled at all if the anti-affinity condition matches.
  
* **preferredDuringSchedulingIgnoredDuringExecution** → Soft rule
  * Scheduler avoids nodes if possible, but will place if no alternatives.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-node-anti-affinity
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: disktype
            operator: In
            values:
            - hdd
          - key: disktype
            operator: NotIn
            values:
            - ssd
  containers:
  - name: nginx
    image: nginx
```
🔹 This pod will **never** schedule on nodes labeled `disktype=ssd`.


```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-soft-node-anti-affinity
spec:
  affinity:
    nodeAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 100
        preference:
          matchExpressions:
          - key: env
            operator: NotIn
            values:
            - dev
  containers:
  - name: busybox
    image: busybox
    command: ["sleep", "3600"]
```

🔹 Pod will **prefer avoiding nodes labeled `env=dev`**, but still run there if no other option.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-no-gpu
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: gpu
            operator: DoesNotExist
  containers:
  - name: app
    image: myapp:1.0
```

🔹 Pod only schedules on nodes **without any `gpu` label**.

Label nodes:

```bash
kubectl label nodes node1 disktype=hdd
kubectl label nodes node2 env=dev
```

Check where pod scheduled:

```bash
kubectl get pod pod-node-anti-affinity -o wide
```

Debug pending pod:

```bash
kubectl describe pod pod-node-anti-affinity
```

#### Real Use Cases
* Keep lightweight apps away from expensive GPU nodes
* Ensure dev/test workloads don’t land on prod nodes
* Avoid running stateful apps on HDD-backed nodes
* Separate compliance workloads from general-purpose nodes

#### Best Practices
* Use **soft anti-affinity** where possible → prevents scheduling failures.
* Combine with **taints & tolerations** for stricter isolation.
* Use **meaningful node labels** (`gpu=true`, `env=prod`, `storage=hdd`).
* Test carefully → too many anti-affinity rules can make pods **unschedulable**.


#### Understanding `preferredDuringSchedulingIgnoredDuringExecution`  

`preferredDuringSchedulingIgnoredDuringExecution` allows you to **influence** pod placement without enforcing strict scheduling rules.  

- The **scheduler prefers nodes** that match the preference but **can place pods elsewhere if needed**.  
- Nodes are assigned **weights** that determine their preference level.  
- The **higher the weight, the stronger the preference** for scheduling on that node.  

### **How Node Affinity Weight Works**  

1. The **scheduler finds all nodes** that satisfy the `requiredDuringSchedulingIgnoredDuringExecution` rule.  
2. It then **evaluates each preferred rule** (`preferredDuringSchedulingIgnoredDuringExecution`) and **adds the node's weight to a sum**.  
3. The **node with the highest final score is preferred**, but **other scheduling factors still apply**.  

#### Demonstration: Using Preferred Node Affinity  

##### Step 1: Apply Labels to Nodes  

We need to ensure our worker nodes have labels for **storage type**.  

```sh
kubectl label nodes my-second-cluster-worker storage=ssd
kubectl label nodes my-second-cluster-worker2 storage=hdd
```

##### Step 2: Define Node Affinity with Preferences  

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: preferred-na-deploy
spec:
  replicas: 5
  selector:
    matchLabels:
      app: app1
  template:
    metadata:
      labels:
        app: app1
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: storage
                operator: In
                values:
                  - ssd
                  - hdd
          preferredDuringSchedulingIgnoredDuringExecution:
            - weight: 10
              preference:
                matchExpressions:
                  - key: storage
                    operator: In
                    values:
                      - ssd
            - weight: 5
              preference:
                matchExpressions:
                  - key: storage
                    operator: In
                    values:
                      - hdd
      containers:
        - name: nginx
          image: nginx
```

##### Explanation  

- The **pod is required** to run on nodes labeled **storage=ssd OR storage=hdd** (hard rule).  
- The **scheduler prefers `storage=ssd` nodes** (higher weight: `10`).  
- The **scheduler prefers `storage=hdd` nodes** slightly less (lower weight: `5`).  

##### Step 3: Deploy and Observe Pod Placement  

```sh
kubectl apply -f preferred-na-deploy.yaml
kubectl get pods -o wide
```

##### Expected Behavior  

- Pods **prefer `my-second-cluster-worker` (storage=ssd) over `my-second-cluster-worker2` (storage=hdd)**.  
- If there are **not enough resources on `my-second-cluster-worker`**, some pods may be placed on `my-second-cluster-worker2`.  
- Increasing **replica count** makes the preference **more noticeable**.  

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-preferred-affinity
spec:
  affinity:
    nodeAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 80
        preference:
          matchExpressions:
          - key: zone
            operator: In
            values:
            - us-east-1a
      - weight: 20
        preference:
          matchExpressions:
          - key: high-memory
            operator: Exists
  containers:
  - name: busybox
    image: busybox
    command: ["sleep", "3600"]
```



#### Key Considerations  

- **Weight only influences placement; it does not guarantee it.**  
- **Pods may still be placed on lower-weight nodes** if the preferred nodes **lack resources**.  
- Other scheduling factors **still apply**, such as:  
  - Available CPU & memory on each node.  
  - Other affinity/anti-affinity rules.  
  - Taints & tolerations.  

### Combined Example

You can **mix required + preferred** rules:

```yaml
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
      - matchExpressions:
        - key: env
          operator: In
          values:
          - prod
    preferredDuringSchedulingIgnoredDuringExecution:
    - weight: 50
      preference:
        matchExpressions:
        - key: instance-type
          operator: In
          values:
          - m5.large
```

* Use **`requiredDuringSchedulingIgnoredDuringExecution`** for strict placement rules, and **`preferredDuringSchedulingIgnoredDuringExecution`** for weighted preferences.
---
## Pod Affinity
* Places a pod **on the same node (or topology domain)** as another pod with specific labels.
* “Place this pod *close to* (on the same node, zone, rack, etc.) other pods that have specific labels.”
* It’s used to **co-locate pods** for performance, latency, or data-sharing reasons.

##### How Pod Affinity Works
* Unlike **Node Affinity** (which matches against **node labels**),
  **Pod Affinity** matches against **labels on other pods** and considers **topology domains**.

* Topology domains are defined by **node labels** like:
  * `kubernetes.io/hostname` (same node)
  * `topology.kubernetes.io/zone` (same zone)
  * `topology.kubernetes.io/region` (same region)
  * custom node labels

* Example: Run a **frontend pod** on the same node/zone as the **backend pod** to reduce network latency.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-affinity-pod
spec:
  containers:
  - name: nginx
    image: nginx
  affinity:
    podAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
          - key: app
            operator: In
            values:
            - backend
        topologyKey: "kubernetes.io/hostname"
```

✅ Here:
* **labelSelector** → Match pods by labels-->finds backend pods
* **topologyKey** → Defines the topology scope (node, zone, region, etc.)-->tells Kubernetes to match on node (`hostname`)

* **requiredDuringSchedulingIgnoredDuringExecution**
  * Hard rule: pod won’t schedule unless co-location requirement is met.
  
* **preferredDuringSchedulingIgnoredDuringExecution**
  * Soft rule: scheduler *tries* to co-locate pods, but will still schedule elsewhere if not possible.

* Hard Pod Affinity (same node as backend pods)
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: frontend
spec:
  affinity:
    podAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
          - key: app
            operator: In
            values:
            - backend
        topologyKey: kubernetes.io/hostname
  containers:
  - name: nginx
    image: nginx
```
🔹 Pod runs only on nodes that already have a pod with label `app=backend`.

* Soft Pod Affinity (prefer same zone as backend)
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: analytics
spec:
  affinity:
    podAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 100
        podAffinityTerm:
          labelSelector:
            matchExpressions:
            - key: app
              operator: In
              values:
              - backend
          topologyKey: topology.kubernetes.io/zone
  containers:
  - name: busybox
    image: busybox
    command: ["sleep", "3600"]
```
🔹 Pod will prefer to run in the same **zone** as `app=backend`, but not mandatory.

* Complex Pod Affinity (match multiple labels)
```yaml
affinity:
  podAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
    - labelSelector:
        matchExpressions:
        - key: app
          operator: In
          values: ["backend"]
        - key: tier
          operator: In
          values: ["db"]
      topologyKey: topology.kubernetes.io/region
```
🔹 Pod runs in the same **region** as pods labeled `app=backend, tier=db`.

#### Real Use Cases
* Place **frontend** close to **backend** for low latency
* Run **analytics workers** in same zone as **data nodes**
* Ensure **logging agents** run on the same node as the app pods
* Improve **cache hit rates** by co-locating with cache pods

#### Best Practices
* Use **soft (preferred) affinity** where possible to avoid scheduling failures.
* Use **topologyKey=zone** or **region** for HA (don’t force same node unless necessary).
* Combine with **Pod Anti-Affinity** to spread workloads while still co-locating groups.
* Be mindful: **too strict affinity rules = pods stuck in Pending**.
---
## Pod Anti-Affinity

* **Pod Anti-Affinity** ensures that pods **do not get scheduled on the same node (or zone, or region)** as other pods with specific labels.
* It’s useful for **spreading workloads** to improve availability and fault tolerance.
* Example: 
    * Spread multiple replicas of a pod across different nodes.
    * Don’t schedule two replicas of the same app on the same node → avoids single-node failure taking down the app.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-anti-affinity-pod
spec:
  containers:
  - name: nginx
    image: nginx
  affinity:
    podAntiAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
          - key: app
            operator: In
            values:
            - frontend
        topologyKey: "kubernetes.io/hostname"
```

✅ Here:

* Ensures that if one pod with label `app=frontend` is on a node, another `frontend` pod won’t be scheduled on the same node.

#### How It Works

* Matches **pod labels** (not node labels).
* Uses a **topologyKey** (scope of separation):
  * `kubernetes.io/hostname` → prevent scheduling on same node
  * `topology.kubernetes.io/zone` → prevent scheduling in same zone
  * `topology.kubernetes.io/region` → prevent scheduling in same region

* **requiredDuringSchedulingIgnoredDuringExecution** → Hard rule
  * If no valid node exists, pod stays Pending.
  
* **preferredDuringSchedulingIgnoredDuringExecution** → Soft rule
  * Scheduler avoids co-location if possible, but may still place pods together.

* Hard Pod Anti-Affinity (no two pods on same node)
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - web
            topologyKey: kubernetes.io/hostname
      containers:
      - name: nginx
        image: nginx
```

* Soft Pod Anti-Affinity (prefer spreading across zones)
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api
spec:
  replicas: 3
  selector:
    matchLabels:
      app: api
  template:
    metadata:
      labels:
        app: api
    spec:
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 100
            podAffinityTerm:
              labelSelector:
                matchExpressions:
                - key: app
                  operator: In
                  values:
                  - api
              topologyKey: topology.kubernetes.io/zone
      containers:
      - name: api-container
        image: myapi:1.0
```
🔹 Pods will **prefer spreading across different zones**, but still run in same zone if needed.

* Mixed: Hard + Soft Rules
```yaml
affinity:
  podAntiAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
    - labelSelector:
        matchLabels:
          app: db
      topologyKey: kubernetes.io/hostname
    preferredDuringSchedulingIgnoredDuringExecution:
    - weight: 50
      podAffinityTerm:
        labelSelector:
          matchLabels:
            app: db
        topologyKey: topology.kubernetes.io/zone
```
🔹 Pods **must not run on the same node**, and should also **spread across zones** if possible.



#### Real-World Use Cases

* High availability: Ensure replicas of an app don’t all run on the same node
* Fault tolerance: Spread across zones/regions to handle outages
* Stateful apps (DBs, brokers): Keep replicas apart to avoid data loss
* Large workloads: Prevent resource contention by separating pods

#### Best Practices
* For HA deployments, combine **pod anti-affinity** with **replicas > 1**.
* Use **soft rules** when clusters are small (strict rules can cause scheduling failures).
* Use **topology spread constraints** (newer alternative) for easier spreading across zones/nodes.
* Check pod scheduling with:
  ```bash
  kubectl describe pod <pod-name>
  ```

#### Pod Affinity vs Pod Anti-Affinity

| Feature  | Pod Affinity                              | Pod Anti-Affinity                  |
| -------- | ----------------------------------------- | ---------------------------------- |
| Purpose  | Co-locate pods                            | Spread pods apart                  |
| Use case | Frontend near backend                     | DB replicas on different nodes     |
| Risk     | Too much co-location → reduced resilience | Too much separation → Pending pods |
---
### Task details
- create a pod with nginx as the image and add the nodeffinity with property requiredDuringSchedulingIgnoredDuringExecution and condition disktype = ssd
- check the status of the pod and see why it is not scheduled
- add the label to your worker01 node as distype=ssd and then check the status of the pod
- It should be scheduled on worker node 1
- create a new pod with redis as the image and add the nodeaffinity with property requiredDuringSchedulingIgnoredDuringExecution and condition disktype without any value
- add the label to worker02 node with disktype and no value
- ensure that pod2 should be scheduled on worker02 node

---

### Real World Uses
* **Node Affinity**: Database pod runs only on high-performance SSD nodes.
* **Pod Affinity**: Cache pod runs close to the app pod for faster communication.
* **Pod Anti-Affinity**: Replicas of backend pods spread across nodes → no single node failure brings down all replicas.


### Topology Key

##### **Node-Level Topology Keys**

* `kubernetes.io/hostname`
  • Identifies the **node name**.
  • Used to spread or restrict Pods across individual nodes.

##### **Zone & Region Topology Keys**

*(used mostly for multi-AZ or multi-region clusters)*

* `topology.kubernetes.io/zone` ✅ (current stable key)
  • Represents the **Availability Zone** of a node.
  • Example: `us-east-1a`, `us-east-1b`, etc.

* `failure-domain.beta.kubernetes.io/zone` ⚠️ (deprecated)
  • Old version of the above, still seen in legacy clusters.

* `topology.kubernetes.io/region` ✅ (current stable key)
  • Represents the **region** of a node.
  • Example: `us-east-1`, `ap-south-1`, etc.

* `failure-domain.beta.kubernetes.io/region` ⚠️ (deprecated)
  • Legacy version of the above.


##### **Rack / Data Center / Custom Topology Keys**

*(used in on-prem or advanced environments)*

* `topology.kubernetes.io/rack`
  • Represents the **physical rack** in a data center.

* `topology.kubernetes.io/datacenter`
  • Represents the **data center** location.

* `topology.kubernetes.io/cluster`
  • Represents the **cluster identifier** in multi-cluster setups.

##### **Storage-Related Topology Keys**

*(used for CSI drivers, volumes, and storage classes)*

* `topology.kubernetes.io/zone` or driver-specific keys, e.g.:

  * `topology.gke.io/zone` (Google Cloud)
  * `topology.ebs.csi.aws.com/zone` (AWS EBS CSI driver)
  * `topology.disk.csi.azure.com/zone` (Azure Disk CSI driver)


##### **Custom / User-Defined Keys**

* You can define **custom topology keys** by labeling nodes:

  ```bash
  kubectl label node <node-name> topology.kubernetes.io/rack=rack-1
  ```

  Then use `topologyKey: topology.kubernetes.io/rack` in your Pod spec to spread Pods by rack.


##### **Summary Table**

| Category     | Topology Key                               | Description             | Status            |
| ------------ | ------------------------------------------ | ----------------------- | ----------------- |
| Node         | `kubernetes.io/hostname`                   | Node name               | Stable            |
| Zone         | `topology.kubernetes.io/zone`              | Availability Zone       | Stable            |
| Region       | `topology.kubernetes.io/region`            | Cloud Region            | Stable            |
| Zone (old)   | `failure-domain.beta.kubernetes.io/zone`   | Deprecated zone label   | Deprecated        |
| Region (old) | `failure-domain.beta.kubernetes.io/region` | Deprecated region label | Deprecated        |
| Rack         | `topology.kubernetes.io/rack`              | Data center rack        | Custom            |
| Datacenter   | `topology.kubernetes.io/datacenter`        | Data center identifier  | Custom            |
| Cluster      | `topology.kubernetes.io/cluster`           | Cluster identifier      | Custom            |
| Storage      | `topology.ebs.csi.aws.com/zone`, etc.      | CSI driver topology     | Provider-specific |

##### **Topology keys apply to nodes, not pods**
* The topologyKey in Pod affinity / anti-affinity or topology spread constraints refers to a node label (for example, the node’s zone, hostname, or rack).
* Pods themselves don’t have topology keys — instead, their scheduling behavior depends on how Kubernetes interprets node topology keys when placing them.

##### **“Pod-level topology” = controlling pod distribution behavior**
* When people say pod-level topology, they usually mean controlling how pods are spread or grouped relative to each other across topology domains.
* That’s done using these mechanisms:
    ```bash
    affinity:
      podAntiAffinity:
        requiredDuringSchedulingIgnoredDuringExecution:
        - labelSelector:
            matchLabels:
              app: web
          topologyKey: kubernetes.io/hostname
    ```
* Here, topologyKey defines the domain where pods should not be co-located.
* For example:
    * `kubernetes.io/hostname` → don’t schedule two same pods on the same node.
    * `topology.kubernetes.io/zone` → don’t schedule in the same zone.
* This is often described as pod-level topology control because it defines how pods relate to each other spatially.

##### **Pod Topology Spread Constraints**
```bash
topologySpreadConstraints:
- maxSkew: 1
  topologyKey: topology.kubernetes.io/zone
  whenUnsatisfiable: DoNotSchedule
  labelSelector:
    matchLabels:
      app: frontend
```
* This tells the scheduler to evenly spread pods matching the selector across all topology domains (like zones or nodes).
* It prevents all replicas of a Deployment or StatefulSet from landing on a single node or zone.
```bash
spec:
  affinity:
    podAntiAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchLabels:
            app: web
        topologyKey: kubernetes.io/hostname
  topologySpreadConstraints:
  - maxSkew: 1
    topologyKey: topology.kubernetes.io/zone
    whenUnsatisfiable: DoNotSchedule
    labelSelector:
      matchLabels:
        app: web
```
* Anti-affinity keeps two same pods off the same node.
* Spread constraint ensures even distribution across zones.
---
---

## Taint and Tolerations
```bash
kubectl taint node <node_name> key=value:Effect
kubectl taint node <node_name> key:Effect-
kubectl get pods -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.spec.tolerations}{"\n"}{end}' #List pods with their tolerations
```
**Taints and Tolerations** help **control which pods can be scheduled on which nodes**. They allow cluster administrators to **prevent certain workloads from running on specific nodes** while allowing exceptions when necessary. 


##### **Important:**  
Taints and tolerations **only apply during scheduling**. If a pod is already running on a node, adding a taint **will not remove the existing pod** (unless you use the **NoExecute** effect).  

![Alt Text](/images/taint-toleration1.png)

---

#### Understanding Taints  

A **Taint** is applied to a **node** to indicate that it **should not accept certain pods** unless they explicitly tolerate it.  

##### **How to Apply a Taint?**  
Taints are applied to nodes using the following command:  
```sh
kubectl taint nodes <node-name> <key>=<value>:<effect>
```
**Example:**  
```sh
kubectl taint nodes my-second-cluster-worker storage=ssd:NoSchedule
kubectl taint nodes my-second-cluster-worker2 storage=hdd:NoSchedule
```
- These commands taint `my-second-cluster-worker` to **only allow pods that tolerate `storage=ssd:NoSchedule`** and `my-second-cluster-worker2` to **only allow pods that tolerate `storage=hdd:NoSchedule`**.  

---

##### **Effects of a Taint**  
• `NoSchedule` → Pods without matching toleration will **not** be scheduled on the node.
• `PreferNoSchedule` → Scheduler will **try to avoid** placing Pods without toleration on the node, but it’s not strict.
• `NoExecute` → New Pods without toleration are not scheduled, and existing Pods without toleration are **evicted**.

---

#### Understanding Tolerations  

A **Toleration** is applied to a **pod**, allowing it to **bypass a node’s taint** and be scheduled on it.  

Tolerations are defined in a pod’s YAML under `spec.tolerations`:  

```yaml
tolerations:
  - key: "storage"
    operator: "Equal"
    value: "ssd"
    effect: "NoSchedule"
```

- This **tolerates** nodes that have the taint `storage=ssd:NoSchedule`, allowing the pod to be scheduled on them.  

- The **Effect in the toleration must match the Effect in the taint** for it to take effect.  

#### **Toleration Structure**

```yaml
tolerations:
- key: "dedicated"
  operator: "Equal"
  value: "gpu"
  effect: "NoSchedule"
  tolerationSeconds: 360
```

• Fields:
  * **key** → Must match taint key.
  * **operator** → Can be `Equal` (default) or `Exists`.
  * **value** → Compared to taint value (required with `Equal`).
  * **effect** → Must match taint effect.
  * **tolerationSeconds** → Used with `NoExecute` effect, specifies how long the Pod can remain before eviction.

* Operators:
  * **Equal**(default): The key and value **must exactly match** the taint.
  * **Exists**: Only the key needs to match, and the value is ignored.
  * **Exists with Effect**: Only key and effect matter; value is ignored.

**Example using Exists:**  
```yaml
tolerations:
  - key: "storage"
    operator: "Exists"
    effect: "NoSchedule"
```
- This toleration allows the pod to be scheduled on **any node that has a `storage` taint**, regardless of its value.  

* **Add Toleration via kubectl run**
```bash
kubectl run nginx-web --image=nginx --overrides='{
  "apiVersion": "v1",
  "spec": {
    "tolerations": [
      {
        "key": "dedicated",
        "operator": "Equal",
        "value": "web",
        "effect": "NoSchedule"
      }
    ]
  }
}'
```

* **Pod Stuck in Pending**
  ```bash
  kubectl describe pod <pod-name>
  ```
  * Check for taint mismatch.
  * Ensure the pod tolerations match node taints.

* **Node Unschedulable**
  * Remove taint or
  * Add matching tolerations to pods.
  * View Pod Tolerations
  ```bash
  kubectl get pods -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.spec.tolerations}{"\n"}{end}'
  ```


---

#### How Taints and Tolerations Work Together  

| **Node Taint** | **Pod Toleration** | **Effect** |
|---------------|-------------------|-----------|
| `storage=ssd:NoSchedule` | **Toleration exists** | ✅ Pod **can** be scheduled. |
| `storage=ssd:NoSchedule` | ❌ No toleration | ❌ Pod **cannot** be scheduled. |
| `storage=ssd:NoExecute` | **Toleration exists** | ✅ Pod **remains** on the node. |
| `storage=ssd:NoExecute` | ❌ No toleration | ❌ Pod **is evicted** from the node or not schedule. |

---

#### Demonstration  

##### **Cluster Setup for Demonstration**  

Before we begin applying taints and tolerations, let's first review our Kubernetes cluster setup. We have a total of **three nodes** in our KIND cluster:  

1. **my-second-cluster-control-plane** → This is the **control-plane node** responsible for managing the cluster.  
2. **my-second-cluster-worker** → This is a **worker node** where application workloads can be scheduled.  
3. **my-second-cluster-worker2** → This is another **worker node** available for scheduling workloads.  


##### **Applying Taints**  
```sh
kubectl taint nodes my-second-cluster-worker storage=ssd:NoSchedule
kubectl taint nodes my-second-cluster-worker2 storage=hdd:NoSchedule
```

##### **Verifying Taints**  
```sh
kubectl describe node my-second-cluster-worker | grep -i taint
kubectl describe node my-second-cluster-worker2 | grep -i taint
```

##### **Checking Pod Behavior Without Tolerations**  
```sh
kubectl run mypod --image=nginx
kubectl get pods
kubectl describe pod mypod
```
- The pod **remains in Pending state** because it does **not tolerate any taint**.

##### **Why the Pod Is Not Scheduled on the Control Plane Node?**  

After applying taints to our worker nodes, we will attempt to create a pod **without any tolerations** using:  

```sh
kubectl run mypod --image=nginx
```

Since **both worker nodes are tainted**, the pod will remain in a **Pending** state. However, it **will not be scheduled on the control plane node either**.  

##### **Checking Why the Pod Is Not Scheduled**  

To investigate, we will describe the pod using:  

```sh
kubectl describe pod mypod
```

This will show that **no suitable nodes were found for scheduling** due to the applied taints. Additionally, the **control plane node is already tainted by default**, preventing general workloads from running on it.  

##### **Verifying the Taint on the Control Plane Node**  

We can check the taints applied to the control plane node using:  

```sh
kubectl describe node my-second-cluster-control-plane | grep Taints
```

Expected output:  

```plaintext
Taints: node-role.kubernetes.io/control-plane:NoSchedule
```

This confirms that the **control plane node has a taint that prevents regular workloads from being scheduled on it** unless the pod has a corresponding toleration.  

##### **Why Were Manually Scheduling a Pod on the Control Plane in Possible**  

When we manually scheduled a pod on the control plane node by specifying the `nodeName` field in the pod definition. This **bypasses the Kubernetes scheduler entirely**. 

##### **Key takeaway:**  
- **Taints and tolerations affect scheduling decisions made by the scheduler.**  
- **When using manual scheduling (`nodeName` field), the scheduler is not involved, so taints are ignored.**  

##### **Scheduling Pods with Tolerations**  

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app1-deploy
spec:
  replicas: 3
  selector:
    matchLabels:
      app: app1
  template:
    metadata:
      labels:
        app: app1
    spec:
      tolerations:
        - key: "storage"
          operator: "Equal"
          value: "ssd"
          effect: "NoSchedule"
      containers:
        - name: nginx-container
          image: nginx
```
- This deployment **will now schedule pods on `my-second-cluster-worker`**.  

##### **Using the Exists Operator**  

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod2
spec:
  containers:
  - name: nginx
    image: nginx
  tolerations:
    - key: "storage"
      operator: "Exists"
      effect: "NoSchedule"
```
- This **pod can be placed on either `my-second-cluster-worker` or `my-second-cluster-worker2`**.  

##### **Applying Multiple Taints and Tolerations**  
```sh
kubectl taint nodes my-second-cluster-worker env=prod:NoSchedule
kubectl taint nodes my-second-cluster-worker2 env=dev:NoSchedule
```

```yaml
tolerations:
  - key: "storage"
    operator: "Exists"
    effect: "NoSchedule"
  - key: "env"
    operator: "Equal"
    value: "prod"
    effect: "NoSchedule"
```
- This pod tolerates **any `storage` taint** and **only `env=prod:NoSchedule`**.  

##### **Deleting Taints**  

Once a taint is applied to a node, it restricts pod scheduling based on the taint effect. If you need to **remove a taint** from a node, you can do so using the following syntax:  

```sh
kubectl taint nodes <node-name> <key>=<value>:<effect>- 
```

Here, the **`-` (hyphen) at the end** tells Kubernetes to **remove** the taint. 

---
#### **Example: Removing a Single Taint**  
Remove the `storage=ssd:NoSchedule` taint from **my-second-cluster-worker**:  
```sh
kubectl taint nodes my-second-cluster-worker storage=ssd:NoSchedule-  
```
Now, the **ssd taint is removed**, and pods that don’t have the matching toleration can be scheduled on this node.  

#### **Example: Removing All Applied Taints**  
We previously applied the following taints:  
```sh
kubectl taint nodes my-second-cluster-worker storage=ssd:NoSchedule
kubectl taint nodes my-second-cluster-worker2 storage=hdd:NoSchedule
kubectl taint nodes my-second-cluster-worker env=prod:NoSchedule
kubectl taint nodes my-second-cluster-worker2 env=dev:NoSchedule
```
To **remove all these taints**, run:  
```sh
kubectl taint nodes my-second-cluster-worker storage=ssd:NoSchedule-
kubectl taint nodes my-second-cluster-worker2 storage=hdd:NoSchedule-
kubectl taint nodes my-second-cluster-worker env=prod:NoSchedule-
kubectl taint nodes my-second-cluster-worker2 env=dev:NoSchedule-
```

#### **Verifying Taint Removal**  
After deleting the taints, verify that they are removed using:  
```sh
kubectl describe node my-second-cluster-worker | grep -i taints
kubectl describe node my-second-cluster-worker2 | grep -i taints
```
If no taints are listed in the output, the nodes are now available to schedule any pod without restrictions.

- If already running pod is crashed on a tainted node and pod not have toleration then pod will be schedule on different node.


#### Example with **NoExecute** (Evicting Pods)
* **1. What is a `NoExecute` taint?**
  * A `NoExecute` taint is a special taint applied to a node that **affects both existing and new pods**:
  * **Existing pods** that do **not tolerate** the taint are **evicted immediately** (or after a toleration period if specified).
  * **New pods** that do **not tolerate** the taint are **not scheduled** on the node.
  **Format:**
  ```text
  key=value:NoExecute
  ```
* **Example:**
  ```bash
  kubectl taint nodes node01 critical=true:NoExecute
  ```
  * This marks `node01` as “critical,” so any pod without a toleration for `critical=true:NoExecute` will be evicted or prevented from scheduling.

* **2. How tolerations interact with `NoExecute`**
  * A pod can tolerate a `NoExecute` taint using a **toleration**. This defines **how long the pod can stay** on a tainted node:
  ```yaml
  tolerations:
  - key: "critical"
    operator: "Equal"
    value: "true"
    effect: "NoExecute"
    tolerationSeconds: 60
  ```
  * **Behavior:**
    * `tolerationSeconds` = 60 → The pod can remain on the node for 60 seconds. After that, it is evicted automatically.
    * If no `tolerationSeconds` is specified → The pod can stay indefinitely as long as the toleration matches.

* **3. Timeline of `NoExecute` behavior**
  1. Node is tainted:
  ```bash
  kubectl taint nodes node01 critical=true:NoExecute
  ```
  2. New pod without toleration:
  * Will **not be scheduled** on `node01`.
  
  3. Existing pod without toleration:
  * Will be **evicted immediately** from `node01`.
  
  4. Existing pod with toleration:
  * Will **stay on the node**.
  * If `tolerationSeconds` is set, it will be evicted **after that time**.
  * If no `tolerationSeconds` is set, it can stay **indefinitely**.

* **4. Observing the behavior**
  * Use events to see eviction:
  ```bash
  kubectl get events --field-selector involvedObject.name=<pod-name>
  ```
  Example output:
  ```bash
  Normal   TaintManagerEviction   pod/temporary-critical   Marking for deletion Pod default/temporary-critical
  Normal   Killing                pod/temporary-critical   Stopping container busybox
  ```
  * This shows that the pod tolerated the taint for the specified period and was then evicted automatically.

* **5. Key points**
  * `NoExecute` affects **both existing and future pods**.
  * A pod **must have a matching toleration** to remain on the node.
  * `tolerationSeconds` controls **temporary toleration**.
  * Useful for:
    * Evicting non-critical pods during maintenance
    * Protecting critical nodes from general workloads


--- 

#### **Advanced Concepts of Kubernetes Taints & Tolerations**  

In Kubernetes, **taints and tolerations** control **where pods can be scheduled**. This analogy will help you **visualize** how these concepts work together with **scheduler preferences**.

##### **Scenario: A Cluster with Colored Nodes**  

Imagine we have **four nodes** in our Kubernetes cluster, each with different taints and behaviors:  

![Alt text](/images/16a.png)

1. **Green Node** 🟢  
   - **Taint:** `color=green:NoSchedule`  
   - **Effect:** Only pods that have a **matching toleration (`color=green`)** can be scheduled here.  
   - **Current Pods:** **Two green pods** (already running).  

2. **Blue Node** 🔵  
   - **Taint:** `color=blue:PreferNoSchedule`  
   - **Effect:** The **scheduler tries to avoid placing pods** here unless necessary.  
   - **Current Pods:** **One blue pod** (already running).  

3. **Purple Node** 🟣  
   - **Taint:** `color=purple:NoExecute`  
   - **Effect:** Any pod **without a matching toleration** will be **immediately evicted** from this node.  
   - **Current Pods:** **Two purple pods** (already running).  

4. **Untainted Node** ⚪  
   - **A new node has been added to the cluster.**  
   - **No taints applied** → Any pod **can be placed here** freely.  

##### **Effect of Taints on Existing Pods**  

- **Pods in the Green Node (`NoSchedule`) and Blue Node (`PreferNoSchedule`) remain unaffected.**  
- **Pods in the Purple Node (`NoExecute`) will be evicted if they lack a matching toleration.**  
  - If you wish to **delay eviction**, you can use the **`tolerationSeconds`** parameter. 

#### **Pod Placement Behavior for New Pods**  

![Alt text](/images/16a.png)

Now, let's introduce **four new pods** and observe where they are scheduled **after a new untainted node is added to the cluster**.  


##### **1️⃣ Yellow Pod**  
- **Toleration:** `color=yellow`  
- **Placement Possibilities:**  
  - **Untainted Node** (✅ **First Preference**)  
  - **Blue Node** (🔵 **Only if the untainted node is full**)  

📌 **Explanation:**  
Since **no node has a `color=yellow` taint**, this pod is treated like a normal pod.  
- **Untainted node** is the **first preference** because it has no restrictions.  
- **Blue node (`PreferNoSchedule`) is the second choice**—the scheduler **will try to avoid it** unless the untainted node **does not have enough capacity**.  


##### **2️⃣ Normal Pod (No Toleration)**  
- **Toleration:** None  
- **Placement Possibilities:**  
  - **Untainted Node** (✅ **First Preference**)  
  - **Blue Node** (🔵 **Only if the untainted node is full**)  

📌 **Explanation:**  
- Since this pod **has no toleration**, it **cannot be scheduled on Green or Purple nodes**.  
- **Untainted node is the first choice** since it has **no restrictions**.  
- **Blue node (`PreferNoSchedule`) is the backup option** if there is no space in the untainted node.  


##### **3️⃣ Green Pod**  
- **Toleration:** `color=green`  
- **Placement Possibilities:**  
  - **Green Node** (🟢 **First Preference**)  
  - **Untainted Node** (⚪ **Second Preference**)  
  - **Blue Node** (🔵 **Only if both green and untainted nodes are full**)  

📌 **Explanation:**  
- **Green node is the first preference** because it **has a matching taint (`color=green:NoSchedule`)**.  
- **If the green node is full**, the scheduler **places pods on the untainted node**.  
- In testing, when **one replica was created**, it was scheduled on the **green node**.  
- When **ten replicas were created**, Kubernetes **distributed them across the green and untainted nodes** based on available **resources**.  
- **Blue node (`PreferNoSchedule`) is only used if both green and untainted nodes are full.**  


##### **4️⃣ Blue Pod**  
- **Toleration:** `color=blue`  
- **Placement Possibilities:**  
  - **Untainted Node** (⚪ **First Preference**)  
  - **Blue Node** (🔵 **Only if untainted node is full**)  

📌 **Explanation:**  
- Since **PreferNoSchedule is a soft restriction**, the scheduler **tries to avoid the blue node** if there are better options.  
- In testing:  
  - **One replica** → Placed in the **untainted node** (scheduler prefers it over `PreferNoSchedule`).  
  - **Ten replicas** → Pods were **distributed equally** between the untainted and blue nodes.

---
#### **Why Should Pods with Tolerations Prefer Tainted Nodes?**  

We would want **pods with tolerations to be scheduled onto nodes with matching taints** rather than being placed on **untainted nodes**.  

Imagine if:  
- The **Green Node** is reserved for **Project A**, and  
- The **Untainted Node** belongs to **Project B**.  

Now, if a **Project A pod** is scheduled on the **Untainted Node (Project B's node)**, it **breaks the intended segregation**.  

##### **Real-World Use Cases for This Segregation**  
This type of isolation can be based on:  
- **Projects:** Different teams using dedicated nodes.  
- **Departments:** Keeping workloads from Finance, HR, and IT separate.  
- **Environments:** Ensuring **production** workloads do not mix with **development** ones.  
- **Criticality:** Reserving high-performance nodes for **mission-critical applications**.  

By combining **taints, tolerations, node affinity, and anti-affinity**, we can **enforce stricter placement policies** and **ensure workloads run on appropriate nodes** based on project requirements and business logic.
We will discuss **node affinity and anti-affinity** in the next lecture.

#### Key Takeaways  

- **Taints are applied to nodes**, **Tolerations are applied to pods**.  
- A **pod can only be scheduled** on a tainted node if it has a **matching toleration**.  
- **Three effects of taints:** `NoSchedule`, `PreferNoSchedule`, and `NoExecute`.  
- **Operator `Equal` (default)** requires an exact match, while **`Exists` ignores values**.  
- **Taints and tolerations work together** to control **pod placement and node access**.

---
#### **All Toleration Combinations**

##### **Equal Operator with All Fields**
```yaml
tolerations:
  - key: "mysize"
    operator: "Equal"
    value: "large"
    effect: "NoSchedule"
```
→ Pod tolerates **exactly** `mysize=large:NoSchedule`.

##### **Equal Operator, Empty Effect**
```yaml
tolerations:
  - key: "mysize"
    operator: "Equal"
    value: "large"
    effect: ""
```
→ If `effect` is empty, the Pod tolerates **all effects** for that key/value combination:
`mysize=large:NoSchedule`, `mysize=large:PreferNoSchedule`, and `mysize=large:NoExecute`.

##### **Exists Operator with Key + Effect**
```yaml
tolerations:
  - key: "mysize"
    operator: "Exists"
    effect: "NoSchedule"
```
→ Pod tolerates **any value** for key `mysize` **if** the taint’s effect is `NoSchedule`.

Example taints tolerated:
* `mysize=small:NoSchedule`
* `mysize=medium:NoSchedule`
* `mysize=large:NoSchedule`

##### **Exists Operator with Key Only**
```yaml
tolerations:
  - key: "mysize"
    operator: "Exists"
```
→ Pod tolerates **all taints** with key `mysize`, regardless of value or effect.
Covers:
* `mysize=small:NoSchedule`
* `mysize=small:NoExecute`
* `mysize=small:PreferNoSchedule`
* and so on.

##### **Exists Operator without Key**
```yaml
tolerations:
  - operator: "Exists"
```
→ Pod tolerates **every taint** on the node, no matter what key, value, or effect.
Basically — the Pod can run on **any tainted node**.

##### **NoExecute with tolerationSeconds**
```yaml
tolerations:
  - key: "mysize"
    operator: "Equal"
    value: "large"
    effect: "NoExecute"
    tolerationSeconds: 60
```
→ Pod can stay on a node tainted with `mysize=large:NoExecute` for **60 seconds**,
then it will be **evicted**.
If `tolerationSeconds` is not set, the Pod **stays indefinitely**.

---

#### **When Do We Need Taints & Tolerations?**
* Taints and tolerations are used to **control which Pods can be scheduled onto which nodes**.
* They help Kubernetes **influence scheduling decisions** and **isolate workloads** based on purpose, priority, or hardware capability.

##### **Key Reasons to Use Taints & Tolerations**
* To **dedicate specific nodes** for particular workloads — such as GPU, monitoring, or critical system components.
* To **prevent normal Pods** from being scheduled on sensitive nodes (e.g., control plane or master nodes).
* To **isolate environments**, ensuring that production workloads don’t mix with development or testing Pods.
* To **handle special hardware**, such as GPU or high-memory nodes, and keep unrelated workloads off those nodes.
* To **implement multi-tenant isolation** — separating workloads of different teams, customers, or SLAs.
* To **evict Pods gracefully** when nodes become unhealthy, go under maintenance, or need to be drained.
* To **influence scheduling** logic beyond labels and affinities — taints act as a *“keep away”* rule unless tolerated.

> 🧠 Think of **taints** as “repelling marks” on nodes and **tolerations** as “permissions” on Pods.

##### 📊 **Real-World Use Cases**

###### **Dedicated Nodes (Workload Isolation)**
* Reserve specific nodes for certain workloads like GPU-based ML jobs or critical services.
* **Example:**
  ```bash
  kubectl taint nodes gpu-node gpu=true:NoSchedule
  ```
* This prevents regular Pods from running on GPU nodes unless they have a matching toleration:
  ```yaml
  tolerations:
  - key: "gpu"
    operator: "Equal"
    value: "true"
    effect: "NoSchedule"
  ```
* **Result:**
  * Only GPU Pods can be scheduled on GPU nodes, ensuring resource isolation.

###### **Critical System Pods (Control-Plane Protection)**
* Prevent user workloads from running on control-plane nodes.
* Kubernetes automatically taints control-plane nodes:
  ```bash
  node-role.kubernetes.io/control-plane=:NoSchedule
  ```
* Regular Pods **cannot run** here.
* System Pods (like CoreDNS, kube-proxy, etc.) include tolerations to bypass this taint.
* **Result:**
  * System components stay isolated; normal workloads are kept off control-plane nodes.


###### **Temporary Evictions (Maintenance Mode)**
* Gracefully evict Pods during node maintenance or upgrade.
* **Example:**
  ```bash
  kubectl taint nodes node1 maintenance=true:NoExecute
  ```
* Pods without toleration are **immediately evicted**.
* Critical Pods can remain temporarily using `tolerationSeconds`:
  ```yaml
  tolerations:
  - key: "maintenance"
    operator: "Equal"
    value: "true"
    effect: "NoExecute"
    tolerationSeconds: 120
  ```
* **Result:**
  * Non-critical Pods are evicted immediately.
  * Critical Pods stay for 120 seconds before eviction.
* Perfect for rolling updates or maintenance windows.

###### **Dedicated Business Services (Example: Billing Service)**
* You want the **billing service** to run only on specific high-performance nodes, avoiding shared workloads.
* **Step 1 — Taint the dedicated nodes:**
  ```bash
  kubectl taint nodes node1 billing=true:NoSchedule
  ```
* **Step 2 — Add toleration in Pod spec:**
  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: billing-pod
  spec:
    containers:
    - name: billing-container
      image: billing-image
    tolerations:
    - key: "billing"
      operator: "Equal"
      value: "true"
      effect: "NoSchedule"
  ```
* **Result:**
  * Only billing Pods with the matching toleration can run on these nodes.

###### **Multi-Tenant or SLA-Based Isolation**
* Keep workloads of different teams or customers separate.
* **Example:**
  ```bash
  kubectl taint nodes teamA=true:NoSchedule
  kubectl taint nodes teamB=true:NoSchedule
  ```
* Pods for each team include a matching toleration — ensuring they never share nodes.
* **Result:**
  * Team workloads stay isolated, protecting performance and data boundaries.

###### **Special Hardware (GPU, FPGA, High-Memory Nodes)**
* Assign workloads that require special hardware only to nodes that have it.
* **Example:**
  ```bash
  kubectl taint nodes node1 gpu=true:NoSchedule
  ```
* Only GPU workloads that tolerate this taint will be scheduled there.
* **Result:**
  * General workloads won’t consume expensive GPU resources.

---

#### Taints on control plane nodes
* By default, **control-plane (master) nodes** are **tainted** so that **regular workloads do not run on them**.

* This is because the control-plane is meant to run critical system components (like `kube-apiserver`, `etcd`, `controller-manager`) and should not be overloaded by user workloads.

* When you check taints:
  ```sh
  kubectl describe node <control-plane-node> | grep Taints
  ```
  ```
  Taints: node-role.kubernetes.io/control-plane:NoSchedule
  ```
  (or in older versions)

  ```
  Taints: node-role.kubernetes.io/master:NoSchedule
  ```
  * **NoSchedule** → Regular Pods cannot be scheduled here unless they have a matching toleration.

##### Running Pods on the Control-Plane Node

* 2 Options

* **Option 1: Add Toleration to the Pod**
  Pod spec with toleration:
  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: pod-on-control-plane
  spec:
    tolerations:
    - key: "node-role.kubernetes.io/control-plane"
      operator: "Exists"
      effect: "NoSchedule"
    containers:
    - name: nginx
      image: nginx
  ```
  * `operator: Exists` means: *as long as the taint key exists, I tolerate it.*
  * This allows the Pod to be scheduled on the control-plane.

* **Option 2: Remove the Taint (Not Recommended for Production)**
  If you want all Pods to be allowed on the control-plane (useful for **single-node clusters like Minikube or kubeadm lab setups**):
  ```bash
  kubectl taint nodes <control-plane-node> node-role.kubernetes.io/control-plane:NoSchedule-
  ```
  ⚠️ But: In production, this is **not recommended**, since user workloads might consume CPU/memory needed for critical control-plane processes.


```bash
[vagrant@vbox ~]$ kubectl describe node mycluster1-control-plane | grep -i taints

Taints:             node-role.kubernetes.io/control-plane:NoSchedule
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: nginx  
  name: nginx   
spec:
  containers:   
  - image: nginx
    name: nginx 
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
  nodeName: "mycluster1-control-plane"
  tolerations:
  - key: "node-role.kubernetes.io/control-plane"
    operator: "Exists"
    effect: "NoSchedule"

status: {}
```

---
---

## Affinity + Taint and Tolerations

```bash
kubectl taint nodes node1 dedicated=ml:NoSchedule
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: demo-pod
spec:
  containers:
  - name: nginx
    image: nginx

  tolerations:
  - key: "dedicated"
    operator: "Equal"
    value: "ml"
    effect: "NoSchedule"

  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: disktype
            operator: In
            values:
            - ssd
    podAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
          - key: app
            operator: In
            values:
            - frontend
        topologyKey: "kubernetes.io/hostname"
```
#### **Why do we need Node Affinity when we already have Taints & Tolerations?**  

To answer this, let's recall what we observed while working with each approach.

##### **What We Observed with Taints & Tolerations**
- Taints **repel** pods from certain nodes unless they have a matching **toleration**.
- Even if a node has a **hard restriction** (NoSchedule or NoExecute), **pods with tolerations could still land on untainted nodes**.
- **Taints & Tolerations do not interact with node labels**.

##### **What We Observed with Node Affinity**
- **Node Affinity** ensures pods are scheduled only on nodes with **specific labels**.
- Pods **without affinity** can still land on labeled nodes.
- **Affinity does not prevent other pods from using the same node** unless combined with taints.

##### **Why Do We Need Both?**
The **ideal scheduling strategy** requires using **both** Taints & Tolerations **and** Node Affinity.  

- **Taints prevent unwanted pods from landing on a node.**  
- **Node Affinity ensures the right pods land on the right nodes.**  
- **Without both, pods might still land on unintended nodes.**  

##### **Key Takeaway**
- **If a pod has both Tolerations and Node Affinity, we can confidently predict where it will land.**  

### **Scenario: Taints & Tolerations vs. Node Affinity in Action**

![Alt text](/images/18a.png)


##### **Cluster Setup**
We have **two worker nodes** in our cluster:  

| **Node**  | **Labels**  | **Taints** |
|-----------|------------|------------|
| **Node1** | `storage=ssd` | `storage=ssd:NoSchedule` |
| **Node2** | `storage=hdd` | No taints |

**Important:**  
- **The default node labels** (e.g., `topology.kubernetes.io/region`, `kubernetes.io/hostname`, etc.) **vary based on where your cluster is running** (AWS, Azure, GCP, On-Premises etc..). Some of these labels are **added by the cloud provider**.

##### **Pod1: Using Only Tolerations**
```yaml
tolerations:
  - key: "storage"
    operator: "Equal"
    value: "ssd"
    effect: "NoSchedule"
```
✅ **Can be scheduled on Node1 (because it tolerates the taint).**  
✅ **Can also be scheduled on Node2 (because it is untainted).**  

🚨 **Problem:**  
- **The pod should ideally run on an SSD-backed node**, but it **might still land on Node2 (HDD)**.  
- **Tolerations alone do not force pods onto specific nodes.**  
---

#### Pod1:- Only taint-toleration
* While Node Affinity guides **where a pod wants to go**, **taints repel all other pods**.
```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: redis
  name: redis
spec:
  containers:
  - image: redis
    name: redis
  tolerations:
  - key: "gpu"
    operator: "Equal"
    value: "true"
    effect: "NoSchedule"
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: toleration-pod
spec:
  tolerations:
  - key: "dedicated"
    operator: "Equal"
    value: "experiment"
    effect: "NoSchedule"
  containers:
  - name: nginx
    image: nginx
```

>Note: This pod specification defines a toleration for the "gpu" taint with the effect "NoSchedule." This allows the pod to be scheduled on tainted nodes.

---


##### **Pod2: Using Only Node Affinity**
* **Node Affinity** specifies which nodes a pod **should prefer or require**.

```yaml
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
      - matchExpressions:
        - key: storage
          operator: In
          values:
            - ssd
```

❌ **Cannot be scheduled on Node1** – although Node1 has the required label (`storage=ssd`), it also has a taint (`storage=ssd:NoSchedule`) and the pod lacks a matching toleration.

❌ **Cannot be scheduled on Node2** – it doesn’t have the required label, so it fails the node affinity condition.

---

##### ⚠️ **Important Caveat**

* This pod does **not** include a **toleration** for `storage=ssd:NoSchedule`.
* Even though Node1 matches the affinity **label**, the **taint** on Node1 (`storage=ssd:NoSchedule`) will prevent the pod from being scheduled there.
* As a result, **Pod2 will remain in a `Pending` state** unless a matching toleration is added.

---

##### Common Misconception

> "Node affinity alone is enough to land a pod on a specific node."

---

##### **Pod3: No Tolerations, No Node Affinity**
✅ **Can be scheduled on Node2 (because it has no taints).**  
❌ **Cannot be scheduled on Node1 (because of the `NoSchedule` taint).**  

🚨 **Problem:**  
- **What if this pod actually requires SSD storage?**  
- **It lands on Node2 (HDD), which might not be appropriate for its workload.**  

---

#### **Final Solution: Using Both Taints & Tolerations + Node Affinity**
To ensure **Pod1 & Pod2** **only** land on Node1, we combine **Tolerations** and **Node Affinity**:

```yaml
tolerations:
  - key: "storage"
    operator: "Equal"
    value: "ssd"
    effect: "NoSchedule"
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
      - matchExpressions:
        - key: storage
          operator: In
          values:
          - ssd
```
✅ **Now, Pod1 & Pod2 will only be placed on Node1.**  
❌ **They will not land on Node2, even if there is space.**  

---

#### **Why Not Use Only Node Affinity?**
You might be thinking:  
💡 *"If my pods have affinity defined, won’t they already land on the right nodes?"*  

From an **administrator's perspective**, you want **complete control** over **which pods can run on which nodes**.  

##### **Key Reasons to Use Taints & Tolerations Along with Node Affinity**  
- **Taints prevent unintended workloads** from running on specialized or maintenance nodes.  
- **Without taints, pods without affinity** can still land on specialized nodes, disrupting workload segregation.  
- **Taints ensure only pods with explicit tolerations** are scheduled on critical or reserved nodes.  
- **Without taints, general workloads may consume resources** on nodes meant for high-priority applications.  
- **Taints provide an extra layer of enforcement** beyond node affinity, giving admins greater scheduling control.   

---

#### **Key Takeaways**
| **Concept** | **Behavior** |
|-------------|--------------|
| **Taints & Tolerations** | Tolerations allow pods to run on tainted nodes, but do not guarantee placement. |
| **Node Affinity** | Pods with affinity are forced to run on labeled nodes but do not repel other pods. |
| **Pods Without Both** | May end up on inappropriate nodes, leading to suboptimal performance. |
| **Best Practice** | Use **both** to get full control over pod scheduling. |

#### Limitations to Remember
* They cannot handle complex expressions like "AND" or "OR." 
* We use a combination of Taints, tolerance, and Node affinity.

##### Example- nodeAffinity +Taints

```bash
kubectl taint nodes node1 dedicated=finance:NoSchedule
kubectl label nodes node1 team=finance disktype=ssd
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: finance-app
spec:
  tolerations:
  - key: "dedicated"
    operator: "Equal"
    value: "finance"
    effect: "NoSchedule"
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: team
            operator: In
            values:
            - finance
          - key: disktype
            operator: In
            values:
            - ssd
  containers:
  - name: app
    image: myfinance:1.0
```
* How it works:
  * Taint prevents general pods from scheduling on `node1`.
  * Toleration allows this pod to land there.
  * NodeAffinity ensures pod **must land on nodes labeled `team=finance, disktype=ssd`**.

##### Example – PodAntiAffinity + Taints
Say you want **DB replicas** spread across nodes, but only on “db-dedicated” nodes.

```bash
kubectl taint nodes node2 dedicated=db:NoSchedule
kubectl taint nodes node3 dedicated=db:NoSchedule
kubectl label nodes node2 db=true
kubectl label nodes node3 db=true
```

* StatefulSet with Anti-Affinity

```yaml
affinity:
  podAntiAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
    - labelSelector:
        matchLabels:
          app: database
      topologyKey: kubernetes.io/hostname
tolerations:
- key: "dedicated"
  operator: "Equal"
  value: "db"
  effect: "NoSchedule"
```
*Effect:
  * Only **db pods** can tolerate the taint.
  * Node labels + anti-affinity ensure **replicas don’t land on the same node**.

##### Example – PodAffinity + Tolerations
Suppose you want **logging agents** (sidecar-like pods) to run on the same nodes as apps, but apps are tainted.
```bash
kubectl taint nodes node4 dedicated=app:NoSchedule
kubectl label nodes node4 app=true
```
###### Logging Agent Pod
```yaml
affinity:
  podAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
    - labelSelector:
        matchLabels:
          app: true
      topologyKey: kubernetes.io/hostname
tolerations:
- key: "dedicated"
  operator: "Equal"
  value: "app"
  effect: "NoSchedule"
```
* Effect:
  * Pod **must run on the same node as `app=myapp` pod**.
  * Toleration allows it to bypass the taint.


##### Best Practices
* Use **taints** for **hard isolation** (finance, db, GPU, prod).
* Add **tolerations** only for pods that should run on those nodes.
* Use **affinity/anti-affinity** for finer placement rules.
* Keep **soft (preferred) rules** where possible → prevents pods stuck in **Pending**.
* For multi-zone HA → prefer `topologyKey=topology.kubernetes.io/zone`.
* Combine with **PodDisruptionBudgets (PDBs)** for HA guarantees.

---
---

## TopologySpreadConstraints


---
---

## When labels changes how `nodeSelector`, `nodeAffinity` or `taint` behaves:


  

- `preferredDuringSchedulingIgnoredDuringExecution` means that once a pod is scheduled, it **won't be re-evaluated** if the node's labels change.  
- **Node affinity behaves the same way**—if a node’s labels change after scheduling, pods will **not be evicted or rescheduled** automatically.
- The **`NoExecute` taint effect supports re-evaluation**—if a node’s taints change and a pod doesn’t tolerate them, it will be evicted.  