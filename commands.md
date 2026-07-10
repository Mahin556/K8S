## Master Command Reference

| Command | Purpose |
|---------|---------|
| `kubectl logs <pod>` | Check pod logs |
| `kubectl describe pod <pod>` | Inspect pod details |
| `kubectl describe node <node>` | Inspect node details |
| `kubectl top node` | Check node resource usage |
| `kubectl top pod` | Check pod resource usage |
| `kubectl get events` | View cluster events |
| `kubectl get pv` / `kubectl get pvc` | Check persistent volumes |
| `kubectl get storageclass` | List storage classes |
| `kubectl get endpoints` | View service endpoints |
| `kubectl get hpa` | Check HPA status |
| `kubectl describe hpa <name>` | HPA details |
| `kubectl rollout restart deployment <name>` | Rolling restart |
| `kubectl rollout undo deployment <name>` | Rollback deployment |
| `kubectl rollout history deployment <name>` | View rollout history |
| `kubectl drain <node> --ignore-daemonsets --force` | Drain a node |
| `kubectl uncordon <node>` | Make node schedulable again |
| `kubectl delete pod <pod> --grace-period=0 --force` | Force delete pod |
| `kubectl exec -it <pod> -- <command>` | Execute command in pod |
| `journalctl -u kubelet` | Check kubelet logs |
| `systemctl restart kubelet` | Restart kubelet service |
| `etcdctl snapshot save` | Backup etcd |
| `kubeadm upgrade apply <version>` | Upgrade cluster |