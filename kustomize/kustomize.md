- https://devopscube.com/kustomize-tutorial/
- https://devopscube.com/kuztomize-configmap-generators/


Here’s a **complete and detailed guide on Kustomize** — including how it works, how to use `secretGenerator`, and how to manage deployments cleanly with customization layers.

---

### **What is Kustomize**

* **Kustomize** is a configuration management tool built into `kubectl`.
* It lets you **customize Kubernetes YAML manifests** without modifying the original files.
* It operates purely through YAML (no templating language like Helm).
* It’s ideal for managing multiple environments (dev, staging, prod) using a single base configuration.

---

### **Basic Concepts**

* **Base**: The original YAML manifests (Deployments, Services, etc.) stored in a directory.
* **Overlay**: A customized version of the base configuration (e.g., changes for different environments).
* **kustomization.yaml**: The main file that defines how to customize resources.

---

### **Installation**

* Kustomize is **already included** in `kubectl` (v1.14+).

  ```bash
  kubectl apply -k <directory>
  kubectl kustomize <directory>
  ```
* Or install standalone:

  ```bash
  curl -s "https://raw.githubusercontent.com/kubernetes-sigs/kustomize/master/install_kustomize.sh" | bash
  ```

---

### **Basic Example**

Suppose you have:

```
.
└── base/
    ├── deployment.yaml
    ├── service.yaml
    └── kustomization.yaml
```

`deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 2
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
        image: nginx:1.21
        ports:
        - containerPort: 80
```

`service.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
```

`kustomization.yaml`

```yaml
resources:
  - deployment.yaml
  - service.yaml
```

Apply:

```bash
kubectl apply -k base/
```

---

### **Customizing Resources**

You can modify the resources without changing their YAML directly.

#### **1. Change Image**

```yaml
images:
  - name: nginx
    newTag: 1.25
```

#### **2. Add Labels**

```yaml
commonLabels:
  env: dev
```

#### **3. Add Annotations**

```yaml
commonAnnotations:
  maintainer: "team@example.com"
```

#### **4. Patch Resources**

You can use **patches** to modify existing manifests.

`patch.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 5
```

Then reference it:

```yaml
patches:
  - path: patch.yaml
```

---

### **Secret and ConfigMap Generation**

#### **Secret Generator**

You can create secrets dynamically with **Kustomize**.

##### Option 1: From Files

`kustomization.yaml`

```yaml
secretGenerator:
  - name: db-credentials
    files:
      - username.txt
      - password.txt
```

Each file will become a key in the secret:

* `username.txt` → key `username`
* `password.txt` → key `password`

##### Option 2: From Literal Values

```yaml
secretGenerator:
  - name: db-credentials
    literals:
      - username=user
      - password=54f41d12e8fa
```

##### Apply:

```bash
kubectl apply -k .
```

##### Output example:

```bash
kubectl get secrets
kubectl describe secret db-credentials-9mfh2tcb24
```

Note: Kustomize **adds a hash suffix** (e.g., `db-credentials-9mfh2tcb24`) to ensure new secrets are created when data changes.

---

#### **ConfigMap Generator**

Same idea as secrets, but for non-sensitive data.

```yaml
configMapGenerator:
  - name: app-config
    literals:
      - log_level=debug
      - mode=dev
```

---

### **Overlays (For Different Environments)**

Directory structure:

```
.
├── base/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── kustomization.yaml
└── overlays/
    ├── dev/
    │   └── kustomization.yaml
    └── prod/
        └── kustomization.yaml
```

`overlays/dev/kustomization.yaml`

```yaml
resources:
  - ../../base

namePrefix: dev-
commonLabels:
  env: dev
images:
  - name: nginx
    newTag: 1.25
```

`overlays/prod/kustomization.yaml`

```yaml
resources:
  - ../../base

namePrefix: prod-
commonLabels:
  env: prod
images:
  - name: nginx
    newTag: 1.27
```

Apply for specific environment:

```bash
kubectl apply -k overlays/dev/
kubectl apply -k overlays/prod/
```

---

### **Other Useful Options**

#### **namespace**

```yaml
namespace: my-namespace
```

Applies the same namespace to all resources.

#### **replicas**

```yaml
replicas:
  - name: nginx-deployment
    count: 4
```

#### **transformers**

You can apply transformations such as label injection or prefix changes.

---

### **View the Final YAML**

Instead of applying directly:

```bash
kubectl kustomize overlays/dev/
```

This prints the fully rendered YAML to stdout (useful for debugging or CI/CD).

---

### **In Summary**

* **Base** = common manifests
* **Overlay** = environment-specific customization
* **secretGenerator/configMapGenerator** = dynamic secrets/configs
* **patches** = change specific fields
* **kubectl apply -k** = deploy everything

---

Would you like me to give you a **real working example directory** (with files and structure) that you can copy and test directly on your RHEL Kubernetes setup?
