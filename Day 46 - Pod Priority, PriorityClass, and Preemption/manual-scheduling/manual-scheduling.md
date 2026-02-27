![Alt text](/images/15a.png)

The **Kubernetes Scheduler** is responsible for **automatically placing pods** on available worker nodes based on factors like:  
- **Resource availability** (CPU, memory).  
- **nodeSelectors**(basic)
- **Taints and tolerations** (node restrictions, discussed in Day 16).  
- **Affinity and anti-affinity rules** (Discussed in Day 17).  

However, **we can bypass the scheduler and manually assign pods to nodes**  


## **Manual Scheduling**  
* Manual scheduling means **explicitly assigning a pod to a node** using the `nodeName` field in the pod’s YAML manifest. This completely **bypasses the Kubernetes scheduler**.
- **`nodeName` Field**: Use this field in the pod specification to specify the node where the pod should run.
- **No Scheduler Involvement**: When `nodeName` is specified, the scheduler bypasses the pod, and it’s directly assigned to the given node.


### **Why is Manual Scheduling Required?**  
- **Troubleshooting & Debugging:** Helps diagnose scheduling issues by placing a pod on a specific node.  
- **Testing Node-Specific Workloads:** Ensures an application runs on a specific node (e.g., a database pod requiring an SSD).  
- **Kubernetes Scheduler Is Disabled:** If the scheduler is down, you can manually schedule pods as a fallback.  


### **How is Manual Scheduling Useful?**  
- Guarantees that a pod runs on a particular node.  
- Useful when a workload requires **special hardware** or **node-specific configurations**.  
- Helps **troubleshoot** why a pod isn't scheduled automatically.

### **Demonstration: Assigning a Pod to a Node**  

#### **Step 1: List Available Nodes**  
```sh
kubectl get nodes -owide
```

#### **Step 2: Create a Pod and Assign It to a Specific Node**  

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-manual
spec:
  nodeName: my-second-cluster-worker2  # Assign pod to a specific worker node
  containers:
    - name: nginx
      image: nginx
```

#### **Step 3: Apply the Manifest**
```sh
kubectl apply -f nginx-manual.yaml
```

#### **Step 4: Verify the Pod's Node Assignment**  
```sh
kubectl get pods -o wide
kubectl get pods <pod_name> -o yaml | grep -i nodename
```

#### **Step 5: What Happens If the Node Does Not Exist?** 

When a Pod specifies a wrong or non-existent `nodeName`, Kubernetes cannot schedule the Pod and it remains in the `Pending` state. Over time, due to resource management and cluster policies, Kubernetes deletes the Pod to avoid resource wastage and maintain cluster efficiency.

**Note:** Kubernetes has a default **garbage collection mechanism** that removes stuck or unschedulable Pods after a certain period.

### **Running a Pod on the Control Plane**  
By default, workloads are placed on **worker nodes**. However, you can manually schedule a pod on the **control-plane node**.

Modify the `nodeName` field:
```yaml
spec:
  nodeName: my-second-cluster-control-plane
```

Apply the updated YAML and verify that the **control plane node is running the pod**.


### **How Can Control Plane Run Workloads?**  
- We know that the **scheduler is bypassed** when performing manual scheduling. This is why, even though the control-plane node has a **taint** that **prevents** workloads from running unless they have a matching **toleration**, we were still able to manually assign a pod to the control-plane node.

- The **kubelet** is also installed on control plane nodes, enabling them to run both **static pods and manually scheduled pods**. This is why control plane nodes can execute pods even though scheduling is typically reserved for worker nodes. Additionally, the **kube-proxy** is also running on control plane nodes, facilitating network communication and load balancing for the pods, just as it does on worker nodes.

---

## **Summary**  

- **Manual Scheduling** allows us to assign pods to specific nodes using `nodeName`.  


