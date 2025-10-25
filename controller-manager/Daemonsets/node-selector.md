* **DaemonSet basics**
  Normally, a DaemonSet schedules one Pod per node across the entire cluster. But sometimes, you don’t want it on *every node*, only on a subset of nodes (e.g., logging agents only on worker nodes). That’s where **nodeSelector** (or affinities/taints) comes in.

* **Step 1: Label the Node(s)**
  First, you add a label to the specific nodes where you want your DaemonSet Pods to run:

  ```bash
  kubectl label node k8s-worker-1 type=platform-tools
  kubectl label node k8s-worker-2 type=platform-tools
  ```

  Now these nodes have the label `type=platform-tools`.

* **Step 2: Update DaemonSet spec with nodeSelector**
  Inside the DaemonSet Pod spec, you add:

  ```yaml
  spec:
    nodeSelector:
      type: platform-tools
  ```

  This tells Kubernetes:
  👉 “Only schedule this DaemonSet Pod on nodes that have the label `type=platform-tools`.”

* **Step 3: DaemonSet Behavior**

  * The controller looks at all nodes.
  * Only nodes that match the selector get a Pod.
  * Nodes without that label are skipped (DaemonSet won’t schedule Pods there).
  * If you add a new node with that label, DaemonSet will automatically create a Pod on it.

* **Your YAML Screenshot**
  In your DaemonSet:

  ```yaml
  nodeSelector:
    type: platform-tools
  ```

  ensures `fluentd` runs **only** on nodes you explicitly marked as `type=platform-tools`.
  Without this, it would run everywhere.

* **Use case example**

  * Run logging agents (fluentd) only on worker nodes.
  * Run monitoring agents (Prometheus node exporter) only on Linux nodes (not Windows).
  * Run GPU drivers only on nodes with GPUs.

### References:
- https://devopscube.com/kubernetes-daemonset/