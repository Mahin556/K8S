### References:-
- https://www.youtube.com/watch?v=uDlhbtGy1AU&ab_channel=CloudWithVarJosh
- https://youtu.be/yVLXIydlU_0
- https://spacelift.io/blog/kubernetes-namespaces
- https://www.tutorialspoint.com/kubernetes/kubernetes_namespace.htm
- https://www.geeksforgeeks.org/devops/kubernetes-namespaces/
- https://medium.com/@ravipatel.it/understanding-kubernetes-namespaces-5363bd8823dd
- https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/

---

## **What Are Namespaces in Kubernetes?**
A **namespace** in Kubernetes is a **logical partition** within a cluster that helps organize and isolate **resources**. Namespaces enable:  
- **Isolation & Security:** Separate or group workloads(pod,deployments etc) to prevent unwanted interactions(RBAC).  
- **Avoiding Naming Conflicts:** Resources with the **same name** can exist in **different namespaces**.  
- Can create in both ways(CLI,manifest)
- Avoid accidental deletion/modification
- **Resource Management:** Apply **resource quotas** and **limits** at the namespace level.  
- **Application Segregation:** Separate **environments** (e.g., **dev**, **test**, **prod**) or **projects**.  
- **Organizational Clarity:** Manage resources by **teams**, **departments**, or **projects**.  
- Virtual clusters over physical cluster.
- With in same namespace 2 resource of same type must have unique name.
- Two resource of same type in 2 diff namespace can have same name.
- Namespace-based scoping is applicable only for namespaced objects (e.g. Deployments, Services, etc.) 
- Cluster-wide objects (e.g. StorageClass, Nodes, PersistentVolumes, etc.).
- No nesting of namespaces.
- One resource only belong to only one namespace.
- User based access.
- In production not use `default` namespace user custom namespace.
- Avoid creating namespaces with the prefix kube-, since it is reserved for Kubernetes system namespaces.
---

## **Analogy to Understand Namespaces**  
Imagine a **large house** where **four families** live together:  

- Without namespaces, all families share **common spaces**, leading to **no privacy, security, or organization**.  
- When you **create rooms** for each family, each family has its **own space**, improving **isolation, security, and organization**.  

### **Relating to Kubernetes:**  
- The **large house** is the **Kubernetes cluster**.  
- The **families** are **applications** or **workloads**.  
- **Rooms** are **namespaces**, providing **segregation and control** over resources.  

---

## **Why Use Namespaces?**
* **Environment Separation**
  * Create separate namespaces for environments like **development**, **testing**, and **production**.
  * Prevents accidental modifications (e.g., deleting production resources while testing).
  * Can create a diff namespace for diff teams.

* **Access Control**
  * Define and regularly audit **RBAC policies** to enforce least-privilege access. 
  * Restrict users and service accounts to the Namespaces relevant to their roles.
  * You can define fine-grained permissions per namespace (e.g., developers have access to `dev` namespace but not `prod`).

* **Resource Management**
  * You can apply **resource quotas** and **limits** per namespace.
  * Helps ensure fair resource usage between teams or environments.

* **Security and Isolation**
  * Provides a boundary for workloads, reducing the risk of misconfigurations affecting unrelated applications, Limit access and apply **network policies**.

* **Multi-Tenancy:** 
  * Host **multiple applications** in the **same cluster** without conflicts.

* **Simplification:** Manage **related resources together**.  

* **Labels and Annotations:**
  Add **labels** and **annotations** to Namespaces to store metadata such as owner, environment, or project name.
  Example:

  ```bash
  kubectl label namespace dev-team env=development owner=team-a
  ```

  This makes it easier to automate management and reporting.

* **Rollback and Canary Deployments:**
  Namespaces can host different application versions for canary or rollback testing.
  Example:

  * Deploy `app-v1` in `prod`
  * Deploy `app-v2` in `prod-canary`
    If `app-v2` fails, simply delete the canary Namespace to revert quickly.

---

## **Default Namespaces in Kubernetes**  
When you run `kubectl get namespaces`, you’ll see these **default namespaces**:  

* **default**-> 
  * Kubernetes includes this namespace so that you can start using your new cluster without first creating a namespace. 

* **kube-system** -> 
  * The namespace for objects created by the Kubernetes system. Holds **Kubernetes control plane components** (e.g., **kube-dns**, **kube-proxy**). 
  * Contains control plane components (scheduler, controller-manager, DNS, Dashboard, Logging, Monitoring, etc.).
  * Kube-system Namespace is not meant for our (developer's) use. 
  * We do not have to create anything or modify anything in this namespace.

* **kube-public** -> 
  * This namespace is readable by all clients (including those not authenticated). **Publicly accessible data**, primarily used for **cluster information**.
  * Accessible to all users, contains publicly readable data, without authentication.

* **kube-node-lease** -> 
  * **Heartbeats of nodes** in the cluster, used by the **control plane** for **node health**. 
  * Tracks node heartbeats for availability.
  * So each Node basically gets its own lease object in the Namespace. This object contains the information about that nodes availability.

* **`local-path-store`** -> 
  * For storage related resource(based on setup) 

* Creating objects in the default namespace is considered a bad practice as it makes it harder to implement NetworkPolicies, use RBAC, and segregate objects.

**Note:** In **KIND** (**K**ubernetes **IN** **D**ocker) clusters, the local-path-storage namespace is created by default to support persistent storage using the Local Path Provisioner.

---

## **Communication Between Namespaces**
* **Within the same namespace**: Services and Pods communicate directly using their names.

* **Across namespaces**: You must use the **fully qualified domain name (FQDN)**:

  ```
  <service-name>.<namespace>.svc.cluster.local
  ```

  Example:

  ```
  backend-service.dev.svc.cluster.local
  ```

* Headless Service (Pod DNS records)
  * If service is headless (clusterIP: None), then each Pod behind it gets its own DNS entry: `<pod-name>.<service-name>.<namespace>.svc.cluster.local`
  ```bash
  kubectl get svc my-headless-svc -n dev -o yaml #---> db-0.my-headless-svc.dev.svc.cluster.local

  kubectl exec -it nginx -- cat /etc/resolve 
  ```

---

## **Working with Namespaces**

### **Creating Namespaces**

#### **1. Imperative Way:**
```sh
kubectl create namespace app1-ns
```

#### **2. Declarative Way:**
```yaml
# app1-ns.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: app1-ns

---
apiVersion: v1
kind: Namespace
metadata:
  labels:
    kubernetes.io/metadata.name: demo
  name: demo
```

```sh
kubectl apply -f app1-ns.yaml
```

---

### **Viewing and Deleting Namespaces**

#### **View All Namespaces:**
```sh
kubectl get namespaces

controlplane:~$ kubectl get ns
NAME                 STATUS   AGE
default              Active   10d
kube-node-lease      Active   10d
kube-public          Active   10d
kube-system          Active   10d
local-path-storage   Active   10d
```

#### **Delete a Namespace:**
```sh
kubectl delete namespace app1-ns
```
**Warning:** Deleting a namespace **removes all resources within it**, including **pods**, **services**, **configmaps**, **secrets etc**..

---

## **Rename a Namespace in Kubernetes (Workaround)**
Kubernetes does not allow directly renaming a namespace.
However, you can effectively “rename” it by exporting and recreating it under a new name.
```bash
kubectl get namespace my-namespace-declarative -o yaml > new_namespace.yaml
```
Open new_namespace.yaml in a text editor.

Modify the metadata.name field with your new namespace name.

(Example: change my-namespace-declarative → my-namespace-declarative-modified)

```bash
kubectl apply -f new_namespace.yaml
namespace/my-namespace-declarative-modified created
```
---

## **Migrate resources to the new namespace**
Export each resource’s YAML (e.g., deployments, services, secrets) from the old namespace.
Example:
```bash
kubectl get all -n my-namespace-declarative -o yaml > resources.yaml
```
Edit each resource YAML and change the namespace: field to the new namespace name.
```bash
kubectl apply -f resources.yaml
```
Delete the old namespace
```bash
kubectl delete namespace my-namespace-declarative
namespace "my-namespace-declarative" deleted
```
---

## **Set the Default Namespace for kubectl**
By default, kubectl uses the default namespace unless overridden with -n or --namespace.

You can change the default namespace for all subsequent commands by updating your current kubeconfig context:
```bash
kubectl config set-context --current --namespace=<your-namespace>
kubectl config set-context --current --namespace=dev
# Validate it
kubectl config view --minify | grep namespace:
kubectl get pods 
kubectl config set-context --current --namespace=default
```
---

## **Using namespace**
```yaml
apiVersion: v1
kind: Service
metadata:
   name: elasticsearch
   namespace: elk
   labels:
      component: elasticsearch
spec:
   type: LoadBalancer
   selector:
      component: elasticsearch
   ports:
   - name: http
      port: 9200
      protocol: TCP
   - name: transport
      port: 9300
      protocol: TCP
```

* Configmap create in default namespace
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
    name: my-configmap
data:
    db_url: my-service.database
```

---

### **Using Namespace Flags**

| **Flag** | **Description** |
|----------|-----------------|
| `-n <namespace-name>` or `--namespace` | Execute commands within a **specific namespace**. |
| `-A` or `--all-namespaces` | Execute commands across **all namespaces**. |

```sh
kubectl get pods -n app1-ns
kubectl get services -A
kubectl api-resources --namespaced=flase 
```

---

## **Deploying Frontend and Backend in a Namespace**

- We'll use **existing YAML files** for our **frontend** and **backend deployments**.
- Deploy the **frontend** and **backend** components in the **app1-ns** namespace.

---

### **1. Applying Frontend and Backend YAMLs**

#### **1.1 Frontend YAML (`frontend-deploy.yaml`)**
```sh
kubectl apply -f frontend-deploy.yaml -n app1-ns
```

#### **1.2 Backend YAML (`backend-deploy.yaml`)**
```sh
kubectl apply -f backend-deploy.yaml -n app1-ns
```

---

### **2. Verify the Deployments in `app1-ns`**

```sh
kubectl get all -n app1-ns
```

---

## **Testing Namespace Isolation**

### **1. Test Pod in `default` Namespace**
```sh
kubectl run test-pod --image=busybox -it --rm --restart=Never -- /bin/sh
```

```sh
curl backend-svc:9090 # ❌ Will not work
```
**For cross-namespace access, use the following format:**  
```sh
curl http://backend-svc.app1-ns:9090
```

### ✅ **Format:**  
```sh
curl http://<service-name>.<namespace-name>:<service-port>
```

- **`<service-name>`**: Name of the **Kubernetes Service**, e.g., **`backend-svc`**.  
- **`<namespace-name>`**: Namespace where the **service** is deployed, e.g., **`app1-ns`**.  
- **`<service-port>`**: The **ClusterIP port** exposed by the **service**, e.g., **`9090`**.  


### 💡 **Coming Up:**  
In our **DNS resolution lecture**, we'll explore **why this format is needed** and understand **how Kubernetes handles service discovery** within and across **namespaces**.

---

### **2. Test Pod in `app1-ns` Namespace**
```sh
kubectl run test-pod -n app1-ns --image=busybox -it --rm --restart=Never -- /bin/sh
```

```sh
curl backend-svc:9090 # ✅ Should work
```

---

## **Setting a Default Namespace in Kubernetes Context**

### **Why?**  
It’s **cumbersome** to use `-n <namespace>` with **every command**. You can set a **default namespace** in the **Kubernetes context**.

```sh
kubectl config set-context --current --namespace=app1-ns
```

- Now, you **don't need** to use `-n app1-ns` with **every command**.  
- To **check the current context**, run:  

```sh
kubectl config get-contexts
```
---

## **Warning: Namespace Names Conflicting with Public DNS**

* **Issue:**
  Creating a Kubernetes Namespace with the same name as a **public top-level domain (TLD)** (e.g., `com`, `org`, `net`) can cause **DNS conflicts**.

  * Services in these Namespaces get **short DNS names** that can overlap with public DNS records.
  * Workloads performing DNS lookups **without a trailing dot (`.`)** may resolve to internal services in these Namespaces **instead of public internet addresses**.
  * This can unintentionally redirect traffic meant for external domains to internal services.

* **Example:**
  If a Namespace is named `com` and you create a Service `web`, then:

  ```
  web.com
  ```

  may resolve internally to your Kubernetes Service instead of the public `web.com`.

---

## **Kubernetes Namespace Naming Rules**
* The **maximum length** of a namespace name is **63 characters**.
* Only **lowercase alphanumeric characters (`a–z`, `0–9`)** and **hyphens (`-`)** are allowed.
* The name **must start and end with an alphanumeric character**.
* The name **cannot start or end with a hyphen (`-`)**.
* Names must conform to the **DNS-1123 label standard**, which applies to many Kubernetes resource names (like Pods, Services, and Deployments).

* **Examples (Valid Names)**
* `default`
* `dev-environment`
* `project123`
* `k8s-prod-01`

* **Examples (Invalid Names)**
* `-production` → starts with a hyphen
* `Production` → contains uppercase letters
* `dev_env` → contains an underscore
* `test-` → ends with a hyphen
* `namespace@1` → contains invalid character `@`

---

## **Best Practices for Using Namespaces**

- **Segregate Workloads:** Separate **dev**, **test**, and **prod environments**.  
- **Use Namespaces for Multi-Tenancy:** Avoid **resource conflicts** by **isolating teams or projects**.  
- **Resource Quotas:** Set **limits** on **CPU**, **memory**, and **storage** per namespace.  
- **Namespace Naming Conventions:** Use **clear** and **consistent names** (e.g., `team-app-env` → `frontend-prod-ns`).  
- **Avoid Manual Deletion:** Use `kubectl delete namespace` **carefully**, as it **removes all resources within**.  
- **Apply Network Policies:** Secure namespaces using **network policies** to control **traffic flow** between pods and services, improving security and isolation..  
- Avoid using the kube- prefix for custom namespaces since it’s reserved for Kubernetes system namespaces.
- Use RBAC (Role-Based Access Control) to enforce least-privilege access; bind roles to specific namespaces only when necessary.
* Integrate monitoring and logging solutions (like Prometheus, Grafana, or ELK) for namespace-specific visibility and alerts.
* Use multiple namespaces to separate workloads by environment (dev/test/prod), team, or application to maintain scalability, governance, and compliance.

---

### Task details
- Create two namespaces and name them ns1 and ns2
- Create a deployment with a single replica in each of these namespaces with the image as nginx and name as deploy-ns1 and deploy-ns2, respectively
- Get the IP address of each of the pods (Remember the kubectl command for that?)
- Exec into the pod of deploy-ns1 and try to curl the IP address of the pod running on deploy-ns2
- Your pod-to-pod connection should work, and you should be able to get a successful response back.
- Now scale both of your deployments from 1 to 3 replicas.
- Create two services to expose both of your deployments and name them svc-ns1 and svc-ns2
- exec into each pod and try to curl the IP address of the service running on the other namespace.
- This curl should work.
- Now try curling the service name instead of IP. You will notice that you are getting an error and cannot resolve the host.
- Now use the FQDN of the service and try to curl again, this should work.
- In the end, delete both the namespaces, which should delete the services and deployments underneath them.

---

## **Mitigation Strategies**
1. **Restrict Namespace Creation:**
   * Limit privileges to create Namespaces only to **trusted users or administrators**.
   * Use **RBAC** to prevent unprivileged users from creating arbitrary Namespaces.

2. **Admission Webhooks:**
   * Configure **admission controllers** or third-party webhooks to **block Namespace names matching public TLDs**.
   * Example: Block Namespaces like `com`, `org`, `net`, `io`, etc.

3. **Namespace Naming Conventions:**
   * Enforce **prefixes or suffixes** for internal Namespaces (e.g., `dev-`, `team-`, `internal-`).
   * Avoid short, generic names that may conflict with public domains.

4. **DNS Hygiene:**
   * Always use **fully qualified domain names (FQDNs)** in application DNS lookups when querying external services.
   * Example: Use `example.com.` with trailing dot to ensure lookup goes to public DNS.


---

## **Summary**

- **Namespaces** provide **logical isolation** in Kubernetes.  
- They help with **security**, **resource management**, and **multi-tenancy**.  
- Use **imperative** and **declarative methods** to **create and manage namespaces**.  
- Be **careful when deleting namespaces**, as it **removes all resources within**.  
- **Set default namespaces** in your **Kubernetes context** for **ease of use**.  
- Follow **best practices** to **organize resources effectively** and **ensure security**.  

---

## References  
- [Kubernetes Namespaces Documentation](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/)
