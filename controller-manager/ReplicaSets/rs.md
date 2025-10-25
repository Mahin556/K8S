- Manage the replicas of pods
- support both equality based and set-based selector

![](https://github.com/Mahin556/K8S-artifects/blob/main/images/rs1.png)

```yaml
apiVersion: extensions/v1beta1 --------------------->1
kind: ReplicaSet --------------------------> 2
metadata:
  name: Tomcat-ReplicaSet
spec:
  replicas: 3
  selector:
    matchLables:
      tier: Backend ------------------> 3
    matchExpression:
      { key: tier, operation: In, values: [Backend]} --------------> 4
template:
  metadata:
    lables:
      app: Tomcat-ReplicaSet
      tier: Backend
   spec:
      containers:
      - name: Tomcat
        image: tomcat: 8.0
        ports:
        - containerPort: 7474
```
```yaml
apiVersion: extensions/v1beta1 --------------------->1
kind: ReplicaSet --------------------------> 2
metadata:
  name: Tomcat-ReplicaSet
spec:
  replicas: 3
  selector:
    matchLables:
      tier: Backend ------------------> 3
    matchExpression:
      - key: tier
        operation: In
        values: [Backend] --------------> 4
template:
  metadata:
    lables:
      app: Tomcat-ReplicaSet
      tier: Backend
   spec:
      containers:
      - name: Tomcat
        image: tomcat: 8.0
        ports:
        - containerPort: 7474
```

#### Equality based selector
- Uses = or == or !=
- Matches Pods with labels equal (or not equal) to the given value.
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: frontend-rs
spec:
  replicas: 3
  selector:
    matchLabels:        # equality-based (simple key=value)
      app: frontend
  template:
    metadata:
      labels:
        app: frontend   # must match selector
    spec:
      containers:
      - name: nginx
        image: nginx:1.19
```
Here:
    matchLabels: { app: frontend }
    Pods must have label app=frontend.

#### Set-based selector
- Uses In, NotIn, Exists, or DoesNotExist.
- Allows more expressive selection.
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: backend-rs
spec:
  replicas: 2
  selector:
    matchExpressions:     # set-based selector
      - key: tier
        operator: In
        values:
          - backend
          - api
      - key: env
        operator: NotIn
        values:
          - dev
  template:
    metadata:
      labels:
        tier: backend
        env: prod
    spec:
      containers:
      - name: backend-app
        image: my-backend:2.0

```
Here:
    Pod must have tier=backend or tier=api
    And env label must not be dev

#### ReplicaSet with Both Equality-based & Set-based Selectors
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: combined-rs
spec:
  replicas: 3
  selector:
    matchLabels:              # Equality-based selector
      app: myapp
    matchExpressions:         # Set-based selectors
      - key: tier
        operator: In
        values:
          - frontend
          - backend
      - key: env
        operator: NotIn
        values:
          - dev
  template:
    metadata:
      labels:
        app: myapp
        tier: frontend
        env: prod
    spec:
      containers:
      - name: myapp-container
        image: nginx:1.25
```

- If we create a rs and pod with this `app.kubernetes.io/name: hello` labels and selector respectively.
- Then is we create a standalone Pod (stray-pod) with a label `app.kubernetes.io/name: hello`.
- The ReplicaSet hello already manages 5 Pods and its selector matches the same label (app.kubernetes.io/name=hello).
- Kubernetes saw your stray-pod and said:
    - “This Pod matches the ReplicaSet’s label selector.”
    - Since it didn’t have an ownerReference, ReplicaSet adopted it.
- Now ReplicaSet temporarily had 6 Pods (5 existing + 1 stray), but the ReplicaSet spec says `replicas: 5`.
- So, to reconcile desired state vs actual state, it deleted the extra Pod (your stray one).
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: stray-pod
  labels:
    app.kubernetes.io/name: hello
spec:
  containers:
    - name: stray-pod-container
      image: busybox
      command: ['sh', '-c', 'echo Hello from stray pod! && sleep 3600']

```

- For the template's restart policy field, `.spec.template.spec.restartPolicy`, the only allowed value is `Always`, which is the default.



### References
- https://learncloudnative.com/blog/2021-07-10-pods-replicasets
- https://www.tutorialspoint.com/kubernetes/kubernetes_replica_sets.htm
- https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/