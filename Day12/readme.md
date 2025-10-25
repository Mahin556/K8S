# Day 12/40 - Kubernetes Daemonset Explained - Daemonsets, Job and Cronjob in Kubernetes

## Check out the video below for Day12 👇

[![Day12/40 - Kubernetes Daemonset Explained - Daemonsets, Job and Cronjob in Kubernetes](https://img.youtube.com/vi/kvITrySpy_k/sddefault.jpg)](https://youtu.be/kvITrySpy_k)


### What is a daemonset?
- A daemon set is another type of Kubernetes object that controls pods. Unlike deployment, the DS automatically deploys 1 pod to each available node. You don't need to update the replica based on demand; the DS takes care of it by spinning X number of pods for X number of nodes.
- If you create a ds in a cluster of 5 nodes, then 5 pods will be created.
- If you add another node to the cluster, a new pod will be automatically created on the new node.

### Examples of daemonset
- kube-proxy
- calico
- weave-net
- monitoring agents etc

### Daemonset for a 3 nodes cluster

![image](https://github.com/piyushsachdeva/CKA-2024/assets/40286378/bb803dc2-f9ab-4fe3-a0bb-0eacdfcf3ce0)

- we can only schedule a control plane component on a control node.
```bash
kubectl get ds -n kube-system
```
- does not deploy at control-node because it is tainted

### Sample DS yaml used in the demo

```yaml
apiVersion: apps/v1
kind:  DaemonSet
metadata:
  name: nginx-ds
  labels:
    env: demo
spec:
  template:
    metadata:
      labels:
        env: demo
      name: nginx
    spec:
      containers:
      - image: nginx
        name: nginx
        ports:
        - containerPort: 80
  selector:
    matchLabels:
      env: demo
```

* Cron-job
```bash
apiVersion: batch/v1
kind: CronJob
metadata:
  name: hello
spec:
  schedule: "* * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hello
            image: busybox:1.28
            imagePullPolicy: IfNotPresent
            command:
            - /bin/sh
            - -c
            - date; echo Hello from the Kubernetes cluster
          restartPolicy: OnFailure
```

* Job
```bash
https://kubernetes.io/docs/concepts/workloads/controllers/job/
```