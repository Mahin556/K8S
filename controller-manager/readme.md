## Check out the task.md file for the hands-on exercises

- Application is deployed as a container on the pod and then pod is backed by high-level k8s objects(RC,RS,deploy)
- Maintainer the desire no of replicas(desire no of instance)
- Support scaling(updating the replica count using declarative or immperatritive approach)
- Can spin up a pod on multiple pod
- Scaling ---> manually(using declarative or immperatritive approach) and auto(HPA,VPA).
- annocation --> one is lastappliedannotation(default,used in rollback)

## Cheatsheet for Kubernetes commands:
https://kubernetes.io/docs/reference/kubectl/quick-reference/

### Replicaset
https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/

![image](https://github.com/piyushsachdeva/CKA-2024/assets/40286378/3e9792d4-1127-44b4-a6ec-cdc2a82219e3)


### Deployment
https://kubernetes.io/docs/concepts/workloads/controllers/deployment/

![image](https://github.com/piyushsachdeva/CKA-2024/assets/40286378/b888d272-c623-4a00-8381-45c25ce9d9c0)
- mostly used 
- zero downtime deployment
- rolling update
- rollback(undo)
- manage a replicaset for each version
- manage history in form of replicaset
- default history/replicaset count in 10(can be changed)

### Replication Controller(legacy)
https://kubernetes.io/docs/concepts/workloads/controllers/replicationcontroller/


```yaml
apiVersion: v1
kind: ReplicationController
metadata: 
  name: rc-1
  namespace: default
  labels:
    app: nginx
  annotations: 
    kubernetes.io/description: "Rc example"
spec:
  replicas: 3
  selector:
    app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx-container
        image: nginx:latest
        ports:
        - containerPort: 80
```
```yaml
apiVersion: v1
kind: ReplicaSet
metadata: 
  name: Rs-1
  namespace: default
  labels:
    app: nginx
  annotations: 
    kubernetes.io/description: "Rc example"
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
      - name: nginx-container
        image: nginx:latest
        ports:
        - containerPort: 80
```
```yaml
apiVersion: v1
kind: Deployment
metadata: 
  name: deploy-1
  namespace: default
  labels:
    app: nginx
  annotations: 
    kubernetes.io/description: "deploy example"
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
      - name: nginx-container
        image: nginx:latest
        ports:
        - containerPort: 80
```

```bash
kubectl api-resources | grep -i rc
kubectl api-resources | grep -i rs
kubectl api-resources | grep -i deploy
kubectl apply -f <manifest>
kubectl describe rc <rc_name>
kubectl describe rs <rs_name>
kubectl describe deploy <deploy_name>
kubectl get pods
kubectl get rc
kubectl get rs
kubectl get deploy
kubectl get deploy -l app=nginx
kubectl edit rc <rc_name>
kubectl edit rs <rs_name>
kubectl edit deploy <deploy_name>
kubectl scale rc <rc_name> --replicas=5
kubectl scale rs <rs_name> --replicas=5
kubectl scale deploy <deploy_name> --replicas=5
kubectl delete rs/<rsname>
kubectl get all
kubectl get all -A/--all-namespaces
kubectl set image deploy/<deploy_name> <con>=<image>:<tag>
kubectl rollout history deploy/<deploy_name>
kubectl rollout undo deploy/<deploy_name> #create a new version/revision from the just previous version
kubectl create deploy <deploy_name> --image=nginx --dry-run=client -oyaml > deploy.yaml
```

- imparitive is fast(use them in exam)