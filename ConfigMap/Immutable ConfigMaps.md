## **Using Immutable ConfigMaps**
* Default ConfigMaps are mutable: you can change the data in place.
* Readonly
* Can't modify
* make sure consistency and no alteration.
* Recreate to change value.
* Attempts to modify it will fail.
* Prevents accidental changes that could destabilize your application.

### 🗂️ Step 1 — Create an Immutable ConfigMap

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: demo-config-immutable
data:
  foo: bar
immutable: true
```
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: test-config
data:
  config.properties: |
    database_url=http://example.com/db
    debug_mode=true
    log_level=debug
immutable: true   # ✅ Correct placement of the immutable flag
```

Apply it:

```bash
kubectl apply -f demo-config-immutable.yaml
```

✅ Output:

```
configmap/demo-config-immutable created
```

---

### 🧱 Step 2 — Try to Modify It

If you edit and try to change:

```yaml
data:
  foo: baz
```

Then reapply:

```bash
kubectl apply -f demo-config-immutable.yaml
```

You’ll get:

```
The ConfigMap "demo-config-immutable" is invalid: data: Forbidden: field is immutable when `immutable` is set
```

Kubernetes **blocks all modifications** to immutable ConfigMaps.

---

### 🧹 Step 3 — How to “update” an immutable ConfigMap

You **can’t modify** it directly.
If you need a change, you must **delete and recreate** it with a new name or the same name:

```bash
kubectl delete configmap demo-config-immutable
kubectl apply -f demo-config-immutable.yaml
```

---

## 🧠 **Why make ConfigMaps immutable?**

| Benefit               | Description                                                              |
| --------------------- | ------------------------------------------------------------------------ |
| 🧱 **Stability**      | Prevents accidental or unauthorized edits that could break running apps. |
| ⚡ **Performance**     | Nodes skip checking for updates from the API server.                     |
| 🧩 **Predictability** | Ensures app configuration remains constant throughout its lifecycle.     |
| 🔒 **Safety**         | Ideal for production environments with strict config control.            |

---

## 🧰 **Understanding ConfigMap updates (mutable vs immutable)**

| Consumption Method                           | Does change apply live? | Notes                                         |
| -------------------------------------------- | ----------------------- | --------------------------------------------- |
| **Environment Variables** (`env`, `envFrom`) | ❌ No                    | Pods must restart to pick up changes.         |
| **Command Line Arguments**                   | ❌ No                    | Restart required.                             |
| **Mounted Volumes**                          | ✅ Yes (delayed)         | Kubelet periodically refreshes mounted files. |
| **Immutable ConfigMap**                      | ❌ Never                 | Must delete/recreate to update.               |

---

### 🧪 Example — Using Immutable ConfigMap in a Pod

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: demo-config-immutable
data:
  foo: bar
immutable: true

---

apiVersion: v1
kind: Pod
metadata:
  name: immutable-demo-pod
spec:
  containers:
    - name: app
      image: busybox
      #command: ["sh", "-c", "echo Config: $(cat /config/foo) && sleep 3600"] #Behave unexpected if there is delay on mounting
      #command: ["sh", "-c", "sleep 5; echo Config: $(cat /config/foo); sleep 3600"] #Delay until the volume is mounted
      command: ["sh", "-c", "while true; do echo Config: $(cat /config/foo); sleep 10; done"] #Run a simple command that runs after mounting
      volumeMounts:
        - name: config
          mountPath: /config
  volumes:
    - name: config
      configMap:
        name: demo-config-immutable
```

When you run:

```bash
kubectl apply -f immutable-demo-pod.yaml
kubectl logs immutable-demo-pod
```

✅ Output:

```
Config: bar
```

If you later try to change the ConfigMap, Kubernetes blocks it — ensuring your Pod’s configuration remains frozen.

---

## ⚙️ **Pro Tip: Versioned ConfigMaps**

In production, a **best practice** is to use **versioned ConfigMaps** like this:

```
demo-config-v1
demo-config-v2
```

Then update your Pods or Deployments to reference the new one:

```yaml
env:
  - name: APP_NAME
    valueFrom:
      configMapKeyRef:
        name: test-config
        key: app_name
```

```bash
kubectl create configmap demo-config-v2 --from-literal=app_name=demo-v2

kubectl set env deployment/myapp --from=configmap/demo-config-v2

kubectl rollout restart deployment my-app
```

That allows controlled, predictable updates — without mutating the existing ConfigMap.

### References:
- https://spacelift.io/blog/kubernetes-configmap
- https://www.geeksforgeeks.org/devops/kubernetes-configmap/
- https://www.groundcover.com/blog/kubernetes-configmap