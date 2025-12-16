### References:- 
- https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/
- https://devopscube.com/kubernetes-daemonset/
- https://spacelift.io/blog/kubernetes-cheat-sheet
- https://youtu.be/kvITrySpy_k
- https://spacelift.io/blog/kubernetes-daemonset *
- [Day 29: MASTER DaemonSet, Job & CronJob in Kubernetes](https://www.youtube.com/watch?v=gKbIkyE0TTI&ab_channel=CloudWithVarJosh)
- DaemonSet: [https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/) 


---
```bash
kubectl apply -f daemonset.yaml
kubectl get daemonset
kubectl get daemonset -A
kubectl describe daemonset <daemonset_name>
kubectl delete daemonset <daemonset_name>
kubectl edit daemonset <daemonset_name>
kubectl describe ds <daemonset_name> -n <ns>
kubectl rollout restart daemonset/<daemonset_name>
kubectl rollout status daemonset/<daemonset_name>
```
---
![Alt text](/images/29a.png)

• When new nodes join the Kubernetes cluster, the **DaemonSet controller** automatically creates a Pod on each new node.

• When a node is removed, Kubernetes automatically **garbage-collects (deletes)** the DaemonSet Pod that belonged to that node.

• If you manually delete a Pod created by a DaemonSet, the DaemonSet controller will **immediately recreate** it to maintain the desired state.

• If you delete the DaemonSet itself, Kubernetes will automatically **delete all Pods** that the DaemonSet had created.

• You can control whether a DaemonSet should run on control-plane nodes using **tolerations + nodeSelector**.  
  Example: kube-proxy runs on control-plane nodes because it includes the necessary tolerations.

---

### Static pod vs daemonsets
* Static pod node-scoped, daemonsets provide cluster level control.
* daemonsets --> scale in/out pod based on adding/removing of nodes.
* static pod --> each node kubelet manage a pod independently.
* to scale static pod we need to manally place a pod manifest into the kubelet monitored directory.


------------------------------------------------------------
### kube-proxy (DaemonSet)

• Kube-proxy manages **Service → Pod networking**.  
• It ensures traffic is routed only to **healthy Pods/nodes**.  
• Kube-proxy rewrites the **destination IP** for packets when a request hits a Service, sending it to the correct Pod.

------------------------------------------------------------
### How traffic flows:

Frontend Pod → accesses Service DNS name → CoreDNS resolves Service name → cluster IP → kube-proxy rewrites destination → forwards traffic to backend Pod

------------------------------------------------------------

### kube-proxy runs as a DaemonSet

• Ensures 1 kube-proxy Pod per node  
• Includes tolerations so it can also run on **control-plane nodes**  

---

### Common Use Cases for DaemonSets

DaemonSets are used when you need a Pod to run on **every node** in the cluster, or on a specific group of nodes. They guarantee node-level coverage, something ReplicaSets cannot provide.
Basically when you want to install any node level component.

##### 1. Logging Agents (Node + Application Logs)
  Run log collectors on every node so all container and system logs are shipped to a central logging backend.
  Examples:
  - Fluentd / Fluent Bit
  - Logstash
  - Splunk Forwarder
  - Humio
  - Filebeat

##### 2. Monitoring Agents (Node-Level Metrics)
  Deploy node-level monitoring exporters to gather CPU, memory, disk, and network metrics from every node.
  Examples:
  - Prometheus Node Exporter
  - Datadog Agent
  - New Relic Infrastructure Agent

##### 3. Kubernetes System Components
  Some core Kubernetes components run as DaemonSets.
  Example:
  - kube-proxy (handles service → pod networking and traffic routing)

##### 4. Networking Plugins (CNI)
  Network plugins must run on every node to set up routing rules, iptables, firewall configuration, or overlay networks.
  Examples:
  - Calico
  - Flannel
  - Weave
  - Cilium

##### 5. Security & Compliance Agents
  Security tools that need node-level visibility run as DaemonSets.
  Examples:
  - Falco (runtime security)
  - kube-bench (CIS benchmark scans)
  - Intrusion detection systems
  - Vulnerability scanners for PCI/PII-compliant nodes

##### 6. Storage Plugins (CSI)
  Storage drivers often require node components to be present everywhere.
  Examples:
  - CSI Node Plugin
  - NFS/GFPP mount helpers
  - Local PV provisioners

##### 7. Specialized Hardware Nodes (GPU, FPGA)
  DaemonSets can install drivers or system services only on specific nodes that match labels.
  Examples:
  - NVIDIA GPU drivers daemonset
  - FPGA device plugins

##### 8. Telemetry and Observability Collectors
  Collectors that gather traces, events, and logs at the node level.
  - Examples:
    - OpenTelemetry Collector (DaemonSet mode)  


##### Why Use DaemonSets?
  - Ensures a Pod runs **on every node**
  - Automatically schedules Pods on **new nodes** as they join
  - Automatically removes Pods when nodes leave
  - Useful for system-level services that support the entire cluster
  - Guaranteed node coverage → **ReplicaSets cannot guarantee this**

DaemonSets are ideal when you need consistent, reliable node-level functionality across the entire Kubernetes cluster.


---

### **DaemonSets We Have Already Seen: kube-proxy**

We have actually been working with a DaemonSet since early in this course.

- **Kube-proxy**, a critical system component responsible for **Service-to-Pod networking** inside the cluster.

- **kube-proxy** is deployed as a **DaemonSet** in Kubernetes to ensure that **every node** has the necessary networking functionality.
- You can verify this in your cluster using:

```bash
kubectl get daemonsets.apps -n kube-system kube-proxy
kubectl get pods -n kube-system -o wide | grep -i kube-proxy
```

---

### **DaemonSets for CNIs and CSIs**

Most cloud-native networking (CNI) and storage (CSI) plugins are deployed using DaemonSets:

- **CNI Plugins:**  
  AWS VPC CNI, Azure CNI, Calico, Flannel, Cilium
- **CSI Node Plugins:**  
  AWS EBS CSI, AWS EFS CSI, GCP PD CSI, and others

By deploying these plugins as DaemonSets, Kubernetes ensures that **every new or existing node** has the necessary networking and storage components running to handle Pod traffic, volume mounts, and attachments properly.

---

### **DaemonSets and Control Plane Nodes**

**Will a DaemonSet run on the control plane node?**  
The answer depends on the cluster setup:

- In **cloud-managed Kubernetes** services (like EKS, GKE, AKS), the **control plane nodes are fully managed** and **isolated**.  
  Therefore, **DaemonSets that you deploy for monitoring, logging, or security will not run on the control plane nodes**.

- In a **self-managed cluster** (like a cluster created with kubeadm or KIND), **DaemonSets can run on control plane nodes** if:
  - The control plane node has a **taint** (typically `node-role.kubernetes.io/control-plane:NoSchedule`), **and**
  - The DaemonSet has a **toleration** allowing it to tolerate that taint.

In our **KIND cluster**, we have three nodes:

```bash
kubectl get nodes
```

Output:

```
NAME                              STATUS   ROLES           AGE   VERSION
my-second-cluster-control-plane   Ready    control-plane   40d   v1.31.4
my-second-cluster-worker          Ready    <none>          40d   v1.31.4
my-second-cluster-worker2         Ready    <none>          40d   v1.31.4
```

The **control plane node** (`my-second-cluster-control-plane`) has the following taint:

```
Taints: node-role.kubernetes.io/control-plane:NoSchedule
```

You can verify it by running:

```bash
kubectl describe node my-second-cluster-control-plane
```

Now, the `kube-proxy` DaemonSet has the following toleration:

```yaml
tolerations:
  - operator: Exists
```

**What does it mean when `operator: Exists` is used without a `key`?**

- **No key specified** means the pod **tolerates *any* taint on the node** — regardless of key, value, or taint source.
- As long as the `effect` matches (or if no effect is specified, it tolerates any effect too), **the pod can be scheduled**.

In kube-proxy's case:

- kube-proxy must run on **all nodes** — control plane nodes, worker nodes, tainted nodes — everywhere.
- Kubernetes can't predict what taints the nodes may have (some clusters are custom).
- Instead of listing specific taint keys, kube-proxy's DaemonSet says:
  > "I don't care what taints the node has. I need to run there anyway."

---

### **Key Takeaways**

- A **DaemonSet** ensures that a specific Pod runs on **every node** in the cluster.
- It automatically handles Pod scheduling on new nodes and cleans up Pods from removed nodes.
- DaemonSets are heavily used for system-level components such as networking, storage, logging, monitoring, and security.
- On **managed cloud clusters**, DaemonSets generally **do not run on control plane nodes**.
- On **self-managed clusters**, DaemonSets **can run on control plane nodes** if appropriate **tolerations** are specified.


---

## **Demo: DaemonSet - Deploying a Dummy Logging Agent**

In this demo, we will deploy a **dummy logging agent** across all nodes in our Kubernetes cluster using a **DaemonSet**.  
The agent will simulate log collection by printing a message every 30 seconds.

As a best practice, **system-level DaemonSets are typically deployed into their own dedicated namespace** for better organization and access control.

---

### **Step 1: Create a New Namespace for Logging**

We will create a new namespace called `logging-ns` to isolate our DaemonSet:

```bash
kubectl create namespace logging-ns
```

---

### **Step 2: Switch the Context to the New Namespace**

To avoid typing `-n logging-ns` with every command, we will temporarily set the default namespace in our current context:

```bash
kubectl config set-context --current --namespace=logging-ns
```

This ensures that all our upcoming commands automatically target the `logging-ns` namespace.

---

### **Step 3: Apply the DaemonSet Manifest**

Here’s the manifest (`ds.yaml`) for our dummy logging agent:

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: log-collector
  namespace: logging-ns  # Best practice: Deploy system-level agents into a dedicated namespace for better management and isolation.
  labels:
    app: log-collector  # Label to identify the DaemonSet and its Pods.
spec:
  selector:
    matchLabels:
      app: log-collector  # Ensures Pods managed by this DaemonSet match this label.
  template:
    metadata:
      labels:
        app: log-collector  # Labels assigned to Pods created by this DaemonSet.
    spec:
      tolerations:
        - key: "node-role.kubernetes.io/control-plane"
          operator: "Exists"
          effect: "NoSchedule"
          # This toleration allows Pods created by this DaemonSet to be scheduled even on control-plane nodes,
          # which are tainted by default with "NoSchedule" to block regular workloads.
      containers:
        - name: log-collector
          image: busybox  # Using a lightweight busybox image to simulate a logging agent.
          command: ["/bin/sh", "-c", "while true; do echo 'Collecting logs...'; sleep 30; done"]
          # The container runs an infinite loop that prints a message every 30 seconds, simulating log collection behavior.
          resources:
            requests:
              cpu: "50m"
              memory: "50Mi"
              # Resource requests ensure the scheduler reserves at least this much CPU and memory for the container.
            limits:
              cpu: "100m"
              memory: "100Mi"
              # Resource limits prevent the container from consuming more than the specified amount of CPU and memory.
          volumeMounts:
            - name: varlog
              mountPath: /var/log
              # Mounts the host's /var/log directory into the container, simulating real-world log collection from the node.
      volumes:
        - name: varlog
          hostPath:
            path: /var/log
            type: Directory
            # A hostPath volume that provides direct access to the host machine’s /var/log directory.
            # In production, instead of just echoing logs inside the container, a real logging agent (like Fluentd, Fluent Bit, or Filebeat)
            # would collect logs from /var/log and ship them to a centralized destination such as:
            # - A file storage server (e.g., NFS, EFS)
            # - An object storage service (e.g., AWS S3, Google Cloud Storage)
            # - A logging service (e.g., ElasticSearch, Loki, Splunk)
            #
            # This ensures logs are persisted, searchable, and available for audits, troubleshooting, and monitoring.

```

Apply the DaemonSet using:

```bash
kubectl apply -f ds.yaml
```

> **Note:**  
> Here we are using a `hostPath` volume to simulate access to the node’s `/var/log` directory.  
> **In production environments**, logs would typically be shipped to a **centralized file storage** (like NFS), an **object storage** (like AWS S3), or **streamed directly** to a logging platform (like ElasticSearch, Loki, or a cloud-native log service).

---

### **Step 4: Verify the DaemonSet**

Check the DaemonSet status:

```bash
kubectl get daemonset
```

Describe the DaemonSet for more details:

```bash
kubectl describe daemonset log-collector
```

You should see that **one Pod is scheduled on every node** in the cluster.

Additionally, you can confirm the Pod placement with:

```bash
kubectl get pods -o wide
```

Example output:

```
NAME                  READY   STATUS    RESTARTS   AGE   IP            NODE                              NOMINATED NODE   READINESS GATES
log-collector-4krdf   1/1     Running   0          11m   10.244.1.26   my-second-cluster-worker          <none>           <none>
log-collector-bzsln   1/1     Running   0          11m   10.244.2.43   my-second-cluster-worker2         <none>           <none>
log-collector-nvvz4   1/1     Running   0          11m   10.244.0.5    my-second-cluster-control-plane   <none>           <none>
```

You can observe that **each node**, including the **control-plane node**, is running an instance of the log-collector Pod.  
This is possible because **we added a toleration** for the `control-plane` taint in the DaemonSet spec.

---

### **Bonus Exercise: Observe DaemonSet Pod Re-Creation**

To understand how the **DaemonSet controller** maintains the Pods:

1. **Manually delete a Pod** created by the DaemonSet:

```bash
kubectl delete pod <pod-name>
```

2. Immediately run:

```bash
kubectl get pods -o wide
```

We can also delete the namespace:
```bash
kubectl delete namespace <namespace>
```
You will see that **Kubernetes automatically recreates the missing Pod** on the same node.

> This demonstrates that the **DaemonSet controller constantly monitors the cluster** and ensures that **the desired state is maintained** — one Pod on every node.

---

![](/images/image-4-44.png)
![image](https://github.com/piyushsachdeva/CKA-2024/assets/40286378/bb803dc2-f9ab-4fe3-a0bb-0eacdfcf3ce0)

---

### Scoping DaemonSets to Specific Nodes
* DaemonSets don’t always need to run on every node.  
* You can restrict them to run only on selected nodes using:
  • `spec.template.spec.nodeSelector`  
  • `spec.template.spec.affinity`  
* Example: Fluentd DaemonSet restricted to nodes with log collection enabled:

```bash
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd
spec:
  selector:
    matchLabels:
      name: fluentd
  template:
    metadata:
      labels:
        name: fluentd
    spec:
      nodeSelector:
        log-collection-enabled: "true"
      containers:
      - name: fluentd-elasticsearch
        image: quay.io/fluentd_elasticsearch/fluentd:latest

Before applying the DaemonSet, label the node:
  kubectl label node minikube-m02 log-collection-enabled=true
Now apply the manifest:
  kubectl apply -f fluentd.yaml
Check the DaemonSet:
  kubectl get daemonsets
You’ll see DESIRED = 1 because only one node matches the selector.
Confirm the Pod’s node placement:
  kubectl get pod -o wide
The Pod should be scheduled on the labeled node.
```

---

### DaemonSet Best Practices
```
1. Use DaemonSets only when scaling follows node count:
   If Pod count is independent of node count, use Deployments or ReplicaSets instead.

2. DaemonSet Pods must use restartPolicy = Always 
   Required so Pods come back when nodes reboot.

3. Do not manually edit or delete DaemonSet Pods: 
   The DaemonSet controller will recreate deleted Pods.  
   Manual modification can cause inconsistencies.

4. Use rollbacks to revert DaemonSet updates quickly:  
   Rollouts and rollbacks provide safer, more reliable changes for system services.

5. Apply proper resource requests/limits and security context:  
   Important since DaemonSets often run critical system-level agents.

6. Use node labels and affinity rules to precisely target nodes:  
   Especially useful for GPU nodes, storage nodes, logging nodes, or secured nodes (PCI, PII).
```

---

* Node labels updated → Pods are added/removed based on label matching.
* Updating a DaemonSet
  - You can modify the Pod template in the DaemonSet.
  - Limitations:
      - Some fields in existing Pods cannot be updated.
      - The DaemonSet controller applies the original template when new nodes are added.

* Deleting a DaemonSet:
  - If a DaemonSet is deleted, the Pods it created are automatically cleaned up.
  - `--cascade=orphan` → Leaves Pods running on nodes.
  - Re-creating a DaemonSet with the same selector can adopt existing Pods.

```yaml
apiVersion: apps/v1
kind:  DaemonSet
metadata:
  name: nginx-ds
  labels:
    env: demo
spec:
  template:
    metadata:
      labels:
        env: demo
      name: nginx
    spec:
      containers:
      - image: nginx
        name: nginx
        ports:
        - containerPort: 80
  selector:
    matchLabels:
      env: demo
```


#### Example DaemonSet Manifest

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: log-agent
  namespace: kube-system
spec:
  selector:
    matchLabels:
      name: log-agent
  template:
    metadata:
      labels:
        name: log-agent
    spec:
      containers:
      - name: fluentd
        image: fluentd:latest
        resources:
          limits:
            memory: "200Mi"
            cpu: "200m"
        volumeMounts:
        - name: varlog
          mountPath: /var/log
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
```
* In this example, `fluentd` is deployed on every node to collect logs from `/var/log`.

#### Updating a DaemonSet
* By default, Pods are replaced **one at a time** (rolling update).
* DaemonSets support **two update strategies** under `.spec.updateStrategy`:
  * **RollingUpdate** (default): Replace Pods gradually.
  * **OnDelete**: New Pods are created only after you manually delete the old ones.

* **OnDelete**
* DaemonSet controller does **not** automatically replace Pods after a template change.
* You must **manually delete Pods** → only then the controller creates new ones.
* Best for **critical system components** (e.g., networking plugins) where you want **full control** over restarts.
```bash
kubectl create namespace logging
```
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd-ondelete
  namespace: logging
spec:
  updateStrategy:
    type: OnDelete
  selector:
    matchLabels:
      app: fluentd
  template:
    metadata:
      labels:
        app: fluentd
    spec:
      containers:
      - name: fluentd
        image: quay.io/fluentd_elasticsearch/fluentd:v2.5.2
        resources:
          requests:
            cpu: 100m
            memory: 200Mi
          limits:
            cpu: 200m
            memory: 200Mi
```
* Get image
  ```bash
  kubectl describe daemonset fluentd-ondelete -n logging | grep -i image
  Image:      quay.io/fluentd_elasticsearch/fluentd:v2.5.2

  kubectl describe pods fluentd-ondelete-b2j5l -n logging | grep -m 1 Image
  Image:          quay.io/fluentd_elasticsearch/fluentd:v2.5.2
  ```

* update image
  ```bash
  kubectl set image daemonset fluentd-ondelete fluentd=ewok/fluentd:v2.5.1 -n logging

  kubectl describe daemonset fluentd-ondelete -n logging | grep -i image
  Image:      ewok/fluentd:v2.5.1

  kubectl describe pods fluentd-ondelete-b2j5l -n logging | grep -m 1 Image
  Image:          quay.io/fluentd_elasticsearch/fluentd:v2.5.2
  ```

* If you change the image version here, Pods **stay old** until you delete them:
  ```bash
  kubectl delete pod -l app=fluentd -n logging

  kubectl describe pods fluentd-ondelete-p5pqz -n logging | grep -m 1 Image
  Image:          ewok/fluentd:v2.5.1
  ```
* DaemonSet controller then replaces them with new Pods.

---

* **RollingUpdate Strategy (default)**
* When you update the DaemonSet Pod spec, old Pods are automatically killed and replaced with new ones.
* Controlled by `rollingUpdate.maxUnavailable`:
    * `maxUnavailable: 1` → ensures that only 1 Pod is unavailable at a time.
    * You can use absolute numbers (e.g., `2`) or percentages (e.g., `20%`).
* Ensures **gradual replacement** across nodes.
```bash
kubectl create namespace logging
```
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd-rolling
  namespace: logging
spec:
  updateStrategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1   # can be "1" or "20%"
  selector:
    matchLabels:
      app: fluentd
  template:
    metadata:
      labels:
        app: fluentd
    spec:
      containers:
      - name: fluentd
        image: quay.io/fluentd_elasticsearch/fluentd:v2.5.2
        resources:
          requests:
            cpu: 100m
            memory: 200Mi
          limits:
            memory: 200Mi
```
```bash
kubectl set image daemonset fluentd-rolling fluentd=quay.io/fluentd_elasticsearch/fluentd:v2.6.0 -n logging

kubectl rollout status daemonset fluentd-rolling -n logging

kubectl describe pod fluentd-rolling-lb926 -n logging | grep -m 1 Image
Image:          quay.io/fluentd_elasticsearch/fluentd:v2.6.0

kubectl rollout undo daemonset fluentd-rolling -n logging #If something goes wrong with an update

kubectl describe pod fluentd-rolling-wz576 -n logging | grep -m 1 Image
Image:          quay.io/fluentd_elasticsearch/fluentd:v2.5.2

kubectl rollout undo daemonset fluentd-rolling -n logging --to-revision=2

kubectl delete daemonset fluentd-rolling -n logging

kubectl delete daemonset fluentd-rolling -n logging --cascade=false #Keep Pods running but remove DaemonSet controller (orphan Pods)
#This is useful if you want Pods to keep running independently after deleting the DaemonSet.

kubectl delete daemonset fluentd-rolling -n logging --cascade=false 
warning: --cascade=false is deprecated (boolean value) and can be replaced with --cascade=orphan.
daemonset.apps "fluentd-rolling" deleted from logging namespace
```

* **Different Ways to Trigger a RollingUpdate in DaemonSets**
Rolling updates happen **whenever the Pod template changes**.
Here are the common ways:

1. **Update the image**
```bash
kubectl set image daemonset fluentd-rolling fluentd=quay.io/fluentd_elasticsearch/fluentd:v2.6.0 -n logging
```
This replaces Pods gradually (following `maxUnavailable`).

2. **Patch the DaemonSet**
Apply a patch to change spec fields (like image, env, args):
```bash
kubectl patch daemonset fluentd-rolling -n logging \
  -p '{"spec": {"template": {"spec": {"containers": [{"name": "fluentd","image":"quay.io/fluentd_elasticsearch/fluentd:v2.6.1"}]}}}}'
```

3. **Edit the DaemonSet YAML**
```bash
kubectl edit daemonset fluentd-rolling -n logging
```
Modify the Pod spec (image, env, resource limits). The controller starts rolling out updates automatically.

4. **Apply a new manifest**
If you have a YAML with changes:
```bash
kubectl apply -f fluentd-rolling.yaml
```

5. **Force a rolling restart (without changing spec)**
Sometimes you want to restart Pods to pick up ConfigMap/Secret changes without changing the container image. You can do this with an **annotation trick**:
```bash
kubectl patch daemonset fluentd-rolling -n logging \
  -p "{\"spec\":{\"template\":{\"metadata\":{\"annotations\":{\"kubectl.kubernetes.io/restartedAt\":\"$(date +%Y-%m-%dT%H:%M:%S%z)\"}}}}}"

kubectl rollout restart fluentd-rolling -n logging
```
This updates the Pod template’s annotation → Kubernetes treats it as a spec change → triggers a rolling restart.


* Use **RollingUpdate** for most DaemonSets (monitoring, logging).
* Use **OnDelete** for sensitive system-level DaemonSets (CNI plugins).
* Always check rollout progress (`kubectl rollout status`).
* Keep **resource limits** defined to avoid node pressure during rollout.
* Use `maxUnavailable=0` for **zero downtime upgrades** (but slower rollout).

#### Taint-toleration
```bash
kubectl taint nodes k8s-worker-2 app=fluentd-logging:NoExecute
kubectl describe node k8s-worker-2 | grep Taints
```
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd
  namespace: kube-system
spec:
  selector:
    matchLabels:
      app: fluentd
  template:
    metadata:
      labels:
        app: fluentd
    spec:
      tolerations:
      - key: "app"
        operator: "Equal"
        value: "fluentd-logging"
        effect: "NoExecute"
      containers:
      - name: fluentd
        image: fluent/fluentd:latest
```
```bash
kubectl apply -f fluentd-daemonset.yaml
kubectl get pods -o wide -n kube-system -l app=fluentd
```
* This allows **Fluentd pods** to still run on `k8s-worker-2` despite the taint.
* If you **remove this toleration**, the DaemonSet pod running on `k8s-worker-2` will be evicted (as you noticed).

#### nodeSelector
* Normally, a DaemonSet schedules one Pod per node across the entire cluster. But sometimes, you don’t want it on *every node*, only on a subset of nodes (e.g., logging agents only on worker nodes). That’s where **nodeSelector** (or affinities/taints) comes in.
```bash
kubectl label node k8s-worker-1 type=platform-tools
kubectl label node k8s-worker-2 type=platform-tools
```
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: splunk-monitoring-agent
  labels:
    app: logging
spec:
  selector:
    matchLabels:
      app: logging
  template:
    metadata:
      labels:
        app: logging
    spec:
      nodeSelector:
        type: platform-tools  # Ensures it runs on specific nodes with the label "platform-tools"
      containers:
        - name: splunk-monitoring-agent
          image: splunk:latest
          ports:
            - containerPort: 8088  # Assuming the agent exposes a port
          volumeMounts:
            - name: splunk-config
              mountPath: /etc/splunk  # Specify where the config should be mounted in the container
      volumes:
        - name: splunk-config
          configMap:
            name: splunk-config-map  # Ensure you have a ConfigMap named splunk-config-map
```
* The controller looks at all nodes.
* Only nodes that match the selector get a Pod.
* Nodes without that label are skipped (DaemonSet won’t schedule Pods there).
* If you add a new node with that label, DaemonSet will automatically create a Pod on it.

#### Affinity
* `nodeSelector`: Simple equality-based matching (`key=value`). Very limited.
* `nodeAffinity`: More expressive, supports operators (`In`, `NotIn`, `Exists`, `DoesNotExist`, `Gt`, `Lt`)

* `requiredDuringSchedulingIgnoredDuringExecution`: **Hard rule** – if no node matches, the Pod won’t be scheduled.
* `preferredDuringSchedulingIgnoredDuringExecution`: **Soft rule** – scheduler tries to place Pods there, but falls back if unavailable.

```yaml
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
      - matchExpressions:
        - key: type
          operator: In
          values:
          - platform-tools
    preferredDuringSchedulingIgnoredDuringExecution:
    - weight: 1
      preference:
        matchExpressions:
        - key: instance-type
          operator: In
          values:
          - t2.large
```
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd
  namespace: logging
  labels:
    app: fluentd-logging
spec:
  selector:
    matchLabels:
      name: fluentd
  template:
    metadata:
      labels:
        name: fluentd
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: type
                operator: In
                values:
                - platform-tools
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 1
            preference:
              matchExpressions:
              - key: instance-type
                operator: In
                values:
                - t2.large
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
      terminationGracePeriodSeconds: 30
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
```
* You can combine **affinity** + **tolerations** in DaemonSets.


### DaemonSet vs Deployment
* **Deployment**: Scales Pods arbitrarily across nodes, focuses on stateless apps.
* **DaemonSet**: Ensures **one Pod per node**, ideal for infrastructure-level services.
* **Deployment + HPA**: Used for scalable workloads (e.g., web apps).
* **DaemonSet**: Used for node-level background tasks.



#### Multiple DaemonSets for One Daemon
  * Sometimes, you need the *same kind daemon* (e.g., Fluentd or a monitoring agent) but with **different configurations** depending on hardware, node pool, or workload type.
  * Instead of trying to overload a single DaemonSet with complex logic, you can define **separate DaemonSets**, each scoped to the right nodes.

* **Use Cases**
  1. **Different Resource Requirements**
     * Example: Nodes with GPUs may need higher memory/CPU allocations for monitoring agents.
     * Solution: One DaemonSet with `resources.requests` tailored for GPU nodes, another with lighter requests for standard nodes.

  2. **Different Configuration Flags**
     * Example: A logging agent needs to parse logs differently on database nodes vs application nodes.
     * Solution: Deploy two DaemonSets, each mounting different config files via ConfigMap.

  3. **Different Hardware Types**
     * Example: Bare-metal nodes vs virtual machines may need different system daemons or driver-related flags.
     * Solution: Use multiple DaemonSets with `nodeSelector` / `nodeAffinity` targeting the correct hardware labels.

  4. **Mixed Operating Systems / Architectures**
     * Example: A cluster with both Linux and Windows nodes.
     * Solution: One DaemonSet runs the daemon container built for Linux (`nodeSelector: kubernetes.io/os=linux`), another for Windows.

* **Implementation**
  * Use **labels and selectors** to scope each DaemonSet:
    ```yaml
    spec:
      template:
        spec:
          nodeSelector:
            hardwareType: gpu
    ```
  * Alternatively, use **affinity rules** or **tolerations** for tainted node pools.


#### DaemonSet Best Practices
* **Restart Policy**
  * Set to `Always` or leave unspecified.
  * Ensures DaemonSet pods **automatically restart** if they fail.

* **Namespace Isolation**
  * Deploy each DaemonSet in its **own namespace**.
  * Helps in **resource management** and avoids conflicts with other DaemonSets.

* **Scheduling Preferences**
  * Prefer `preferredDuringSchedulingIgnoredDuringExecution` over `requiredDuringSchedulingIgnoredDuringExecution`.
  * Reason: If required nodes are unavailable, pods won’t start at all. Preferred scheduling is **flexible**.

* **DaemonSet Priority**
  * Set `priorityClassName` to **high priority** (≥ 10000).
  * Prevents critical DaemonSet pods from being **evicted** under resource pressure.

* **Pod Selector**
  * Must match `.spec.template.labels`.
  * Ensures **correct pods are managed** by the DaemonSet controller.

* **minReadySeconds**
  * Defines how long Kubernetes should wait before creating the next pod.
  * Guarantees that **existing pods are ready** before new pods are rolled out during updates.

* **Node Deployment**
  * Use proper **labels and node selectors** to deploy pods only on intended nodes.
  * Useful for **specialized workloads**, e.g., monitoring agents or network plugins.

* **Resource Requests/Limits**
  * Keep CPU and memory requests **minimal** because pods run on every node.
  * Prevents unnecessary resource consumption cluster-wide.

* **PodDisruptionBudgets (PDB)**
  * Define PDB to control **eviction during maintenance or upgrades**.
  * Ensures that **enough DaemonSet pods remain running** to maintain functionality.

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: splunk-monitoring-agent
  namespace: monitoring
spec:
  selector:
    matchLabels:
      app: logging
  updateStrategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
  minReadySeconds: 10
  template:
    metadata:
      labels:
        app: logging
    spec:
      restartPolicy: Always
      priorityClassName: high-priority
      nodeSelector:
        type: platform-tools
      affinity:
        nodeAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 1
            preference:
              matchExpressions:
              - key: type
                operator: In
                values:
                - platform-tools
      containers:
        - name: splunk-monitoring-agent
          image: splunk:latest
          resources:
            requests:
              cpu: 50m
              memory: 64Mi
            limits:
              cpu: 200m
              memory: 256Mi
---
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: splunk-agent-pdb
  namespace: monitoring
spec:
  minAvailable: 80%
  selector:
    matchLabels:
      app: logging
```


#### Purpose of Pod Priority
  * Determines the **importance of a pod** relative to other pods.
  * Higher priority pods are **less likely to be preempted** when resources are scarce.
  * Useful for critical system components.
  * Critical system components (like `kube-proxy` or CNI pods) often use high-priority classes.
  * Ensures **critical DaemonSet pods remain running**, even under node resource pressure.
  * Useful for components like network plugins, logging agents, or monitoring daemons that must always be active.

* **PriorityClass Object**
  * Defines the priority value of a pod.
  * Can be any **32-bit integer ≤ 1 billion**.
  * Higher value → higher priority.

* **Example PriorityClass Creation**
  ```yaml
  apiVersion: scheduling.k8s.io/v1
  kind: PriorityClass
  metadata:
    name: high-priority
  value: 100000
  globalDefault: false
  description: "daemonset priority class"
  ```

  * Check existing priority classes:
    ```bash
    kubectl get priorityclass
    ```

* **Using PriorityClass in DaemonSet**
  ```yaml
  spec:
    priorityClassName: high-priority
    containers:
      ...
    terminationGracePeriodSeconds: 30
    volumes:
      ...
  ```

* **Built-in Critical Priority Classes**
  * `system-node-critical` → highest priority for node-level essential pods; not evicted under any circumstances.
  * `system-cluster-critical` → high priority for cluster-critical pods.

#### Some Troubleshooting
* **When is a DaemonSet Unhealthy?**
  * Any pod in the DaemonSet is **not running on a node**.
  * Common pod statuses causing issues:
    * `CrashLoopBackOff` → pod repeatedly crashing.
    * `Pending` → pod cannot be scheduled.
    * `Error` → pod failed to start or encountered runtime issues.

* **Initial Troubleshooting Step**
  * Check pod logs to identify the issue:
    ```bash
    kubectl -n <NAMESPACE> logs <POD NAME> -f
    ```

* **Common Fixes**
  * **Resource issues:**
    * Pod might be running out of CPU or memory.
    * Reduce **resource requests/limits** in the DaemonSet spec.
  * **Node pressure:**
    * Move some pods off affected nodes.
    * Use **taints and tolerations** to control pod placement.
  * **Cluster capacity:**
    * Scale up the cluster by adding more nodes.
  * **Other pod troubleshooting:**
    * Check events: `kubectl describe pod <POD NAME> -n <NAMESPACE>`
    * Ensure container images exist and are accessible.
    * Validate network connectivity if required by the pod.

* **Key Tip**
  * Since DaemonSets run **on all (or selected) nodes**, even a single node issue can make the DaemonSet appear unhealthy. Start troubleshooting **node by node**.


### **DaemonSet Scaling Concept**
  * **DaemonSets** automatically ensure **one Pod per node** (or per matching node).
  * You **cannot use** `kubectl scale` like you do with Deployments.
  * Scaling occurs **indirectly** by:
    * Adding nodes → **scales up** (creates new pods)
    * Removing nodes → **scales down** (deletes pods)
```bash
kubectl label node node01 type=demo
```
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: splunk-monitoring-agent
  labels:
    app: demo
spec:
  selector:
    matchLabels:
      app: demo
  template:
    metadata:
      labels:
        app: demo
    spec:
      nodeSelector:
        type: demo  # Ensures it runs on specific nodes with the label "platform-tools"
      containers:
        - name: splunk-monitoring-agent
          image: splunk:latest
          ports:
            - containerPort: 8088  # Assuming the agent exposes a port
          volumeMounts:
            - name: splunk-config
              mountPath: /etc/splunk  # Specify where the config should be mounted in the container
      volumes:
        - name: splunk-config
          configMap:
            name: splunk-config-map  # Ensure you have a ConfigMap named splunk-config-map
```

* **Manually Scaling a DaemonSet to Zero**
  * If you want to *temporarily remove all DaemonSet pods*, you can use a **non-matching node selector** to stop the DaemonSet from scheduling on any node.
  ```bash
  kubectl patch daemonset <daemonset-name> -n <namespace> \
    -p '{"spec": {"template": {"spec": {"nodeSelector": {"none": "match"}}}}}'
  ```
  ```bash
  kubectl patch daemonset splunk-monitoring-agent -n kube-system \
    -p '{"spec": {"template": {"spec": {"nodeSelector": {"dummy-nodeselector": "foobar"}}}}}'
  ```
  * This applies a fake label (`none=match`) that doesn’t exist on any node.
  * Result → All existing DaemonSet pods are **terminated**, and **no new pods** will be scheduled.

* **Scale Back to Normal**
  * To restore it, simply remove the dummy selector or reapply the original configuration:

* Option 1: Remove the selector completely
  ```bash
  kubectl patch daemonset splunk-monitoring-agent -n kube-system \
    -p '{"spec": {"template": {"spec": {"nodeSelector": null}}}}'
  ```

* Option 2: Restore the real selector
  ```bash
  kubectl patch daemonset splunk-monitoring-agent -n kube-system \
    -p '{"spec": {"template": {"spec": {"nodeSelector": {"type": "platform-tools"}}}}}'
  ```

* **Verification**
  * Check DaemonSet pods before and after:
  ```bash
  kubectl get pods -n kube-system -l app=logging -o wide
  ```
  * When scaled down → **No pods should appear**.
  * When restored → Pods will **reappear on all matching nodes**.

