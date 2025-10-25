```
kubectl get rc

kubectl explain replicationcontroller

kubectl api-resources | grep rc

kubectl get replicationcontrollers

kubectl get rc -n dev

kubectl describe replicationcontroller <controller-name>

kubectl delete pod <one-of-your-pod-names>

kubectl create -f replication-controller.yaml

kubectl replace -f replication-controller.yaml

kubectl scale --replicas=5 replicationcontroller <controller-name>

kubectl delete replicationcontroller <controller-name>

kubectl get pods --selector=<selector>

kubectl get pods --selector=<selector> -o wide

kubectl expose rc <controller-name> --port=80 --target-port=8080 --type=LoadBalancer

kubectl set image replicationcontroller <controller-name> <container-name>=<new-image> ✔️

kubectl rollout status replicationcontroller <controller-name> ❌

kubectl rollout pause replicationcontroller <controller-name> ❌

kubectl rollout resume replicationcontroller <controller-name> ❌

kubectl rollout undo replicationcontroller <controller-name> ❌

kubectl rollout undo replicationcontroller <controller-name> --to-revision=<revision-number ❌

kubectl edit replicationcontroller <controller-name>

kubectl apply -f replication-controller.yaml --dry-run=client

kubectl get replicationcontroller <controller-name> -o yaml > replication-controller-export.yaml

kubectl delete pods --selector=<selector>

kubectl annotate replicationcontroller <controller-name> key1=value1 key2=value2

kubectl get events --field-selector involvedObject.kind=ReplicationController --watch

# To list all the pods that belong to the ReplicationController in a machine readable form, you can use a command like this:
pods=$(kubectl get pods --selector=app=nginx --output=jsonpath={.items..metadata.name})
echo $pods
```

### References
- https://spacelift.io/blog/kubernetes-cheat-sheet
- https://nidhiashtikar.medium.com/replication-controller-a1c7f103758b
