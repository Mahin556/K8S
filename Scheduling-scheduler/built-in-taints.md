#### **Built-in (System) Node Taints in Kubernetes**
* Kubernetes automatically adds several **system taints** to nodes to reflect their health, state, or role.
* These taints prevent new pods from being scheduled — or evict existing pods — when a node becomes unhealthy, under pressure, or temporarily unschedulable.

| **Taint Key**                                    | **Effect**   | **Added When**                                       | **Description / Use Case**                                                                                                                                                |
| ------------------------------------------------ | ------------ | ---------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `node.kubernetes.io/not-ready`                   | `NoExecute`  | Node status = `NotReady`                             | Node is offline, disconnected, or the Kubelet is unresponsive. All running pods without a matching toleration are evicted immediately.                                    |
| `node.kubernetes.io/unreachable`                 | `NoExecute`  | Node unreachable (API server lost contact)           | Similar to `not-ready`, but triggered when the node can’t be reached due to network issues. Non-tolerant pods are evicted after a short grace period (default 5 minutes). |
| `node.kubernetes.io/memory-pressure`             | `NoSchedule` | Node low on memory                                   | Scheduler prevents new pods without toleration from being scheduled on the node. Used to protect node stability.                                                          |
| `node.kubernetes.io/disk-pressure`               | `NoSchedule` | Node low on disk space                               | Prevents new pods from being scheduled to avoid disk exhaustion.                                                                                                          |
| `node.kubernetes.io/pid-pressure`                | `NoSchedule` | Node is running out of available PIDs (process IDs)  | Prevents overloading nodes when process table is near exhaustion.                                                                                                         |
| `node.kubernetes.io/network-unavailable`         | `NoSchedule` | Node’s network isn’t configured or unavailable       | Common on newly joined nodes until the network plugin (CNI) is ready. Prevents scheduling pods before the network is functional.                                          |
| `node.kubernetes.io/unschedulable`               | `NoSchedule` | Node marked unschedulable (via `kubectl cordon`)     | Automatically tainted when a node is cordoned for maintenance. Scheduler won’t place new pods here.                                                                       |
| `node.cloudprovider.kubernetes.io/uninitialized` | `NoSchedule` | Node not initialized by the cloud controller manager | Used in cloud clusters (AWS, GCP, Azure). Prevents scheduling until the cloud controller sets node metadata.                                                              |
| `node.kubernetes.io/taint` (custom)              | Varies       | You can define your own                              | Used for user-defined taints (e.g., `dedicated=db`, `gpu=true`, `team=devops`).                                                                                           |

---

## **Master / Control Plane Taints**

By default, **control-plane nodes** (previously “masters”) are tainted to avoid scheduling user workloads there.

| **Taint Key**                           | **Effect**   | **Purpose**                                             |
| --------------------------------------- | ------------ | ------------------------------------------------------- |
| `node-role.kubernetes.io/control-plane` | `NoSchedule` | Prevents workloads from running on control-plane nodes. |
| `node-role.kubernetes.io/master`        | `NoSchedule` | Legacy taint (pre-v1.24) — same purpose as above.       |

If you want to schedule pods on master/control-plane nodes (e.g., for single-node clusters like Minikube or kubeadm labs), you can **tolerate** these taints:

```yaml
tolerations:
- key: "node-role.kubernetes.io/control-plane"
  effect: "NoSchedule"
```

or remove the taint:

```bash
kubectl taint nodes <node-name> node-role.kubernetes.io/control-plane:NoSchedule-
```

---

## **Eviction and Recovery Behavior**

| **Scenario**             | **Taint Applied**                       | **Effect**   | **Pod Behavior**                                                              |
| ------------------------ | --------------------------------------- | ------------ | ----------------------------------------------------------------------------- |
| Node loses heartbeat     | `node.kubernetes.io/unreachable`        | `NoExecute`  | Pods without matching toleration evicted after 5 minutes.                     |
| Node out of memory       | `node.kubernetes.io/memory-pressure`    | `NoSchedule` | New pods won’t be placed; existing pods stay unless evicted by the kubelet.   |
| Node under disk pressure | `node.kubernetes.io/disk-pressure`      | `NoSchedule` | Prevents scheduling new pods; existing pods may be evicted if space critical. |
| Node cordoned            | `node.kubernetes.io/unschedulable`      | `NoSchedule` | No new pods are scheduled until uncordoned.                                   |
| Control-plane node       | `node-role.kubernetes.io/control-plane` | `NoSchedule` | Protects control-plane stability.                                             |

---

## **Best Practices When Dealing with Built-in Taints**

1. **Don’t remove them lightly** — These taints protect cluster stability.
2. **Use tolerations cautiously** — Only allow specific critical pods to tolerate pressure-related taints (e.g., DaemonSets or system monitoring agents).
3. **Understand eviction** — Tolerating a `NoExecute` taint can keep pods alive during temporary outages, but you can control how long via:

   ```yaml
   tolerationSeconds: 300
   ```
4. **Monitor node taints regularly**:

   ```bash
   kubectl get nodes -o custom-columns=NODE:.metadata.name,TAINTS:.spec.taints
   ```
5. **For single-node clusters (like labs)** — You can remove the control-plane taint to run everything on one node.
