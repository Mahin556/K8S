```bash
controlplane:~$ kubectl top pods -n kube-system
NAME                                      CPU(cores)   MEMORY(bytes)   
calico-kube-controllers-7bb4b4d4d-8q2r8   2m           43Mi            
canal-djlvw                               12m          118Mi           
canal-kc8m5                               14m          120Mi           
coredns-76bb9b6fb5-4z2mz                  1m           67Mi            
coredns-76bb9b6fb5-7vqcc                  1m           13Mi            
etcd-controlplane                         11m          61Mi            
kube-apiserver-controlplane               21m          278Mi           
kube-controller-manager-controlplane      8m           80Mi            
kube-proxy-6rdfg                          1m           32Mi            
kube-proxy-kgl6s                          1m           54Mi            
kube-scheduler-controlplane               4m           34Mi            
metrics-server-f7747588-5nz6m             2m           16Mi  
```
```bash
controlplane:~$ kubectl get pods -n kube-system | grep -i metrics-server
metrics-server-f7747588-5nz6m             1/1     Running   0             17m
```
```bash
controlplane:~$ kubectl top pods -n kube-system | sort -k2 -hr #Top pods with CPU usage
kube-apiserver-controlplane               27m          267Mi           
canal-djlvw                               14m          117Mi           
etcd-controlplane                         12m          55Mi            
canal-kc8m5                               12m          119Mi           
kube-controller-manager-controlplane      9m           81Mi            
kube-scheduler-controlplane               5m           30Mi            
metrics-server-f7747588-5nz6m             2m           16Mi            
calico-kube-controllers-7bb4b4d4d-8q2r8   2m           43Mi            
kube-proxy-kgl6s                          1m           53Mi            
kube-proxy-6rdfg                          1m           32Mi            
coredns-76bb9b6fb5-7vqcc                  1m           13Mi            
coredns-76bb9b6fb5-4z2mz                  1m           67Mi
```
```bash
controlplane:~$ kubectl top pods -n kube-system | sort -k3 -hr #Top pods with Memory usage
kube-apiserver-controlplane               27m          267Mi           
canal-kc8m5                               12m          119Mi           
canal-djlvw                               14m          117Mi           
kube-controller-manager-controlplane      9m           81Mi            
coredns-76bb9b6fb5-4z2mz                  1m           67Mi            
etcd-controlplane                         12m          55Mi            
kube-proxy-kgl6s                          1m           53Mi            
calico-kube-controllers-7bb4b4d4d-8q2r8   2m           43Mi            
kube-proxy-6rdfg                          1m           32Mi            
kube-scheduler-controlplane               5m           30Mi            
metrics-server-f7747588-5nz6m             2m           16Mi            
coredns-76bb9b6fb5-7vqcc                  1m           13Mi
```
```bash
controlplane:~$ kubectl top nodes
NAME           CPU(cores)   CPU(%)   MEMORY(bytes)   MEMORY(%)   
controlplane   78m          7%       1358Mi          66%         
node01         25m          2%       810Mi           44%  

```