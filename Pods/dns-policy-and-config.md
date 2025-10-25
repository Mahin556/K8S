### 🧩 **1. `dnsPolicy` – DNS Behavior Policy**

`dnsPolicy` defines **how the Pod’s DNS settings are configured**. It determines whether the Pod uses:

* The **cluster’s DNS service (CoreDNS)**
* The **node’s default DNS configuration**
* Or **custom settings**

There are **4 main types** of `dnsPolicy`:

---

#### **a) ClusterFirst (default for most Pods)**

* This is the **default policy** when Pods are running inside a Kubernetes cluster.
* DNS queries for **Services inside the cluster** (like `my-service.default.svc.cluster.local`) are automatically resolved by **CoreDNS**.
* For **external domains** (like `google.com`), the request is forwarded to the upstream DNS servers.

**Example:**

```yaml
dnsPolicy: ClusterFirst
```

**When used:**
✅ For almost all Pods running inside the cluster that need to access both internal and external services.

---

#### **b) Default**

* The Pod inherits the **DNS configuration from the node’s /etc/resolv.conf** (the host machine).
* No automatic integration with cluster DNS.

**Example:**

```yaml
dnsPolicy: Default
```

**When used:**
✅ When Pods need **exactly the same DNS behavior as the host**.
⚠️ Not recommended for normal Pods, as it bypasses CoreDNS and cluster service discovery.

---

#### **c) None**

* This disables all default DNS configuration.
* You must specify **dnsConfig** manually to define custom nameservers, searches, and options.
* Gives you full control over DNS behavior.

**Example:**

```yaml
dnsPolicy: None
dnsConfig:
  nameservers:
    - 1.1.1.1
    - 8.8.8.8
  searches:
    - mycorp.local
    - example.com
  options:
    - name: ndots
      value: "2"
```

**When used:**
✅ When you want to fully control DNS settings (for testing, custom network setups, or isolated environments).

---

#### **d) ClusterFirstWithHostNet**

* Used when Pods run with **hostNetwork: true**, meaning they share the node’s network namespace.
* Ensures DNS queries for cluster services are still sent to CoreDNS (not host DNS).

**Example:**

```yaml
hostNetwork: true
dnsPolicy: ClusterFirstWithHostNet
```

**When used:**
✅ For Pods using host networking but still requiring access to cluster DNS.

---

### 🧠 **2. `dnsConfig` – Custom DNS Settings**

`dnsConfig` allows you to **add or override specific DNS settings** within the Pod, regardless of the policy (in some cases).
It’s used for **customizing name servers, search domains, and resolver options**.

---

#### **Fields of `dnsConfig`:**

**a) `nameservers`**

* List of custom DNS server IPs.
* Maximum of 3 IPs allowed.
* Example:

  ```yaml
  nameservers:
    - 8.8.8.8
    - 1.1.1.1
  ```

---

**b) `searches`**

* Specifies search domains for DNS lookups.
* When you run `ping myservice`, Kubernetes will try:

  ```
  myservice.<namespace>.svc.cluster.local
  myservice.svc.cluster.local
  myservice.cluster.local
  myservice.mydomain.local
  myservice
  ```

  Example:

  ```yaml
  searches:
    - mydomain.local
    - example.com
  ```

---

**c) `options`**

* Adds resolver options from `/etc/resolv.conf`.
* Common examples:

  * `ndots`: How many dots a name must have before trying absolute resolution.
    Example: `ndots: 2`
  * `timeout`: How long to wait for a DNS query response.
  * `attempts`: Number of retries before giving up.

  Example:

  ```yaml
  options:
    - name: ndots
      value: "2"
    - name: timeout
      value: "3"
  ```

---

### 🧾 **3. Example – Full DNS Customization**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: dns-example
spec:
  containers:
  - name: test
    image: busybox
    command: ["sh", "-c", "cat /etc/resolv.conf; sleep 3600"]
  dnsPolicy: None
  dnsConfig:
    nameservers:
      - 8.8.8.8
      - 1.1.1.1
    searches:
      - mydomain.local
      - example.com
    options:
      - name: ndots
        value: "2"
      - name: timeout
        value: "3"
```

If you run into this Pod and check:

```bash
kubectl exec -it dns-example -- cat /etc/resolv.conf
```

You’ll see:

```
nameserver 8.8.8.8
nameserver 1.1.1.1
search mydomain.local example.com
options ndots:2 timeout:3
```

---

### ⚙️ **4. Summary Table**

| dnsPolicy               | Uses Cluster DNS | Uses Node DNS   | Needs dnsConfig | Typical Use        |
| ----------------------- | ---------------- | --------------- | --------------- | ------------------ |
| ClusterFirst            | ✅                | ❌               | Optional        | Default for Pods   |
| Default                 | ❌                | ✅               | Optional        | Match node DNS     |
| None                    | ❌                | ❌               | ✅ Required      | Fully custom setup |
| ClusterFirstWithHostNet | ✅                | ✅ (hostNetwork) | Optional        | Host network Pods  |

---
