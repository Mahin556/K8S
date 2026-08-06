### 🔹 Method 1: Edit the Existing Service

1. Run:

   ```bash
   kubectl edit svc my-app-lb
   ```
2. Change:

   ```yaml
   spec:
     type: LoadBalancer
   ```

   to:

   ```yaml
   spec:
     type: NodePort
   ```
3. Save and exit.
4. Kubernetes will automatically assign a **NodePort** (in range `30000–32767`).

Check it with:

```bash
kubectl get svc my-app-lb
```

Example output:

```
NAME        TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
my-app-lb   NodePort   10.96.50.200    <none>        80:30007/TCP   5m
```

Now, the app is accessible at:

```
http://<NodeIP>:30007
```

(Use `minikube ip` or `kubectl get nodes -o wide` to get Node IP).

---

### 🔹 Method 2: Patch the Service

Instead of editing, you can patch it in one command:

```bash
kubectl patch svc my-app-lb -p '{"spec": {"type": "NodePort"}}'
```

---

### 🔹 Method 3: Export & Reapply (Safe Way)

1. Export the YAML of the current Service:

   ```bash
   kubectl get svc my-app-lb -o yaml > svc-nodeport.yaml
   ```
2. Open `svc-nodeport.yaml` and modify:

   ```yaml
   spec:
     type: NodePort
   ```
3. Delete the old Service:

   ```bash
   kubectl delete svc my-app-lb
   ```
4. Apply the new one:

   ```bash
   kubectl apply -f svc-nodeport.yaml
   ```

---

### 🔹 Access After Conversion

* If you didn’t specify a `nodePort`, Kubernetes will pick a random port in the **30000–32767** range.
* You can **fix a custom NodePort** if needed:

  ```yaml
  ports:
    - port: 80
      targetPort: 8000
      nodePort: 30007   # must be within 30000-32767
  ```

---

✅ **In summary:**

* Change **`type: LoadBalancer` → `type: NodePort`** in the Service definition.
* Access your app using:

  ```
  http://<NodeIP>:<NodePort>
  ```
