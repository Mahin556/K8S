## Check out the task.md file for day07 task details

## Different ways of creating a Kubernetes object
- Imperative way ( Through command or API calls) --> troubleshooting, fast, simple, local deployment
- Declarative way ( By creating manifest files) --> producation deployment, VCS, gitops, ci/cd

![image](https://github.com/piyushsachdeva/CKA-2024/assets/40286378/b038c4d3-87b7-474d-a3aa-5983d978f885)

## commands
```bash
kubectl run nginx --image=nginx:latest
```
There are multiple states of pod ---> pending, container creating, running, failed, successed, unknown, crashloopbackoff, ImagePullbackoff etc

```bash
kubectl describe pod nginx
kubectl edit pod nginx --labels=app=nginx,env=prod
kubectl run nginx --image=nginx --dry-run=client -oyaml > file.yaml
kubectl run nginx --image=nginx --dry-run=client -ojson > file.json
kubectl get pods -l app=nginx
kubectl get pods -owide
kubectl get nodes -owide
```

## Below is the sample pod YAML used in the video:

```YAML
# This is a sample pod yaml

apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    env: demo
    type: frontend
spec:
  containers:
  - name: nginx-container
    image: nginx
    ports:
    - containerPort: 80
```

