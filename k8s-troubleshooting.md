* https://youtu.be/QLbfY_Uh63c?si=H8k-GBRvDyR7n5U5
```bash
────────────────────────────────────────────────────────
KUBERNETES – REAL WORLD / CORPORATE COMMANDS CHEAT SHEET
────────────────────────────────────────────────────────

• Restart Pods safely after ConfigMap/Secret change --> deployment create a new replica set and increase a no of pod one by one(default)
  kubectl rollout restart deployment <deployment-name> -n <namespace>

• Roll back a faulty deployment to previous stable version
  kubectl rollout undo deployment <deployment-name> -n <namespace>

• View full rollout history with revision numbers
  kubectl rollout history deployment <deployment-name> -n <namespace>

• View rollout history with change cause
  kubectl annotate deployment <deployment-name> \
  kubernetes.io/change-cause="reason for change" -n <namespace>

• List Pods in a namespace
  kubectl get pods -n <namespace>

• Check mounted volumes and disk usage inside a Pod
  kubectl exec <pod-name> -n <namespace> -- df -h
  kubectl exec mysql-249824 -n db -- df -h | grep /var/lib/mysql

• View decoded value of a Secret
  kubectl get secret <secret-name> -n <namespace> \
  -o jsonpath="{.data.<key>}" | base64 --decode

• View logs from previous crashed container
  kubectl logs -p <pod-name> -n <namespace>

• Simulate Pod crash (kill main process)
  kubectl exec <pod-name> -n <namespace> -- kill 1

• Manually test Readiness Probe
  kubectl exec <pod-name> -n <namespace> -- \
  wget -qO- http://localhost:8080/health

• Test internal Service DNS resolution
  kubectl run mysql-client --rm -it \
  --image=mysql -- mysql -h <service-name> -u <user> -p

• List all PVC mount paths in a Pod
  kubectl describe pod <pod-name> -n <namespace> | grep -A10 Mounts

• Get Service endpoints and actual Pod IPs
  kubectl get endpoints <service-name> -n <namespace>

• Get Pod IP and Node details
  kubectl get pods -o wide -n <namespace>

• View all cluster events sorted by time
  kubectl get events -n <namespace> --sort-by=.metadata.creationTimestamp

  kubectl get sc
  kubectl get pv
  kubectl get pvc
• Patch StorageClass to allow volume expansion
  kubectl patch storageclass <sc-name> \
  -p '{"allowVolumeExpansion": true}'

• Resize PersistentVolumeClaim (PVC)(limit)
  kubectl patch pvc <pvc-name> -n <namespace> \
  -p '{"spec":{"resources":{"requests":{"storage":"15Gi"}}}}'

• Trigger CrashLoopBackOff manually
  kubectl exec <pod-name> -n <namespace> -- kill 1

• Enable Horizontal Pod Autoscaler (HPA)
  kubectl autoscale deployment <deployment-name> \
  --cpu-percent=50 --min=1 --max=5 -n <namespace>

• View HPA status
  kubectl get hpa -n <namespace>
  kubectl describe hpa <hpa-name> -n <namespace>

• Live stream Pod CPU & Memory usage
  watch kubectl top pods -n <namespace>

• Backup PVC data from Pod to local system
  kubectl cp <namespace>/<pod-name>:/var/lib/mysql ./mysql-backup

• Drain a worker node before maintenance
  kubectl drain <node-name> --ignore-daemonsets --delete-emptydir-data

• Enable scheduling again after maintenance
  kubectl uncordon <node-name>

• Restart all Deployments in a namespace
  kubectl rollout restart deployment -n <namespace>

• List Deployments with container image versions
  kubectl get deployment -n <namespace> \
  -o=jsonpath='{range .items[*]}{.metadata.name}{" -> "}{.spec.template.spec.containers[*].image}{"\n"}{end}'

• Find which node a Pod is running on
  kubectl get pod <pod-name> -n <namespace> -o wide

• Force delete a stuck Pod
  kubectl delete pod <pod-name> -n <namespace> \
  --grace-period=0 --force

• Compare live cluster state with manifest file
  kubectl diff -f <manifest.yaml>

────────────────────────────────────────────────────────
WHY THESE COMMANDS MATTER
────────────────────────────────────────────────────────

• Used daily by DevOps / SRE teams
• Required for production debugging
• Asked in real Kubernetes interviews
• Essential for cluster maintenance
• Helps in RCA (Root Cause Analysis)

────────────────────────────────────────────────────────
```