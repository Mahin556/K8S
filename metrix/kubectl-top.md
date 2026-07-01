```bash

kubectl top po
kubectl top po pod_name...

kubectl top node
kubectl top node node_name...

kubectl top pod --containers
kubectl top node --containers

kubectl top pod --containers --sort-by=cpu
kubectl top pod --containers --sort-by=memory

kubectl top node --containers --sort-by=memory
kubectl top node --containers --sort-by=memory

```