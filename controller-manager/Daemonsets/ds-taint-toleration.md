### 🔹 Step 1: Taint a Node

A taint is applied **to a node**. It marks the node with a key, value, and effect.

```bash
kubectl taint nodes k8s-worker-2 app=fluentd-logging:NoExecute
```

* **Key:** `app`
* **Value:** `fluentd-logging`
* **Effect:** `NoExecute` → This means pods without a matching toleration will be evicted and not scheduled here.

👉 After this, **only pods with a matching toleration** will be allowed on this node.

---

### 🔹 Step 2: Add Toleration in DaemonSet

Now, in your **DaemonSet YAML**, add the toleration section so that it can tolerate the taint.

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

* This allows **Fluentd pods** to still run on `k8s-worker-2` despite the taint.
* If you **remove this toleration**, the DaemonSet pod running on `k8s-worker-2` will be evicted (as you noticed).

---

### 🔹 Step 3: Verify the Behavior

1. Check the taints on the node:

   ```bash
   kubectl describe node k8s-worker-2 | grep Taints
   ```

   You should see:

   ```
   Taints: app=fluentd-logging:NoExecute
   ```

2. Apply the updated DaemonSet YAML:

   ```bash
   kubectl apply -f fluentd-daemonset.yaml
   ```

3. Watch the pods:

   ```bash
   kubectl get pods -o wide -n kube-system -l app=fluentd
   ```

   * Pods with toleration → will stay running on `k8s-worker-2`.
   * Pods without toleration → will get evicted and not scheduled there.

---

### 🔹 Effects in Detail

* **NoSchedule** → prevents *new* pods without toleration from scheduling on that node.
* **PreferNoSchedule** → soft preference: scheduler tries to avoid placing pods without toleration but may do so if needed.
* **NoExecute** → evicts *existing* pods without toleration **and** prevents new ones from scheduling.

---

### 🔹 Best Practices for DaemonSet with Taints/Tolerations

* Use **taints** to dedicate nodes (e.g., GPU nodes, logging nodes, monitoring nodes).
* Add **tolerations** in DaemonSet if the daemon should run on those special nodes.
* If you **don’t want** the DaemonSet to run there → simply **omit toleration**.

---

✅ So in your example:

* Taint applied → `k8s-worker-2` says: *“Only fluentd pods with correct toleration can run here.”*
* You updated DaemonSet → Fluentd tolerates the taint → it runs.
* Without toleration → pod on that node gets evicted.


### References:
- https://devopscube.com/kubernetes-daemonset/