Here’s a structured, easy-to-read version of your **Scaling a Deployment** explanation:

---

# Scaling a Deployment in Kubernetes

Scaling allows you to **increase or decrease the number of Pods** in a Deployment to handle changes in load.

---

## 1. Manual Scaling

You can manually scale a Deployment using:

```bash
kubectl scale deployment/nginx-deployment --replicas=10
```

**Output:**

```
deployment.apps/nginx-deployment scaled
```

This immediately sets the Deployment to run 10 Pods.

---

## 2. Horizontal Pod Autoscaling (HPA)

If **Horizontal Pod Autoscaling** is enabled in your cluster, you can automatically scale Pods based on CPU usage:

```bash
kubectl autoscale deployment/nginx-deployment --min=10 --max=15 --cpu-percent=80
```

**Explanation:**

* `--min=10` → minimum number of Pods
* `--max=15` → maximum number of Pods
* `--cpu-percent=80` → target CPU utilization for scaling

**Output:**

```
deployment.apps/nginx-deployment scaled
```
### References
- https://kubernetes.io/docs/concepts/workloads/controllers/deployment/
