```bash
k get nodes -ojsonpath='{.items[?(@.metadata.labels.node-role\.kubernetes\.io/control-plane)].metadata.name}' | xargs -I {} kubectl get node {} -ojsonpath='{.spec.taints}'
```