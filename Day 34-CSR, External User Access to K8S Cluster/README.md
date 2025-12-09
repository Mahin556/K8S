### References:-
- [Day 34: TLS in Kubernetes MASTERCLASS | PART 4 | CSR, External User Access to K8S Clustero](https://www.youtube.com/watch?v=RZ9O5JJeq9k&ab_channel=CloudWithVarJosh)



### **Granting Cluster Access to a New User (Seema) using Certificates and RBAC**

![Alt text](/images/34a.png)

---

**Granting Cluster Access to a New User (Seema) using Certificates and RBAC**
To securely grant a new user like **Seema** access to a Kubernetes cluster, we follow a series of steps involving certificate-based authentication and Role-Based Access Control (RBAC). This ensures Seema can connect and interact with the cluster within a defined scope.

---

**Step 1: Seema Generates a Private Key**

```bash
openssl genrsa -out seema.key 2048
```

This generates a 2048-bit RSA **private key**, saved to `seema.key`. This key will be used to generate a certificate signing request (CSR) and later to authenticate to the Kubernetes cluster. It must remain **private and secure**.

---

**Step 2: Seema Generates a Certificate Signing Request (CSR)**

```bash
openssl req -new -key seema.key -out seema.csr -subj "/CN=seema"
```

Seema uses her private key to create a **CSR**. The `-subj "/CN=seema"` sets the **Common Name (CN)** to `seema`, which becomes her Kubernetes username. The generated CSR contains her public key and identity, and will be signed by a Kubernetes cluster admin.

---
**Verify CSR**
https://certlogik.com/decoder/

```bash
openssl req -in seema.csr -noout -text
```

----

**Step 3: Seema Shares the CSR with the Kubernetes Admin**

```bash
cat seema.csr | base64 | tr -d "\n"
```

The CSR must be **base64-encoded** to embed it into a Kubernetes object. This command converts the CSR into a single-line base64 string, stripping newlines with `tr -d "\n"`—a necessary step for YAML formatting.


---

**Step 4: Kubernetes Admin Creates the CSR Object in Kubernetes**

```yaml
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: seema
spec:
  request: <BASE64_ENCODED_CSR>
  signerName: kubernetes.io/kube-apiserver-client
  expirationSeconds: 7776000
  usages:
  - client auth
```

The admin creates a Kubernetes `CertificateSigningRequest` object.

* `request` is the base64-encoded CSR.
* `signerName: kubernetes.io/kube-apiserver-client` instructs Kubernetes to treat this as a **client authentication** request.
* `usages` defines that this certificate will be used for **client authentication**, not server TLS or other use cases.
* `expirationSeconds` sets the certificate’s validity to **90 days** (7776000 seconds).

---

**Step 5: Kubernetes Admin Approves the CSR**

```bash
kubectl certificate approve seema
```

This command **approves and signs** the certificate request. Kubernetes issues a certificate for Seema, valid per the defined usage and expiration settings.

---

**Step 6: Admin Retrieves and Shares the Signed Certificate**

```bash
kubectl get csr seema -o jsonpath='{.status.certificate}' | base64 -d > seema.crt
```

The admin retrieves the **signed certificate** from the CSR’s status, decodes it from base64, and saves it as `seema.crt`. This certificate, along with `seema.key`, is sent back to Seema for kubeconfig configuration.

---
```bash
kubectl config view -o jsonpath='{.clusters[0].cluster.certificate-authority-data}' | base64 --decode > ca.crt
```
Go to:- https://certlogik.com/decoder/
  * kubernetes CA
  * Root CA

---

**Step 7: Seema Configures Her `kubeconfig` with Credentials and Cluster Info**

```bash
kubectl config set-credentials seema \
  --client-certificate=seema.crt \
  --client-key=seema.key \
  --certificate-authority=ca.crt \
  --embed-certs=true

kubectl config set-cluster kind-my-second-cluster \
  --server=https://127.0.0.1:59599 \
  --certificate-authority=ca.crt \
  --embed-certs=true \
  --kubeconfig=~/.kube/config

kubectl config set-context seema@kind-my-second-cluster-context \
  --cluster=kind-my-second-cluster \
  --user=seema \
  --namespace=default
```

These commands configure the **user credentials**, **cluster endpoint**, and **context** in Seema’s `kubeconfig`:

* The first command tells kubectl how to authenticate Seema using her certificate/key.
* The second registers the cluster endpoint using the correct CA.
* The third defines a context associating the user, cluster, and default namespace.

---

**Step 8: Admin Creates a Role and RoleBinding for Seema**

```yaml
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: default
  name: seema-role
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "delete"]
```

```bash
kubectl create rolebinding seema-binding \
  --role=seema-role \
  --user=seema \
  --namespace=default
```

The **Role** allows Seema to **get**, **list**, and **delete** pods in the `default` namespace.
The **RoleBinding** assigns this Role to Seema's username (`CN=seema`), authorizing her actions.

---

**Step 9: Admin Verifies Authorization with `can-i`**

```bash
kubectl auth can-i delete pods --namespace=default --as=seema
```

This command is run by the **admin** to simulate whether Seema is allowed to **delete pods** in the `default` namespace.

* `--as=seema` impersonates Seema’s user identity.
* This confirms that the **RBAC permissions** are set correctly before Seema starts using the cluster.

---

**Step 10: Seema Switches to Her Configured Context**

```bash
kubectl config use-context seema@kind-my-second-cluster-context
```

This sets Seema's **active context** to the one defined earlier, allowing `kubectl` to use her certificate and connect to the right cluster/namespace.

---

**Optional: Use REST API or Alternate `kubeconfig` Files**

```bash
curl https://<API-SERVER-IP>:<PORT>/api/v1/namespaces/default/pods \
  --cacert ca.crt --cert seema.crt --key seema.key
```

*Seema can authenticate with the cluster directly via API using her certificate.*

```bash
kubectl get pods --kubeconfig=myconfig.yaml
```

*She can manage multiple clusters by specifying alternate kubeconfig files.*

---

**Step 11: Check Certificate Expiry**

```bash
openssl x509 -noout -dates -in seema.crt
```

This displays the `notBefore` and `notAfter` dates for the certificate, helping Seema monitor its expiration.

---

### **Conclusion**

We’ve just completed a full, practical flow to securely grant a user access to a Kubernetes cluster:
from generating keys and certificates, to configuring kubeconfig, and finally applying RBAC permissions.
This knowledge is foundational for any admin responsible for managing access in production clusters.

---

### References

* [Kubernetes: Authenticating](https://kubernetes.io/docs/reference/access-authn-authz/authentication/)
* [Kubernetes: Using RBAC Authorization](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)
* [CertificateSigningRequest API](https://kubernetes.io/docs/reference/kubernetes-api/authentication-resources/certificate-signing-request-v1/)

---
