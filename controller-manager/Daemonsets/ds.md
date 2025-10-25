
* A **DaemonSet** is a Kubernetes workload object that ensures a copy of a specific Pod runs on **all eligible (or some) nodes** in a cluster.
* It is commonly used for **cluster-wide services** that need to run on every node, such as logging agents, monitoring agents, or networking daemons.
* We use it when we need to run specific application on all nodes.
* Native K8S Object
* Can't not scale only run one pod on a node.
* If the daemonset pod gets deleted from the node, the daemonset controller creates it again.
* If there are 500 worker nodes and you deploy a daemonset, the daemonset controller will run one pod per worker node by default. That is a total of 500 pods. However, using nodeSelector, nodeAffinity, Taints, and Tolerations, you can restrict the daemonset to run on specific nodes.
* For example, in a cluster of 100 worker nodes, one might have 20 worker nodes labeled GPU enabled to run batch workloads. And you should run a pod on those 20 worker nodes. In this case, you can deploy the pod as a Daemonset using a node selector.

![](https://github.com/Mahin556/K8S-artifects/blob/main/images/image-4-44.png)

* Control plane component kube-proxy and CNI is run as daemonset.
* DaemonSets automatically add a Pod to new nodes and remove it from deleted nodes.
* Scheduling is done by the DaemonSet controller, not the default Kubernetes scheduler.
* Node addition/removal:
  - New nodes → Pod automatically added.
  - Removed nodes → Pod automatically deleted.
* Node labels updated → Pods are added/removed based on label matching.
* Updating a DaemonSet
  - You can modify the Pod template in the DaemonSet.
  - Limitations:
      - Some fields in existing Pods cannot be updated.
      - The DaemonSet controller applies the original template when new nodes are added.

* Deleting a DaemonSet:
  - `--cascade=orphan` → Leaves Pods running on nodes.
  - Re-creating a DaemonSet with the same selector can adopt existing Pods.

---

### Key Features of DaemonSet

* Ensures that a Pod is scheduled on every Node (or only on nodes that match node selectors, taints, tolerations, or affinity rules).
* Automatically deploys a Pod **when a new node is added** to the cluster.
* Removes the Pod from a node when the node is deleted from the cluster.
* If a DaemonSet is deleted, the Pods it created are automatically cleaned up.

---

### Common Use Cases

Why use DaemonSets?
Useful when you need a Pod running on every node. Common use cases:
  - Logging agents: Run on every node to collect logs for central logging system Eg: Splunk, Humio, Fluentd, logstash, fluentbit etc.
  - Monitoring agents: Deploy monitoring agents, such as Prometheus Node Exporter, on every node in the cluster to collect and expose node-level metrics. This way prometheus gets all the required worker node metrics.
  - Kubernetes system components: kube-proxy.
  - Network Management: Running a network plugin or firewall on every node to ensure consistent network policy enforcement. For example, Flannel, Calico, or other CNI plugins runs as a Daemonset on all the nodes.
  - Security and Compliance: Running CIS Benchmarks on every node using tools like kube-bench. Also deploy security agents, such as intrusion detection systems or vulnerability scanners, on specific nodes that require additional security measures. For example, nodes that handle PCI, and PII-compliant data.
  - Storage Provisioning: Running a storage plugin on every node to provide a shared storage system to the entire cluster.
Guarantees coverage across all nodes, which ReplicaSets cannot guarantee.

---

### How DaemonSet Works

* When you create a DaemonSet, Kubernetes scheduler ensures one Pod per node.
* Pods are evenly distributed — only one Pod per node unless configured otherwise.
* You can limit DaemonSet to specific nodes using:

  * **Node selectors**
  * **Node affinity/anti-affinity**
  * **Tolerations** for tainted nodes

---

### Example DaemonSet Manifest

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

---

### Managing DaemonSets

* **Create**: `kubectl apply -f daemonset.yaml`
* **List**: `kubectl get daemonset -A`
* **Describe**: `kubectl describe daemonset <name>`
* **Delete**: `kubectl delete daemonset <name>`

---

### Updating a DaemonSet

* By default, Pods are replaced **one at a time** (rolling update).
* Strategy can be controlled with `updateStrategy`:

  * **RollingUpdate** (default): Replace Pods gradually.
  * **OnDelete**: New Pods are created only after you manually delete the old ones.

Example:

```yaml
updateStrategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 1
```

---

### DaemonSet vs Deployment

* **Deployment**: Scales Pods arbitrarily across nodes, focuses on stateless apps.
* **DaemonSet**: Ensures **one Pod per node**, ideal for infrastructure-level services.
* **Deployment + HPA**: Used for scalable workloads (e.g., web apps).
* **DaemonSet**: Used for node-level background tasks.

### References
- https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/
- https://devopscube.com/kubernetes-daemonset/