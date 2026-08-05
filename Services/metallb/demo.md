```bash
cat <<EOF>  config.yaml 
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
- role: worker
- role: worker
EOF
```
```bash
kind create cluster --name=demo --config=config.yaml
```
```bash
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.15.3/config/manifests/metallb-native.yaml
```
```bash
kubectl apply -f -<<EOF
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: default-pool
  namespace: metallb-system
spec:
  addresses:
  - 172.18.0.240-172.18.0.250
EOF
```
```bash
kubectl apply -f -<<EOF
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: default
  namespace: metallb-system
spec:
  ipAddressPools:
  - default-pool
EOF
```
```bash
root@master:~# k get ipaddresspools.metallb.io\,l2advertisements.metallb.io -n metallb-system 
NAME                                    AUTO ASSIGN   AVOID BUGGY IPS   ADDRESSES
ipaddresspool.metallb.io/default-pool   true          false             ["172.18.0.240-172.18.0.250"]

NAME                                 IPADDRESSPOOLS     IPADDRESSPOOL SELECTORS   INTERFACES
l2advertisement.metallb.io/default   ["default-pool"]       
```
```bash
kubectl create deployment hello-server --image=gcr.io/google-samples/hello-app:1.0
kubectl expose deployment hello-server --type LoadBalancer --port 80 --target-port 8080
```
```bash
k get svc hello-server
k get ep hello-server
```
```bash
kubectl scale --replicas=3 deployment hello-server
```
```bash
for i in {1..5}; do curl http://172.18.0.240; done
```

### Layer 2 mode disadvantages
- Only single node Loadbalancing (lb on pod on one node).
- Provide failove --> if leader node failes then route to pod on other nodes.
https://metallb.io/concepts/layer2/