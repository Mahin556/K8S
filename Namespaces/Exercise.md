## 🔹 Step 1: Create Two Namespaces

```yaml
# namespaces.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: dev

---
apiVersion: v1
kind: Namespace
metadata:
  name: prod
```

Apply:

```bash
kubectl apply -f namespaces.yaml
```

Check:

```bash
kubectl get ns
```

You should see:

```
NAME              STATUS   AGE
default           Active   5d
kube-system       Active   5d
dev               Active   1m
prod              Active   1m
```

---

## 🔹 Step 2: Deploy Pods in Each Namespace

### Pod in **dev** namespace

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: dev-pod
  namespace: dev
  labels:
    app: dev-app
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80
```

### Pod in **prod** namespace

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: prod-pod
  namespace: prod
  labels:
    app: prod-app
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80
```

Apply both:

```bash
kubectl apply -f dev-pod.yaml
kubectl apply -f prod-pod.yaml
```

Check:

```bash
kubectl get pods -n dev
kubectl get pods -n prod
```

---

## 🔹 Step 3: Expose Pods as Services

### Service in **dev** namespace

```yaml
apiVersion: v1
kind: Service
metadata:
  name: dev-svc
  namespace: dev
spec:
  selector:
    app: dev-app
  ports:
  - port: 80
    targetPort: 80
```

### Service in **prod** namespace

```yaml
apiVersion: v1
kind: Service
metadata:
  name: prod-svc
  namespace: prod
spec:
  selector:
    app: prod-app
  ports:
  - port: 80
    targetPort: 80
```

Apply both:

```bash
kubectl apply -f dev-svc.yaml
kubectl apply -f prod-svc.yaml
```

---

## 🔹 Step 4: Communication Between Pods

### Within the Same Namespace

* From `dev-pod`, you can curl `dev-svc`:

```bash
kubectl exec -it dev-pod -n dev -- curl dev-svc
```

* From `prod-pod`, you can curl `prod-svc`:

```bash
kubectl exec -it prod-pod -n prod -- curl prod-svc
```

### Across Namespaces

Use the **FQDN format**:

```
<service-name>.<namespace>.svc.cluster.local
```

* From `dev-pod` → access `prod-svc`:

```bash
kubectl exec -it dev-pod -n dev -- curl prod-svc.prod.svc.cluster.local
```

* From `prod-pod` → access `dev-svc`:

```bash
kubectl exec -it prod-pod -n prod -- curl dev-svc.dev.svc.cluster.local
```

---

## 🔹 Step 5: Verify Endpoints

Check services and endpoints:

```bash
kubectl get svc -n dev
kubectl get svc -n prod

kubectl get ep -n dev
kubectl get ep -n prod
```

You’ll see each Service points to its respective Pod IPs.

---

✅ **In summary:**

* Created **two namespaces**: `dev` and `prod`
* Deployed Pods + Services in each namespace
* Verified **intra-namespace** communication (short name works)
* Verified **cross-namespace** communication (FQDN 
