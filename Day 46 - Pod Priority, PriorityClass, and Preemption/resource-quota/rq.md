### References:
- https://kubernetes.io/docs/concepts/policy/resource-quotas/ *
- https://kubernetes.io/docs/tasks/administer-cluster/manage-resources/quota-memory-cpu-namespace/ *

---

# **ResourceQuota in Kubernetes**

* A ResourceQuota in Kubernetes is a way to limit total resource usage per namespace.
* It ensures fair usage and prevents a single team/app from overusing cluster resources.
* Admins define it so that CPU, memory, and object counts (like pods, PVCs, or ConfigMaps) are controlled.
* Prevents **noisy neighbor** issues where one namespace consumes all CPU/memory.
* Very important component to maintain the stablity with in cluster.
* Helps divide resources fairly across teams/environments (dev, staging, prod).
* Works with LimitRange to control per-container defaults and limits.
* Enforces cluster stability by restricting excessive object creation.
* ResourceQuota is namespace-scoped.
* Can limit the number of resources such as Pods, CPU, PVCs, Services, and memory that can be consumed within a namespace.
* When users create or update a resource:
  * Kubernetes API checks if doing so would exceed the quota.
  * If yes → request is rejected with an error.
* We can monitor or track the resource utilization based on namespace.
* K8s have special api object for it `ResourceQuota`.
* If quotas are enabled in a namespace for resource such as `cpu` and `memory`, users must specify requests or limits for those values when they define a Pod; otherwise, the quota system may reject pod creation.
It can limit:

1. **Object count quotas** (e.g., number of Pods, Services, ConfigMaps).
2. **Compute resource quotas** (e.g., CPU, memory requests/limits).

---

## **Example**
```bash
kubectl create namespace dev-team
```
```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: dev-team-quota
  namespace: dev-team
spec:
  hard:
    pods: "10"                       # max 10 pods
    requests.cpu: "4"                # total CPU requests across all pods
    requests.memory: "8Gi"           # total memory requests across all pods
    limits.cpu: "6"                  # total CPU limits
    limits.memory: "10Gi"            # total memory limits
    persistentvolumeclaims: "5"      # max number of PVCs
    services: "5"                    # max number of Services
    configmaps: "10"                 # max ConfigMaps
    secrets: "10"                    # max Secrets
```
```bash
kubectl apply -f resource-quota.yaml
```
---

## **Explanation**

* **pods: 10**
  Max 10 Pods can exist in this namespace.

* **requests.cpu: 5**
  All Pods combined can request only **4 CPU** (4000 millicores).

* **requests.memory: 8Gi**
  Total memory requests for all Pods ≤ **8GiB**.

* **limits.cpu: 6**
  Total CPU limit for all Pods ≤ **6 CPU core**.

* **limits.memory: 10Gi**
  Total memory limit for all Pods ≤ **10 GiB**.

---

## **Key Notes**

* You can define **only object quotas**, **only compute quotas**, or **both**.
* ResourceQuota applies **per namespace**, not cluster-wide.
* ConfigMap, Secrets, PersistentVolumeClaims can also be restricted.
* The variable names (like `requests.cpu`, `limits.memory`) **must match Kubernetes’ expected keys**.

---

## **Other Examples**

### 1. **Limit ConfigMaps and PVCs**

```yaml
spec:
  hard:
    configmaps: "20"
    persistentvolumeclaims: "5"
```

### 2. **Restrict LoadBalancers**

```yaml
spec:
  hard:
    services.loadbalancers: "2"
```

### 3. **Restrict Secrets**

```yaml
spec:
  hard:
    secrets: "50"
```

* **Verify the Quota**
```bash
$ kubectl get resourcequota -n dev-team

NAME                     AGE   REQUEST                                                LIMIT
resource-quota-example   11m   pods: 0/1, requests.cpu: 0/2, requests.memory: 0/5Gi   limits.cpu: 0/4, limits.memory: 0/10Gi


$ kubectl describe resourcequota dev-team-quota -n dev-team
```
```bash
$ kubectl describe ns example-namespace

Name:         example-namespace
Labels:       kubernetes.io/metadata.name=example-namespace
Annotations:  <none>
Status:       Active

Resource Quotas
  Name:            resource-quota-example
  Resource         Used  Hard
  --------         ---   ---
  limits.cpu       0     4
  limits.memory    0     10Gi
  pods             0     1
  requests.cpu     0     2
  requests.memory  0     5Gi

No LimitRange resource.
```

* **Test the Quota Enforcement**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pod
  namespace: dev-team
spec:
  containers:
  - name: nginx
    image: nginx
    resources:
      requests:
        cpu: "500m"
        memory: "256Mi"
      limits:
        cpu: "1"
        memory: "512Mi"
```
```bash
for i in {1..12}; do
  kubectl apply -f pod.yaml --record=false --namespace=dev-team --filename=- <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: test-pod-$i
  namespace: dev-team
spec:
  containers:
  - name: nginx
    image: nginx
    resources:
      requests:
        cpu: "500m"
        memory: "256Mi"
      limits:
        cpu: "1"
        memory: "512Mi"
EOF
done
```
The first 10 pods will be created, but from the 11th pod onward, you’ll see:
```bash
Error from server (Forbidden): exceeded quota: dev-team-quota, requested: pods=1, used: 10, limited: 10
```

* **Check Current Usage**
```bash
kubectl get resourcequota dev-team-quota -n dev-team -o yaml
```
```yaml
status:
  hard:
    pods: "10"
    requests.cpu: "4"
  used:
    pods: "10"
    requests.cpu: "3.5"
```

* **Get resource quota of all namespaces**
```bash
kubectl describe resourcequota -A
```

* **Real-world Example (Team-wise Separation)**
```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: backend-quota
  namespace: backend
spec:
  hard:
    pods: "20"
    requests.cpu: "10"
    requests.memory: "20Gi"
    limits.cpu: "15"
    limits.memory: "25Gi"
---
apiVersion: v1
kind: ResourceQuota
metadata:
  name: frontend-quota
  namespace: frontend
spec:
  hard:
    pods: "15"
    requests.cpu: "6"
    requests.memory: "12Gi"
    limits.cpu: "8"
    limits.memory: "15Gi"
```

* **Combine with LimitRange**
* ResourceQuota sets namespace-wide resource caps.
* LimitRange sets per-container default/request/limit values.
```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: dev-limits
  namespace: dev-team
spec:
  limits:
  - default:
      cpu: "500m"
      memory: "512Mi"
    defaultRequest:
      cpu: "250m"
      memory: "256Mi"
    type: Container
```
Apply both for complete control:
```bash
kubectl apply -f limit-range.yaml
```

---

#### **Different Quota Types**

* **Compute Resource Quotas**
  * Controls total CPU/memory usage.
  * Keys:
    * `requests.cpu`, `requests.memory`, `limits.cpu`, `limits.memory`.

* **Storage Quotas**
  * Controls PVC count and total storage.
  * Keys:
    * `requests.storage`, `persistentvolumeclaims`.
  Example:
  ```yaml
  requests.storage: "50Gi"
  persistentvolumeclaims: "10"
  ```

* **Object Count Quotas**
  * Limits number of specific resource types.
  * Keys:
    * `pods`, `services`, `configmaps`, `secrets`, `replicationcontrollers`, etc.

* **Scope-based Quotas**
  * Quotas can apply to subsets using **scopes**.
  Example: apply only to pods not terminating
  ```yaml
  spec:
    scopes:
    - NotTerminating
  ```
  Other scopes:
  * `Terminating`
  * `NotBestEffort`
  * `BestEffort`
  * `CrossNamespacePodAffinity`

