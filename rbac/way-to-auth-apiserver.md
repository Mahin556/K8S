* Authentication is file line of defence in any system as it is in k8s
* K8S support several authentication mechanisms (x.509 certiticates, bearer tokens and external Identity providers like OpenID Connect)

```bash
[Client Request]
      ↓
[TLS Handshake]  <-- verifies server identity + optionally client cert
      ↓
[Authentication Modules]  <-- certificates, tokens, webhook, OIDC, etc.
      ↓
[Identity Extracted]  <-- userName, groups, uid
      ↓
[Authorization (RBAC/ABAC/Webhook)]
```

---

# **How the API Server Authenticates – All Methods Explained**

Kubernetes supports these authentication mechanisms **simultaneously**:

| Authentication Method      | Input Expected    | Used For                  | Example Who Uses |
| -------------------------- | ----------------- | ------------------------- | ---------------- |
| **Client Certificates**    | x509 TLS cert     | Human users, admins       | kubectl, kubelet |
| **Static Token File**      | Bearer token      | Simple clusters/testing   | Older clusters   |
| **Bootstrap Tokens**       | Token + signature | Joining worker nodes      | kubeadm          |
| **Service Account Tokens** | JWT signed token  | Pods accessing API        | All workloads    |
| **OIDC / External IDP**    | OAuth2 token      | SSO (Google, Azure, Okta) | Enterprise       |
| **Webhook Authentication** | HTTP webhook      | Custom auth logic         | Company IAM      |
| **Proxy Authentication**   | Auth headers      | API gateway/Ingress auth  | Enterprises      |

The API server checks **each method in order** until it finds a valid one.

---

## 1️⃣ **Client Certificate Authentication**

Occurs during the **TLS handshake**, before HTTP request.

* Kubernetes trusts a **client certificate CA** (like /etc/kubernetes/pki/ca.crt)
* If the client presents a certificate signed by that CA → it is considered authenticated
* Username = CN (Common Name)
* Groups = O (Organization)

Example:

```
CN=mahin
O=dev-team
```

API server sets:

```
user = "mahin"
groups = ["dev-team"]
```

Used for:
✔ kubelet
✔ kubectl configured with client certs
✔ admin users

---

## 2️⃣ **Static Bearer Token File**

API server loads a CSV file containing:

```
token,username,uid,"group1,group2"
```

Then the user sends:

```
Authorization: Bearer <token>
```

Used mainly in **old clusters**.

---

## 3️⃣ **Bootstrap Tokens (kubeadm)**

Used only for **cluster node join**.

Token format:

```
abcdef.0123456789abcdef
```

Sent in:

```
Authorization: Bearer <bootstrap-token>
```

API server verifies using **bootstrap-token secret**.

---

## 4️⃣ **Service Account Tokens (JWT)**

Every Pod automatically gets a **JWT token** signed by:

```
/var/run/secrets/kubernetes.io/serviceaccount/token
```

Contains:

* sub: system:serviceaccount:<ns>:<name>
* iss: kubernetes/serviceaccount

API server validates signature using:

```
--service-account-key-file
--service-account-issuer
```

Used for:
✔ Pods calling API
✔ Controllers
✔ DaemonSets

---

## 5️⃣ **OpenID Connect (OIDC) Authentication**

Integrates with external identity provider:

* Google
* Azure AD
* Okta
* Keycloak

Client sends:

```
Authorization: Bearer <OIDC_JWT>
```

API server validates:

* Issuer URL
* TLS certificate
* Client ID
* Signing keys (JWKS)

Used for:
✔ SSO
✔ Enterprise Identity

---

## 6️⃣ **Webhook Token Authentication**

API server forwards the request to your custom auth server:

```
POST /authenticate
```

Your server responds:

```
{
  "apiVersion": "authentication.k8s.io/v1",
  "status": {
     "authenticated": true,
     "user": { "username": "mahin", "groups": ["dev"] }
  }
}
```

Used for integrating with:
✔ LDAP
✔ Active Directory
✔ Custom IAM

---

## 7️⃣ **Proxy Authentication**

Useful when API server sits behind:

* Ingress
* Reverse proxy
* Service mesh gateway

Proxy adds headers:

```
X-Remote-User: mahin
X-Remote-Group: dev-team
```

Only allowed when:

```
--requestheader-allowed-names
--requestheader-group-headers
--requestheader-username-headers
```

are configured.

---

#### **Kubernetes Authorization Methods (Detailed Explanation)**

* Once authentication verifies **WHO** is making the request, Kubernetes must determine **WHAT they are allowed to do**. This is where **authorization** begins.
* Kubernetes uses multiple authorization modes, each serving a different purpose and offering different levels of control.
* Below are the three main authorization mechanisms:

##### **1️⃣ Node Authorization**
**Special-purpose authorization mode exclusively for kubelets.**
* Node Authorization is an authorization mode designed specifically for **kubelets**, the agent running on every Kubernetes node.
* It protects the API server by ensuring that kubelets can only:
    * Read the **Pods assigned to their node**
    * Access **Secrets** or **ConfigMaps** needed by those Pods
    * Read/modify node-specific resources
    * Adjust status fields of Pods running on their node
* To enforce the **Principle of Least Privilege** for kubelets.
* Without Node Authorization, kubelets would have too much power and could:
    * Read secrets for all namespaces
    * Access pods on other nodes
    * Modify cluster-wide resources
* Node authorization ensures each node can only manage **its own resources**, nothing more.
* In kube-apiserver:
    ```bash
    --authorization-mode=Node,RBAC
    ```

##### **2️⃣ Attribute-Based Access Control (ABAC)**
**Older, JSON-based authorization method using request attributes.**
* You define policies in a **JSON file**, and each policy checks attributes such as:
    * user
    * group
    * namespace
    * resource
    * verb
* Example ABAC policy:
    ```json
    {
    "apiVersion": "abac.authorization.kubernetes.io/v1beta1",
    "kind": "Policy",
    "user": "dev-user",
    "namespace": "dev",
    "resource": "pods",
    "readonly": true
    }
    ```
* Granular, flexible
* Can enforce very specific rules using many attributes
* Hard to maintain in large clusters
* Policies live in a JSON file on the API server → not dynamic
* Requires API server restart when modified
* Not GitOps-friendly
* Not recommended for production
* ABAC is still supported but is **rarely used today**. RBAC has fully replaced it in modern environments.


##### **3️⃣ Role-Based Access Control (RBAC)**
**Recommended, modern, most widely used authorization method in Kubernetes.**
* RBAC assigns permissions to:
    * **Roles** (namespace-scoped)
    * **ClusterRoles** (cluster-scoped)
* And binds them to:
    * Users
    * Groups
    * Service Accounts
* A Role defines WHAT you can do.
* A RoleBinding determines WHO can do it.
* **Role**
  * Namespace-specific permissions:
    ```yaml
    kind: Role
    rules:
    - apiGroups: [""]
    resources: ["pods"]
    verbs: ["get", "list"]
    ```
* **ClusterRole**
  * Cluster-wide permissions:
    ```yaml
    kind: ClusterRole
    rules:
    - apiGroups: [""]
    resources: ["nodes"]
    verbs: ["get", "list"]
    ```

* **RoleBinding / ClusterRoleBinding**
* Connects users/groups/service accounts to roles.

* Why RBAC is recommended
    * Easy to understand
    * Easy to manage
    * Easy to audit
    * Dynamic: changes take effect immediately
    * Works perfectly with GitOps
    * Supported by Kubernetes dashboard, tools, and cloud platforms

* Where it is used
    * Production clusters
    * Dev/stage/test clusters
    * Enterprise Kubernetes
    * Cloud providers: EKS / AKS / GKE

---

