### References
- https://kubernetes.io/docs/reference/kubectl/generated/kubectl_label/

---

### Labels
- Labels are key-value pairs attached to Kubernetes objects like pods, services, and deployments. 
- They used to organize, filter, and select resources to target them.

**Examples of Labels:**
- `environment: production`
- `type: backend`
- `tier: frontend`
- `application: my-app`

```
kubectl label [--overwrite] (-f FILENAME | TYPE NAME) KEY_1=VAL_1 ... KEY_N=VAL_N [--resource-version=version]
```

- View labels
  ```bash
  kubectl get pod --show-labels
  kubectl get nodes --show-labels
  kubectl get pod nginx --show-labels
  ```

- Update pod 'foo' with the label 'unhealthy' and the value 'true'
  ```bash
  kubectl label pods foo unhealthy=true
  kubectl label pod nginx app=web
  kubectl label deployment myapp tier=frontend
  kubectl label node worker1 env=prod
  kubectl label namespace dev team=backend
  ```

- Update pod 'foo' with the label 'status' and the value 'unhealthy', overwriting any existing value
  ```bash
  kubectl label --overwrite pods foo status=unhealthy
  kubectl label pod nginx app=web --overwrite
  ```  

- Update all pods in the namespace
  ```bash
  kubectl label pods --all status=unhealthy
  ```
  
- Update a pod identified by the type and name in "pod.json"
  ```bash
  kubectl label -f pod.json status=unhealthy
  ```
 
- Update pod 'foo' only if the resource is unchanged from version 1
  ```bash
  kubectl label pods foo status=unhealthy --resource-version=1
  ```
  
- Update pod 'foo' by removing a label named 'bar' if it exists Does not require the --overwrite flag
  ```bash
  kubectl label pods foo bar-
  kubectl label pod nginx app-
  kubectl label deployment myapp tier-
  ```

- Apply label to all resources of that type.
  ```bash
  kubectl label pod --all environment=prod
  ```

- Apply label to all resources of that type across all namespaces.
  ```bash
  kubectl label pods -A tier=frontend
  ```

- --dry-run
  Test without applying:
    - none → actually apply (default)
    - client → simulate locally
    - server → send request, no persistence
  ```bash
  kubectl label pod foo team=dev --dry-run=client
  ```

- Track ownership of field changes(`--field-manager`).
  ```bash
  kubectl label pods foo role=db --field-manager=my-script
  ```

- Filter resources by fields(`--field-selector`).
  ```bash
  kubectl label pods --field-selector status.phase=Running owner=team1
  kubectl label pods -l app=web version=v2 --overwrite
  ```

- Label multiple resource at once
  ```bash
  kubectl label pods pod1 pod2 pod3 env=dev
  ```

- Apply labels from a file/directory/URL(`-f, --filename`).
  ```bash
  kubectl label -f deployment.yaml app=web
  ```

- Apply labels on resources defined in a kustomize directory(`-k, --kustomize`).
  ```bash
  kubectl label -k ./kustomize-dir team=infra
  ```

- Show labels instead of modifying(`--list`).
  ```bash
  kubectl label pods foo --list
  ```

- Modify YAML locally, no API call(`--local`).
  ```bash
  kubectl label -f pod.yaml env=staging --local -o yaml
  ```

- Output format (json, yaml, name, etc)(`-o, --output`).
  ```bash
  kubectl label -f pod.yaml app=api --dry-run=client -o yaml
  ```

- Process all manifests in a directory(`-R, --recursive`).
  ```
  kubectl label -f ./manifests -R team=backend
  ```

- Ensure update only if resource version matches(`--resource-version`).
  ```bash
  kubectl label pods foo stage=prod --resource-version=3
  ```

- Select resources by labels(`-l, --selector`).
  ```bash
  kubectl label pods -l app=nginx release=stable
  ```

- Keep managedFields when outputting(`--show-managed-fields`).
  ```bash
  kubectl label -f pod.yaml env=test --dry-run=client -o yaml --show-managed-fields
  ```

- Custom output formatting(`--template`).
  ```bash
  kubectl label pods foo app=web -o go-template --template='{{.metadata.labels}}'
  ```

- Specify namespace(`-n, --namespace`)
  ```bash
  kubectl label pods foo env=prod -n test
  ```

- Use a specific kubeconfig file(`--kubeconfig`)
  ```
  kubectl label pods foo app=db --kubeconfig=/path/config
  ```

- Run as another user(`--as`)
  ```bash
  kubectl label pods foo owner=team --as admin
  ```

- Specify API server(`-s, --server`)
  ```bash
  kubectl label pods foo zone=us-east -s https://my-api:6443
  ```

- Others
  ```bash
  kubectl label nodes <node-name> app=frontend

  kubectl label nodes -l node-role.kubernetes.io/worker=true app=backend

  kubectl get nodes --show-labels

  kubectl label nodes <node-name> app=database --overwrite

  kubectl label nodes <node-name> app-

  kubectl label node mynode region=us-east

  #Scaling a deployment with app=web label
  kubectl get pods -l app=web
  kubectl get pods -l app=nginx,env=prod

  # Get pods with env=prod or env=staging
  kubectl get pods -l 'env in (prod,staging)'

  # Get pods with label env not equal to prod
  kubectl get pods -l 'env!=prod'

  # Get pods with env not equal to dev
  kubectl get pods -l 'env notin (dev)'

  # Get pods that have the label `app` (any value)
  kubectl get pods -l 'app'

  kubectl describe pod mypod | grep Labels

  # Get pods that do not have the label `tier`
  kubectl get pods -l '!tier'

  #Scaling a deployment with app=web label
  kubectl scale deployment -l app=web --replicas=5

  kubectl patch pod nginx -p '{"metadata": {"labels": {"env": "prod"}}}'
  kubectl patch pod nginx -p '{"metadata": {"labels": {"env": null}}}'
  ```


### Selectors
Selectors filter Kubernetes objects based on their labels. This is incredibly useful for querying and managing a subset of objects that meet specific criteria.

**Common Usage:**
- **Pods**: `kubectl get pods --selector app=my-app`
- **Deployments**: Used to filter the pods managed by the deployment.
- **Services**: Filter the pods to which the service routes traffic.

```bash
kubectl describe svc nginx-svc | grep Selector
```

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-rs
spec:
  replicas: 3
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
        image: nginx
```
- The selector.matchLabels ensures this RS manages only Pods with app=nginx.


```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-svc
spec:
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
```
- This Service will route traffic to all Pods with app=nginx.


```yaml
selector:
  matchExpressions:
  - { key: env, operator: In, values: [prod, staging] }
  - { key: tier, operator: NotIn, values: [debug] }
```
- Matches Pods where env is prod or staging, but not where tier=debug.

---

### Labels vs. Namespaces
- **Labels**: Organize resources within the same or across namespaces.
- **Namespaces**: Provide a way to isolate resources from each other within a cluster.

---

### Annotations
Annotations are similar to labels but attach non-identifying metadata to objects. For example, recording the release version of an application for information purposes or last applied configuration details etc.

```bash
kubectl annotate <resource> <name> <key>=<value>

kubectl annotate pod nginx description="This is Nginx pod"
kubectl annotate deployment myapp owner="Mahin Raza"

kubectl annotate pod nginx description="Updated text" --overwrite

kubectl annotate pod nginx description-
kubectl annotate deployment myapp owner-

kubectl get pod nginx -o jsonpath='{.metadata.annotations}'

kubectl get pods -o custom-columns=NAME:.metadata.name,ANNOTATIONS:.metadata.annotations

kubectl annotate pods pod1 pod2 pod3 note="multi-pod update" #annotate multiple pod

kubectl annotate pods -l app=web team="frontend"

kubectl annotate namespace dev owner="Dev Team"

kubectl patch pod nginx -p '{"metadata": {"annotations": {"note": "patched"}}}'
kubectl patch pod nginx -p '{"metadata": {"annotations": {"note": null}}}'

# Add annotation
kubectl annotate pod nginx-pod owner=devops-team

# Update annotation (overwrite)
kubectl annotate pod nginx-pod owner=platform-team --overwrite

# Remove annotation
kubectl annotate pod nginx-pod owner-

kubectl describe pod nginx-pod | grep Annotations -A 5
kubectl get pod nginx-pod -o jsonpath='{.metadata.annotations}'
```

* Pod with annotations
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
  annotations:
    description: "This pod runs nginx web server"
    contact: "devops-team@example.com"
spec:
  containers:
  - name: nginx
    image: nginx
```

* Real-world example with Ingress(Tells the NGINX ingress controller how to handle paths)
```yaml
metadata:
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
```



