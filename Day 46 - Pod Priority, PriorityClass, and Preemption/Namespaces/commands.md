### References:
- https://www.geeksforgeeks.org/devops/kubernetes-namespaces/

```bash
kubectl get namespace # View all existing namespaces

# Create a new namespace called jacks-blog
kubectl create namespace jacks-blog

kubectl describe namespace dev

kubectl delete namespace dev
kubectl delete namespace namespace1 namespace2 namespace3

kubectl run nginx --image=nginx -n demo

kubectl create deploy demo -n demo

kubectl top namespace dev

kubectl api-resources | grep -i true

kubectl api-resources --namespaced=true
    
kubectl api-resources --namespaced=true -o name

kubectl api-resources --namespaced=true | awk 'NR>1 {print $1}' #Get only the "NAME" column (resource names)

kubectl api-resources --namespaced=true | awk 'NR>1 {print $2}' #Get only the "KIND" column

kubectl api-resources --namespaced=true | awk 'NR>1 {print $1, "-", $2}' #Get both NAME and KIND neatly

kubectl get all --namespace=kube-system

# Show resource usage of pods in ingress-nginx namespace
kubectl top pod --namespace=ingress-nginx

# List pods in the default namespace
kubectl get pods

# List the pods contained in a namespace
kubectl get pods --namespace ingress-nginx

# Note the short format for namespace can be used (-n)
kubectl get pods -n ingress-nginx

kubectl apply -f my-config-map.yaml --namespace=<namespace_name>

kubectl get all -n <namespace>
kubectl get all --all-namespaces
```
```bash
kubectl get all --namespace=kube-system
```
```bash
NAME                                          READY   STATUS    RESTARTS      AGE
pod/calico-kube-controllers-fdf5f5495-vxpw5   1/1     Running   2 (62m ago)   10d
pod/canal-4xxhw                               2/2     Running   2 (62m ago)   10d
pod/canal-wlt99                               2/2     Running   2 (62m ago)   10d
pod/coredns-6ff97d97f9-4wljj                  1/1     Running   1 (62m ago)   10d
pod/coredns-6ff97d97f9-wmpcb                  1/1     Running   1 (62m ago)   10d
pod/etcd-controlplane                         1/1     Running   3 (62m ago)   10d
pod/kube-apiserver-controlplane               1/1     Running   3 (62m ago)   10d
pod/kube-controller-manager-controlplane      1/1     Running   2 (62m ago)   10d
pod/kube-proxy-d8lhg                          1/1     Running   1 (62m ago)   10d
pod/kube-proxy-txl4k                          1/1     Running   2 (62m ago)   10d
pod/kube-scheduler-controlplane               1/1     Running   2 (62m ago)   10d

NAME               TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE
service/kube-dns   ClusterIP   10.96.0.10   <none>        53/UDP,53/TCP,9153/TCP   10d

NAME                        DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
daemonset.apps/canal        2         2         2       2            2           kubernetes.io/os=linux   10d
daemonset.apps/kube-proxy   2         2         2       2            2           kubernetes.io/os=linux   10d

NAME                                      READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/calico-kube-controllers   1/1     1            1           10d
deployment.apps/coredns                   2/2     2            2           10d

NAME                                                DESIRED   CURRENT   READY   AGE
replicaset.apps/calico-kube-controllers-fdf5f5495   1         1         1       10d
replicaset.apps/coredns-674b8bbfcf                  0         0         0       10d
replicaset.apps/coredns-6ff97d97f9                  2         2         2       10d
```

### References
- https://spacelift.io/blog/kubernetes-cheat-sheet
