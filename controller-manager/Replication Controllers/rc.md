- Manages a replicas of pod (there lifecycle)
- Auto healing
- Handle pod failover
- Not run single pod instead run pod through rc,rs,deployment for handling failover or if node fail where pod is running it will allocate it on different node.
- without Rc,Rs,deployment a unmanaged pod will be terminated and not gonna deploy to other node if node failed.
- Only support equility based selector, not set-based selector
- Resources are identified based on matching labels and their values.
- Using labels is can also manage the existing pods with the same labels, and adjust the replica count to create accourding to it.
- selector == pod selector
- If a Pod managed by an RC fails, is deleted, or its node crashes, the RC will automatically create a new Pod to replace it, maintaining the desired replica count. 

- ReplicationControllers do not support revisions or history.
  - They’re older K8s objects, and all you can do is update the spec (e.g., image). No rollout tracking, no annotations like revisions.
  - If you want revision history, you should use a Deployment instead of an RC.

- ReplicationController is often abbreviated to "rc" in discussion, and as a shortcut in kubectl commands.
  
![](https://github.com/Mahin556/K8S-artifects/blob/main/images/rc1.png)

```
apiVersion: v1
kind: ReplicationController
metadata:
  name: Tomcat-ReplicationController
spec:
  replicas: 3 
  selectors:
    app: App
    component: neo4j
  template:
    metadata:
      name: Tomcat-ReplicationController
      labels:
        app: App
        component: neo4j
    spec:
      containers:
      - name: Tomcat
        image: tomcat: 8.0
        ports:
        - containerPort: 7474
```

### References
- https://www.tutorialspoint.com/kubernetes/kubernetes_replication_controller.htm
- https://kubernetes.io/docs/concepts/workloads/controllers/replicationcontroller/
- https://www.geeksforgeeks.org/devops/kubernetes-replication-controller/