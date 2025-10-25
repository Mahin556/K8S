### References:- 
- https://freedium-mirror.cfd/https://medium.com/@a-dem/a-simple-guide-to-kubectl-run-de8d67aac482
- https://kubernetes.io/docs/reference/kubectl/generated/kubectl_run/
- https://kubernetes.io/docs/reference/kubectl/ *

---

* In modern Kubernetes versions (v1.18+), kubectl run creates a Pod. 
* For Deployments, use kubectl create deployment or YAML manifests.

```bash
kubectl run nginx --image=nginx

kubectl run nginx --image=nginx --dry-run=client -oyaml > pod.yaml

kubectl run nginx --image=nginx --dry-run=client -ojson > pod.json

# Start a hazelcast pod and let the container expose port 5701
kubectl run hazelcast --image=hazelcast/hazelcast --port=5701

# Start a hazelcast pod and set labels "app=hazelcast" and "env=prod" in the container
kubectl run hazelcast --image=hazelcast/hazelcast --labels="app=hazelcast,env=prod"

# Start a nginx pod, but overload the spec with a partial set of values parsed from JSON
kubectl run nginx --image=nginx --overrides='{ "apiVersion": "v1", "spec": { ... } }'

# Start a busybox pod and keep it in the foreground, don't restart it if it exits
kubectl run -i -t busybox --image=busybox --restart=Never

# Start the nginx pod using the default command, but use custom arguments (arg1 .. argN) for that command
kubectl run nginx --image=nginx -- <arg1> <arg2> ... <argN>

kubectl get -f ./pod.yaml

# Start the nginx pod using a different command and custom arguments
kubectl run nginx --image=nginx --command -- <cmd> <arg1> ... <argN>

kubectl get pod/example-pod1 replicationcontroller/example-rc1

kubectl get pod

kubectl get pod -owide

kubectl describe pod <name>

kubectl delete pod <name>

kubectl exec -it <pod> -- /bin/sh

kubectl top pod

kubectl port-forward <pod> 8080:80

kubectl label pod <pod> team=dev

kubectl annotate pod <pod> key=value

kubectl get po -l=app.kubernetes.io/name=hello

kubectl run -i --tty --rm debug-shell --image=busybox -- sh #Automatically remove the pod after you exit.

kubectl run my-app --image=my-image --env="DB_HOST=mydbhost" --env="DB_PORT=5432"

kubectl create -f pod.yaml

kubectl run mypod --image=nginx --restart=Never

kubectl get pods -A/--all-namespaces

kubectl delete -f pod.yaml

kubectl delete pod <pod-name>

kubectl delete pods --all -A

kubectl exec -it <pod-name> -- bash
kubectl exec -it <pod-name> -- sh

kubectl cp ./localfile <pod-name>:/app/remote-file
kubectl cp <pod-name>:/app/remote-file ./localfile

kubectl port-forward <pod-name> 8080:80

kubectl logs <pod-name>

kubectl logs <pod-name> -c <container-name>

kubectl logs -f <pod-name>

kubectl debug -it <pod-name> --image=busybox

kubectl get pods -w

kubectl get events --sort-by=.metadata.creationTimestamp

kubectl logs -f testpod1 -c c01

yq eval '.metadata.name="nginx1"' pod.yaml | kubectl apply -f -


kubectl get events
```

### References
- https://spacelift.io/blog/kubernetes-cheat-sheet
- https://www.geeksforgeeks.org/devops/kubernetes-creating-multiple-container-in-a-pod/
