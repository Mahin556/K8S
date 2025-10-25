* Pass a custom config file:
```bash
kind create cluster --config kind-example-config.yaml
```

* Multi-node cluster (1 control-plane + 2 workers):
```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
- role: worker
- role: worker
```


* HA cluster (3 control-plane + 3 workers):
```yaml
nodes:
- role: control-plane
- role: control-plane
- role: control-plane
- role: worker
- role: worker
- role: worker
```


* Port mapping example (map node port 80 to host port 80):
```bash
nodes:
- role: control-plane
  extraPortMappings:
  - containerPort: 80
    hostPort: 80
    listenAddress: "0.0.0.0"
    protocol: udp
```


* Set Kubernetes version explicitly by node image:
```bash
nodes:
- role: control-plane
  image: kindest/node:v1.16.4@sha256:<sha256sum>
- role: worker
  image: kindest/node:v1.16.4@sha256:<sha256sum>
```


* Enable feature gates:
```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
featureGates:
  FeatureGateName: true
```


* Proxy Settings
• Environment variables supported:
```bash
HTTP_PROXY / http_proxy
HTTPS_PROXY / https_proxy
NO_PROXY / no_proxy
• kind appends internal Kubernetes addresses to NO_PROXY automatically.
```


* Exporting Cluster Logs(Export logs from default cluster)
```bash
kind export logs
```
--> Saves to /tmp/<random_id>


* Export logs to a specific directory:
```bash
kind export logs ./somedir
```
Logs include:
    Docker info
    Node logs (journal, kubelet, pods, etc.)
    Kubernetes version info


* **Cluster-wide options**

  * `name` → custom cluster name
  * `featureGates` → enable/disable Kubernetes features
  * `runtimeConfig` → enable/disable alpha/beta APIs

* **Networking options**

  * `ipFamily: ipv4 | ipv6 | dual`
  * `apiServerAddress`, `apiServerPort`
  * `podSubnet`, `serviceSubnet`
  * `disableDefaultCNI: true` → for using Calico, Cilium, etc.
  * `kubeProxyMode: iptables | ipvs | nftables | none`

* **Nodes**

  * `role: control-plane | worker`
  * `image: kindest/node:<k8s-version>@sha256:...`
  * `extraMounts` → map host directories into nodes
  * `extraPortMappings` → expose node ports to host
  * `labels` → useful for scheduling
  * `kubeadmConfigPatches` → advanced customization of kubeadm

* **Create a named cluster**

```yaml
# cluster-name.yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
name: demo-cluster
```

```bash
kind create cluster --config cluster-name.yaml
kind get clusters
```

---

* **Enable a feature gate**

```yaml
# feature-gates.yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
featureGates:
  "CSIMigration": true
```

```bash
kind create cluster --config feature-gates.yaml
kubectl get nodes -o wide
```

---

* **Custom networking**

```yaml
# networking.yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
networking:
  ipFamily: dual
  podSubnet: "10.244.0.0/16"
  serviceSubnet: "10.96.0.0/12"
  kubeProxyMode: "ipvs"
```

```bash
kind create cluster --config networking.yaml
kubectl get pods -A -o wide
```

---

* **Multi-node cluster with labels**
```yaml
# multinode.yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
- role: worker
  labels:
    tier: frontend
- role: worker
  labels:
    tier: backend
```

```bash
kind create cluster --config multinode.yaml
kubectl get nodes --show-labels
```

---

* **Expose a port from cluster to host**

```yaml
# port-mapping.yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  extraPortMappings:
  - containerPort: 30950
    hostPort: 8080
```

Deploy a NodePort service:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: echo
  labels:
    app: echo
spec:
  containers:
  - name: echo
    image: hashicorp/http-echo:0.2.3
    args: ["-text=Hello from kind!"]
    ports:
    - containerPort: 5678
---
apiVersion: v1
kind: Service
metadata:
  name: echo
spec:
  type: NodePort
  selector:
    app: echo
  ports:
  - port: 5678
    nodePort: 30950
```

```bash
kubectl apply -f echo.yaml
curl http://127.0.0.1:8080
```

---

* **Advanced kubeadm patches**

```yaml
# kubeadm-patch.yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  kubeadmConfigPatches:
  - |
    kind: InitConfiguration
    nodeRegistration:
      kubeletExtraArgs:
        node-labels: "special=true"
```

```bash
kind create cluster --config kubeadm-patch.yaml
kubectl get nodes --show-labels
```

