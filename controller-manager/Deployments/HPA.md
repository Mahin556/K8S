### Horizontal Pod Autoscaling (HPA)
- HPA automatically scales the number of Pods based on observed metrics (usually CPU or memory usage). This is ideal for handling fluctuating workloads without manual intervention.
```
kubectl autoscale deployment/<deployment-name> --min=<min-pods> --max=<max-pods> --cpu-percent=<target-cpu-usage>
```
```
kubectl autoscale deployment/tomcat-1stdeployment --min=5 --max=8 --cpu-percent=75
```
Explanation:
- --min=5: Minimum number of Pods to maintain, even if traffic is low.
- --max=8: Maximum number of Pods HPA can scale to under high load.
- --cpu-percent=75: Target average CPU utilization. If Pods exceed 75% CPU, HPA will scale up; if lower, it may scale down.
```
kubectl get hpa
```

