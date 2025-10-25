```bash
kubectl get rs
kubectl get replicasets
kubectl describe rs <name>
kubectl edit replicaset webapp
kubectl delete replicaset/webapp
kubectl scale rs <name> --replicas=5
kubectl apply -f replicaSet.yaml
kubectl get pods -l 'tier in (frontend,backend)'
kubectl get pods -l 'app=myapp,env notin (dev)'

#You can delete a ReplicaSet without affecting any of its Pods using kubectl delete with the — cascade=false option.
kubectl delete replicaset/webapp --cascade=false


```

### References
- https://spacelift.io/blog/kubernetes-cheat-sheet
- https://medium.com/avmconsulting-blog/replication-controller-replica-sets-in-kubernetes-820f3cec7170
- https://learncloudnative.com/blog/2021-07-10-pods-replicasets
