### ReplicationController & Restart Policy
- When you define a ReplicationController, it manages Pods.
- Each Pod template inside an RC must have `restartPolicy: Always`.
- Infact each Controller must have container restart policay `Always`

```
apiVersion: v1
kind: ReplicationController
metadata:
  name: myapp-rc
spec:
  replicas: 3
  selector:
    app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp-container
        image: nginx:1.19
      restartPolicy: Always   # <-- Only valid option here(default)

```

#### Why only Always for RC?
Because:
- RC ensures a desired number of Pods is running.
- If a Pod dies, kubelet restarts the container (since Always).
- If the Pod itself is deleted, the RC creates a new Pod.
- If Kubernetes allowed `OnFailure` or `Never` under an RC, it would conflict with the controller’s job of keeping Pods running.
- If a pod's container terminates for any reason—whether due to success or failure—the ReplicationController's default behavior is to have the kubelet agent on the node restart that container to restore the desired state.
-  Attempting to specify OnFailure or Never in a ReplicationController manifest will cause the Kubernetes API to reject the pod template. 
