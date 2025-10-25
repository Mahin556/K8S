### References:- 
- https://spacelift.io/blog/kubectl-exec

---

* Used to get shell access in the container within a Pod and run some command in a container.
* use case
    * Logs.
    * Inspect processes.
    * FS
    * Mount Points
    * Install packages
    * Modify something
    * ENVs etc.

```bash
kubectl exec [OPTIONS] POD_NAME [-c CONTAINER] -- COMMAND [ARGS...]
```
| Part                | Description                                                                        |
| ------------------- | ---------------------------------------------------------------------------------- |
| `POD_NAME`          | Name of the Pod where the command will run.                                        |
| `-c CONTAINER`      | Optional. Specify which container inside the Pod if there are multiple containers. |
| `--`                | Required to separate the command you want to run from kubectl options.             |
| `COMMAND [ARGS...]` | The command you want to execute inside the container.                              |

| Option                  | Short | Description                                                                                |
| ----------------------- | ----- | ------------------------------------------------------------------------------------------ |
| `--stdin`               | `-i`  | Passes standard input (stdin) to the container. Needed for interactive commands like bash. |
| `--tty`                 | `-t`  | Allocates a TTY for the session. Useful for interactive shells.                            |
| `--quiet`               | `-q`  | Suppresses kubectl output, only shows the command output.                                  |
| `--pod-running-timeout` | —     | Wait time until at least one Pod is running. Example: `--pod-running-timeout=2m`.          |

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: aks-helloworld-one
spec:
  replicas: 1
  selector:
    matchLabels:
      app: aks-helloworld-one
  template:
    metadata:
      labels:
        app: aks-helloworld-one
    spec:
      containers:
      - name: aks-helloworld-one
        image: mcr.microsoft.com/azuredocs/aks-helloworld:v1
        ports:
        - containerPort: 80
        env:
        - name: TITLE
          value: "Welcome to Azure Kubernetes Service (AKS)"
---
apiVersion: v1
kind: Service
metadata:
  name: aks-helloworld-one
spec:
  type: ClusterIP
  ports:
  - port: 80
  selector:
    app: aks-helloworld-one
```

```bash
kubectl exec --stdin --tty aks-helloworld-one-56c7b8d79d-xqx5t -- /bin/bash #interactive shell

kubectl exec -it aks-helloworld-one-56c7b8d79d-xqx5t -- /bin/sh

kubectl exec aks-helloworld-one-56c7b8d79d-xqx5t -- ls /usr/share/nginx/html

kubectl exec -it my-pod -c my-container -- /bin/bash

kubectl exec --pod-running-timeout=2m -it my-pod -- /bin/bash #Waits up to 2 minutes for the Pod to be ready before executing the command

kubectl exec aks-helloworld-one-56c7b8d79d-xqx5t -- ps aux

kubectl exec aks-helloworld-one-56c7b8d79d-xqx5t -- ls

kubectl exec aks-helloworld-one-56c7b8d79d-xqx5t -- env

kubectl exec aks-helloworld-one-56c7b8d79d-xqx5t -- apt-get update

kubectl exec aks-helloworld-one-56c7b8d79d-xqx5t -- cat /proc/1/mounts

```