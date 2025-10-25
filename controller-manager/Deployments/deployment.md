### Deployment

- It is high-level k8s api resource.
- Manage stateless application.
- It is a type of controller that manage desired no of replicas of pods defined in a configuration in the cluster.
- Handle pod failover.
- Support rolling update, rollback(if rolling update failes), Pause/resume deployments and maintain the version history.
- Desire state stored in etcd and deployment monitor the current state(no of replicas or image:version) and makesure current state always equal to desire state.
- For statefull application statefull sets are used for stable network indentity and persistent storage.
- A Pod itself cannot scale — scaling is achieved by increasing the number of Pods that a Deployment manages.
- A Deployment usually represents a single purpose (e.g., one microservice or one application component).
- In BTS Deployment create a RS and RS manage the Replicas.
- Deployment use the RS to manage different verison of replicas.
- Deployments are configured using declarative YAML or JSON files.
- Scalable - scale up and down according to requirement.
- Deployment Strategies:- 
    - Recreate
        Deletes all existing Pods first, then brings up new Pods.
        Quick but have downtime.
    - Rolling Update (default)
        Gradually replaces old Pods with new Pods.
        Pros: No downtime; a few old Pods + a few new Pods always running
        Cons: Deployment takes longer
- A deployment configuration can be of YAML or JSON format.

![Alt Text](/images/deployment.png)
![Alt Text](https://github.com/Mahin556/K8S-artifects/blob/main/images/dp1.png)

```yaml
apiVersion: apps/v1        # API version: apps/v1 is the stable version for Deployment
kind: Deployment           # Resource type: Deployment ensures Pods are created & managed
metadata:
  name: nginx-deploy       # Name of the Deployment, must be unique in the namespace
  labels:
    app: nginx             # Labels help identify & organize resources; used by selectors
spec:
  replicas: 3              # Desired number of Pod instances
  selector:
    matchLabels:
      app: nginx           # Selector to match Pods created by this Deployment
                            # MUST match template.metadata.labels
  template:                # Pod template: defines Pods that Deployment will create
    metadata:
      labels:
        app: nginx         # Labels on Pods; required to match selector for management
    spec:
      containers:          # PodSpec: container definitions
      - name: nginx        # Container name (unique in the Pod)
        image: nginx:1.21  # Docker image to run
        ports:
        - containerPort: 80  # Port exposed by container
        # Optional fields you can add here:
        # env:               # environment variables
        # volumeMounts:      # attach volumes inside container
        # resources:        # CPU/memory requests & limits
        # livenessProbe:    # health check to restart container if unhealthy
        # readinessProbe:   # check if container is ready to serve traffic

```

```yaml
apiVersion: apps/v1                # Deployment API version (use apps/v1; extensions/v1beta1 is deprecated)
kind: Deployment                   # Resource type
metadata:
  name: tomcat-deployment          # Deployment name
spec:
  replicas: 3                       # Number of Pods to run
  selector:                         # Selector must match template labels
    matchLabels:
      app: tomcat
      tier: backend
  template:                         # Pod template
    metadata:
      labels:
        app: tomcat
        tier: backend
    spec:
      containers:
      - name: tomcat-container      # Container name
        image: tomcat:8.0           # Container image
        ports:
        - containerPort: 7474       # Exposed port
```

### Deployment Status
| Status      | Meaning                                                                      |
| ----------- | ---------------------------------------------------------------------------- |
| Pending     | Deployment is waiting to start or facing issues to start.                    |
| Progressing | Deployment is updating or creating a new ReplicaSet.                         |
| Succeeded   | Deployment completed successfully without errors.                            |
| Failed      | Deployment failed due to some issues.                                        |
| Unknown     | Kubernetes API cannot reach the deployment, or there is an internal problem. |

```
kubectl rollout status deployment/<deployment-name>
```

### References:
- https://spacelift.io/blog/kubernetes-deployment-yaml
- https://www.tutorialspoint.com/kubernetes/kubernetes_deployments.htm
- https://www.geeksforgeeks.org/devops/kubernetes-deployment/
- https://devopscube.com/kubernetes-deployment-tutorial/