### References:- 
- https://medium.com/@muppedaanvesh/a-hand-on-guide-to-kubernetes-resource-quotas-limit-ranges-%EF%B8%8F-8b9f8cc770c5

---

* A LimitRange in Kubernetes defines default, minimum, and maximum resource requests and limits for pods, containers and PersistentVolumeClaims (PVCs) within a namespace.
* It like a policy.
* By using Limit Ranges, you can control resource usage for Pods, containers, and PVCs, ensuring efficient and fair resource distribution across your cluster.
* Unlike a ResourceQuota (which applies to the entire namespace), LimitRange applies per Pod or Container.
* Help to prevent **noise neighbour** issues.
* Namespaces:
  ```bash
  controlplane:~$ kubectl api-resources --namespaced=true | grep -i limitrange
  limitranges                 limits       v1                             true         LimitRange
  ```
* Request Limit ---> Namespace, Prevent namespace to use resources for other namespaces.
* LimitRange ---> objects(pod,container,PVCs), Prevent object to use resources for other object.

* Important points:
    * Prevent users from creating pods without specifying resource requests/limits.
    * Enforce upper and lower bounds on CPU and memory usage per container.
    * Automatically assign default requests and limits if users don’t specify them.
    * Work together with ResourceQuota to achieve fair and controlled resource allocation.
    * Namespace-scoped: each namespace can have its own LimitRange.

* When you create a Pod or Container:
    * Kubernetes checks if it violates the LimitRange.
    * If it does, the API server rejects the request.
    * If resource values are missing and defaults exist, defaults are automatically applied.

* Create a Namespace:
```bash
kubectl create namespace dev-team
```
```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: dev-limits
  namespace: dev-team
spec:
  limits:
  - type: Container
    max:
      cpu: "1"               # Max 1 vCPU per container
      memory: "1Gi"          # Max 1Gi memory per container
    min:
      cpu: "100m"            # Minimum 0.1 vCPU
      memory: "128Mi"        # Minimum 128Mi memory
    default:
      cpu: "500m"            # Default limit (if not specified)
      memory: "512Mi"
    defaultRequest:
      cpu: "250m"            # Default request (if not specified)
      memory: "256Mi"
```
```bash
kubectl apply -f limit-range.yaml
```
* Verify the LimitRange
```bash
kubectl get limitrange -n dev-team
kubectl describe limitrange dev-limits -n dev-team

Name:       dev-limits
Namespace:  dev-team
Type        Resource  Min     Max     Default Request  Default Limit
----        --------  ---     ---     ---------------  -------------
Container   cpu       100m    1       250m             500m
Container   memory    128Mi   1Gi     256Mi            512Mi
```

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: limit-range-example
  namespace: example-namespace
  labels:
    environment: dev
    owner: admin-team
  annotations:
    description: "Limit range for CPU, memory, and storage control in example-namespace"
spec:
  limits:
  # ------------------------ POD LEVEL ------------------------
  - type: Pod
    max:
      cpu: "2"               # Maximum total CPU per Pod (sum of all containers)
      memory: "4Gi"          # Maximum total memory per Pod
      ephemeral-storage: "2Gi"  # Optional - maximum total ephemeral storage per Pod
    min:
      cpu: "200m"            # Minimum total CPU per Pod
      memory: "256Mi"        # Minimum total memory per Pod
      ephemeral-storage: "500Mi"  # Optional - minimum ephemeral storage per Pod
    maxLimitRequestRatio:
      cpu: "4"               # Max allowed ratio between limit and request (4x)
      memory: "8"            # Memory limit cannot exceed 8x of request

  # --------------------- CONTAINER LEVEL ----------------------
  - type: Container
    default:
      cpu: "500m"            # Default CPU limit if not specified in container spec
      memory: "512Mi"        # Default memory limit if not specified
      ephemeral-storage: "1Gi" # Optional - default ephemeral storage limit
    defaultRequest:
      cpu: "250m"            # Default CPU request
      memory: "256Mi"        # Default memory request
      ephemeral-storage: "512Mi" # Optional - default ephemeral storage request
    max:
      cpu: "1"               # Maximum CPU a single container can request
      memory: "1Gi"          # Maximum memory for a single container
      ephemeral-storage: "2Gi"
    min:
      cpu: "100m"            # Minimum CPU per container
      memory: "128Mi"        # Minimum memory per container
      ephemeral-storage: "256Mi"
    maxLimitRequestRatio:
      cpu: "2"               # Limit cannot exceed 2x the request
      memory: "4"            # Limit cannot exceed 4x the request
      ephemeral-storage: "3" # Optional - storage limit ratio

  # ------------------ PERSISTENT VOLUME CLAIM -----------------
  - type: PersistentVolumeClaim
    max:
      storage: "10Gi"        # Maximum storage claim allowed
    min:
      storage: "1Gi"         # Minimum PVC size
    default:
      storage: "5Gi"         # Default limit size if not specified
    defaultRequest:
      storage: "2Gi"         # Default requested size if not defined
    maxLimitRequestRatio:
      storage: "2"           # PVC limit can be 2x the requested size
```
* **spec.limits[] — Object Fields**

  | Field                    | Description                                                                                                 |
  | ------------------------ | ----------------------------------------------------------------------------------------------------------- |
  | **type**                 | Defines the resource type the rule applies to. Options: `Container`, `Pod`, or `PersistentVolumeClaim`.     |
  | **max**                  | Specifies the *maximum* allowed resource usage for the given type.                                          |
  | **min**                  | Specifies the *minimum* allowed resource usage.                                                             |
  | **default**              | Defines *default limit values* if a container or PVC doesn’t specify them.                                  |
  | **defaultRequest**       | Defines *default request values* if none are specified.                                                     |
  | **maxLimitRequestRatio** | Defines how much greater a resource’s limit can be compared to its request. Prevents huge overprovisioning. |

* **Each of the above fields can include these resource types:**

  | Resource            | Unit               | Description                                         |
  | ------------------- | ------------------ | --------------------------------------------------- |
  | `cpu`               | millicores (`m`)   | 1000m = 1 vCPU                                      |
  | `memory`            | bytes (`Mi`, `Gi`) | Memory allocation for Pods/Containers               |
  | `ephemeral-storage` | bytes (`Mi`, `Gi`) | Temporary disk storage for container filesystem     |
  | `storage`           | bytes (`Mi`, `Gi`) | Used in PVCs for persistent storage                 |
  | `hugepages-<size>`  | bytes              | For workloads requiring huge page memory (advanced) |


```bash
$ kubectl apply -f limit-range.yaml
```
```bash
$ kubectl get limitrange -n example-namespace
$ kubectl describe limitrange limit-range-example -n example-namespace
$ kubectl describe namespace example-namespace

Resource Limits
 Type                   Resource           Min     Max    Default Request  Default Limit  Max Limit/Request Ratio
 ----                   --------           ---     ---    ---------------  -------------  -----------------------
 Pod                    cpu                200m    2      -                -              4
 Pod                    memory             256Mi   4Gi    -                -              8
 Container              cpu                100m    1      250m             500m           2
 Container              memory             128Mi   1Gi    256Mi            512Mi          4
 Container              ephemeral-storage  256Mi   2Gi    512Mi            1Gi            3
 PersistentVolumeClaim  storage            1Gi     10Gi   2Gi              5Gi            2

```
This confirms that the LimitRange is active for the namespace.

Now, test whether the defaults are applied when no resource values are defined.
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: limit-test
  namespace: example-namespace
spec:
  replicas: 1
  selector:
    matchLabels:
      app: test
  template:
    metadata:
      labels:
        app: test
    spec:
      containers:
      - name: nginx
        image: nginx:latest
```
```bash
kubectl apply -f deploy.yaml
```
```bash
kubectl describe pod <pod-name> -n example-namespace
```
```yaml
Requests:
  cpu: 250m
  memory: 256Mi
  ephemeral-storage: 512Mi
Limits:
  cpu: 500m
  memory: 512Mi
  ephemeral-storage: 1Gi
```
This confirms that the LimitRange default values are successfully applied.

* **Common Validation Errors**

| Error Message                                      | Meaning                                   | Solution                             |
| -------------------------------------------------- | ----------------------------------------- | ------------------------------------ |
| `requested memory is less than min`                | Pod request below defined minimum         | Increase resource request            |
| `requested cpu exceeds max`                        | Pod requested CPU higher than allowed     | Reduce CPU limit                     |
| `limit/request ratio exceeds maxLimitRequestRatio` | Limit is set too high compared to request | Lower the limit or raise the request |
| `PVC request exceeds max storage`                  | PVC requested storage larger than `max`   | Reduce storage request               |
