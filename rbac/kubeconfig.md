### References:- 
- https://yuminlee2.medium.com/kubernetes-kubeconfig-file-4aabe3b04ade
- https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/
- https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/
- https://spacelift.io/blog/kubeconfig
- https://faun.pub/give-users-and-groups-access-to-kubernetes-cluster-using-rbac-b614b6c0b383
- https://devopscube.com/kubernetes-kubeconfig-file/
- https://freedium-mirror.cfd/https://harsh05.medium.com/kubernetes-service-accounts-simplifying-authentication-and-authorization-07d5c50d2e77

* The kubeconfig file is used to access Kubernetes clusters, mainly by the kubectl command-line tool.
* Kubernetes components like `kubelet`, `kube-controller-manager`, or `kubectl` use the kubeconfig file to interact with the Kubernetes API. Usually, the `kubectl` or oc commands use the kubeconfig file.
* It contains three major sections:
  * clusters → cluster endpoint and certificate details
  * users → authentication credentials
  * contexts → combination of a cluster + user + namespace
* The default location of the kubeconfig file is: `$HOME/.kube/config
`
* In `kubeconfig` PEM format certiticates should be stored in a BASE64 format.
* You can specify a custom kubeconfig file using:
```bash
KUBECONFIG=<path>
--kubeconfig <path>

KUBECONFIG=/my/kubeconfig kubectl get pods
kubectl get pods --kubeconfig /my/kubeconfig

export KUBECONFIG=/my/kubeconfig
kubectl get pods

```

* `kubeconfig.yaml`
```yaml
apiVersion: v1
kind: Config
clusters: #List
- name: production #Identifier
  cluster:
    server: https://production.example.com #API server endpoint
    certificate-authority: /path/to/production/ca.crt #CA certificate to trust the cluster
- name: development
  cluster:
    server: https://development.example.com
    certificate-authority: /path/to/development/ca.crt
- name: test
  cluster:
    server: https://test.example.com
    certificate-authority: /path/to/test/ca.crt
contexts: #A context ties everything together., A context = cluster + user + namespace.
- name: production
  context:
    cluster: production
    user: prod-user
- name: development
  context:
    cluster: development
    user: dev-user
- name: test
  context:
    cluster: test
    user: test-user
current-context: production #This value tells kubectl which context to use by default.
users: #This section stores the authentication credentials needed to log in to a cluster.
- name: prod-user #user identity (prod-user, dev-user, test-user)
  user:
    client-certificate: /path/to/production/prod-user.crt #user certificate file
    client-key: /path/to/production/prod-user.key #user private key
- name: dev-user
  user:
    client-certificate: /path/to/development/dev-user.crt
    client-key: /path/to/development/dev-user.key
- name: test-user
  user:
    client-certificate: /path/to/test/test-user.crt
    client-key: /path/to/test/test-user.key
```
```yaml
apiVersion: v1
clusters:
- cluster:
   certificate-authority-data: LS0tL..
   server: https://127.0.0.1:64914
   name: kind-kind
- cluster:
   certificate-authority-data: LS0tLS1C..
   server: https://127.0.0.1:60963
   name: kind-ope
- cluster:
   certificate-authority: /Users/flaviuscdinu/.minikube/ca.crt
   extensions:
   - extension:
       last-update: Thu, 16 Feb 2023 14:50:26 EET
       provider: minikube.sigs.k8s.io
       version: v1.28.0
     name: cluster_info
   server: https://127.0.0.1:49731
   name: minikube
contexts:
- context:
   cluster: kind-kind
   user: kind-kind
 name: kind-kind
- context:
   cluster: kind-ope
   user: kind-ope
 name: kind-ope
- context:
   cluster: minikube
   extensions:
   - extension:
       last-update: Thu, 16 Feb 2023 14:50:26 EET
       provider: minikube.sigs.k8s.io
       version: v1.28.0
     name: context_info
   namespace: default
   user: minikube
 name: minikube
current-context: minikube
kind: Config
preferences: {}
users:
- name: kind-kind
 user:
   client-certificate-data: LS0t…
   client-key-data: LS0t…
- name: kind-ope
 user:
   client-certificate-data: LS0t..
   client-key-data: LS0t…
- name: minikube
 user:
   client-certificate: /Users/flaviuscdinu/.minikube/profiles/minikube/client.crt
   client-key: /Users/flaviuscdinu/.minikube/profiles/minikube/client.key
```

* Without a kubeconfig, you would have to connect manually using long flags.
```bash
curl https://your-api-server.com/api/v1/namespaces/default/pods \
  --cacert ca.crt \
  --cert user.crt \
  --key user.key
```
```bash
kubectl get pods \
  --server=https://your-api-server.com \
  --certificate-authority=ca.crt \
  --client-certificate=user.crt \
  --client-key=user.key
```

* KUBECONFIG SOLVES THIS PROBLEM
  * Instead of typing all those arguments:
    * API server URL
    * CA certificate
    * user certificate
    * user key
    * namespace
    * cluster name
    * You put all this inside the kubeconfig file just once.
    * After that you simply run: `kubectl get pods`
  * kubectl automatically:
    * reads the kubeconfig file
    * authenticates you with the correct certificate
    * connects to the right cluster
    * uses the default namespace
    * uses the correct API server
    * No need to type credentials every time.
  * stores all cluster login details safely
  * avoids writing long flags
  * allows switching between clusters easily
  * allows switching between users easily
  * sets default namespaces per environment
  * supports multiple environments (prod/dev/test) in one file
  * provides secure certificate-based authentication
  * And kubectl automatically reads it.

* Client Certificate + Client Key
```bash
openssl genrsa -out dev-user.key 2048

openssl req -new -key dev-user.key -out dev-user.csr \
  -subj "/CN=dev-user/O=developers"

openssl x509 -req -in dev-user.csr \
  -CA /etc/kubernetes/pki/ca.crt \
  -CAkey /etc/kubernetes/pki/ca.key \
  -CAcreateserial \
  -out dev-user.crt -days 365

kubectl config set-credentials dev-user \
  --client-certificate=dev-user.crt \
  --client-key=dev-user.key

kubectl --user=dev-user get pods
```

* Username + Password (STATIC BASIC AUTH)
```bash
cat << EOF > /etc/kubernetes/password.csv
password123,dev-user,uid123,"developers"
EOF

#Configure API server
--basic-auth-file=/etc/kubernetes/password.csv
#Restart API server.

kubectl config set-credentials dev-user \
  --username=dev-user --password=password123

kubectl config get-users

kubectl --user=dev-user get pods #Works, but completely insecure.
```

* Bearer Token (SERVICE ACCOUNT TOKEN)
```bash
kubectl create serviceaccount myapp-sa

kubectl create token myapp-sa
eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...

curl -H "Authorization: Bearer <TOKEN>" \
  --cacert /etc/kubernetes/pki/ca.crt \
  https://<api-server>/api/v1/pods

kubectl config set-credentials sa-user --token=<TOKEN>

kubectl --user=sa-user get pods
```

* OIDC Authentication (AUTH PROVIDER)
  * Used in enterprise clusters with:
    * Keycloak
    * Google Cloud
    * Azure AD
    * Okta
    * Auth0
  * This method uses OIDC tokens instead of certificates.
```bash
#Using Google Cloud OIDC on Kubernetes
#Configure API server
--oidc-issuer-url=https://accounts.google.com
--oidc-client-id=my-k8s-client
--oidc-username-claim=email
--oidc-groups-claim=groups

#User logs in through OIDC provider
#Example (Google):
gcloud auth login
gcloud container clusters get-credentials mycluster

#Or with Keycloak:
kubectl oidc-login setup
kubectl oidc-login get-token

#kubeconfig stores OIDC provider section
cat << EOF > ~/.kube/config
users:
- name: oidc-user
  user:
    auth-provider:
      name: oidc
      config:
        client-id: my-k8s-client
        client-secret: mysecret
        id-token: eyJhbGciOiJSUzI1N...
        refresh-token: 1//04cZ...
        idp-issuer-url: https://accounts.google.com
EOF
kubectl --user=oidc-user get pods
```
---
* `Kubeconfig best practices
```bash
#Your kubeconfig file holds sensitive details like tokens and certificates, so it should only be accessible to you. Make sure you're the only one who can read or modify it.
chmod 600 ~/.kube/config 

# For multiple configs
find ~/.kube -name "*.config" -exec chmod 600 {} \;

#You can also lock down the entire .kube directory to make sure no one else can read from or write to it.
chmod 700 ~/.kube
chown -R $USER ~/.kube

#Prevent accidental exposure
echo "/.kube/" >> ~/.gitignore
echo "kubeconfig*" >> ~/.gitignore

#Store all files in ~/.kube/
~/.kube/dev
~/.kube/stage
~/.kube/prod

```

---

```bash
kubectl config view

kubectl config view --raw

kubectl config view --kubeconfig=<path_to_kubeconfig>

kubectl config use-context <context_name>

kubectl cluster-info

kubectl cluster-info --kubeconfig=<path_to_kubeconfig>

kubectl config set-cluster production --server=https://1.1.1.1 --certificate-authority=~/.kube/production.ca.crt #Add cluster to the config

kubectl config set-cluster staging --server=https://2.2.2.2 --certificate-authority=~/.kube/staging.ca.crt

#If you’re running a local cluster without TLS, you can disable TLS verification instead of supplying certificate authority data:
kubectl config set-cluster staging --server=https://2.2.2.2 --insecure-skip-tls-verify

kubectl get clusters #List all the clusters from the default config file

kubectl config set-credentials production-admin --token=cfrDHdb2 #Add user

#Token-based auth is the correct method when your user is a service account created within Kubernetes. Use --username and --password instead if you’re using HTTP Basic Auth, or set --client-certificate and --client-key for certificate-based authentication.

kubectl config get-users #list users

kubectl config get-contexts -o=name

kubectl config set-context production --cluster production --user production-admin #set-context for ease

kubectl config get-contexts #list conteext

kubectl config use-context production #change context

kubectl get pods --context production #change context for this command

kubectl config set-context staging --namespace demo-app #default namespce with context

 kubectl config delete-user staging-admin
 kubectl config delete-context staging-context
 kubectl config delete-cluster staging-cluster
```
* Merging a kubeconfig file
```bash
export KUBECONFIG=~/.kube/kubeconfig-1:~/.kube/kubeconfig-2
unset KUBECONFIG

kubectl config view --flatten > ~/.kube/config 
#view → shows combined configuration
#--flatten → removes duplicate certificates and embeds data directly
#output → redirected into a single kubeconfig file
#After this, you’ll have one unified kubeconfig at:
~/.kube/config
#This new file contains all: clusters, contexts, users from all the kubeconfig files you referenced.

#You can merge all three configs into a single file using the following command. Ensure you are running the command from the HOME/ .kube directory.
KUBECONFIG=config:dev_config:test_config kubectl config view --merge --flatten > config.new
mv $HOME/.kube/config $HOME/.kube/config.old
mv $HOME/.kube/config.new $HOME/.kube/config

kubectl config view --minify

```

* Never Share Kubeconfig Files
  Email kubeconfigs
  Upload them to Slack/Teams
  Store them in public S3 buckets
  Keep them in Git repositories
  Accidental exposure of kubeconfigs is one of the most common Kubernetes security mistakes.

* If a Kubeconfig Is Leaked, Act Immediately
    * Delete the certificate signing request (CSR)
    * Revoke or rotate the client certificate
    * Generate a new kubeconfig
    * `kubectl delete secret <sa-secret>`
    * `kubectl create token <sa-name>`
    * For OIDC users
        * Revoke the refresh token
        * Force new login
        * Disable their OIDC session

* Kubeconfig best practices
    * use short lived credentials

---
---

# **Kubeconfig and Security (Full Detailed Explanation)**
Kubeconfig files are **high-risk security assets** because they contain everything required to authenticate to a Kubernetes cluster. Whether you merge multiple kubeconfigs into one or keep them separate, the security risk remains the same:

**Anyone who gains access to your kubeconfig file gains access to your cluster.**
For this reason, kubeconfigs must be protected with the same seriousness as SSH private keys, cloud access keys, or passwords.

# **Treat Kubeconfig Files as Highly Sensitive Credentials**
A kubeconfig may contain:
* Client certificates
* Private keys
* Bearer tokens (service account tokens)
* OIDC ID tokens
* Refresh tokens
* API server endpoints
* Usernames and groups
With these, an attacker can impersonate you and perform *any action* that your credentials allow.

**Never share kubeconfig files with anyone, including teammates, unless absolutely required.**

# **Prevent Accidental Exposure**
### 🔹 DO NOT commit kubeconfigs to Git
Use `.gitignore` to avoid accidental commits:
```
.kube/
*.kubeconfig
*.crt
*.key
```

### 🔹 DO NOT upload to storage or chat apps
Avoid sending kubeconfigs through:
* Slack
* Teams
* WhatsApp
* Email
* Pastebin
* Shared S3 buckets (unless encrypted)

### 🔹 Use strict file permissions
```
chmod 600 ~/.kube/config
```

# **3️⃣ What To Do If a Kubeconfig Is Leaked**
If you suspect accidental exposure:
### ✔ Immediately revoke credentials
For users using client certificates:
```
kubectl delete csr <csr-name>
```
Or rotate the certificate pair.

### ✔ For service accounts
Regenerate token:
```
kubectl delete secret <sa-secret>
kubectl create token <service-account>
```

### ✔ For OIDC users
Revoke the token from the identity provider:
* Keycloak: revoke session
* Okta: revoke refresh token
* Google/AzureAD: disable app session

### ✔ For static tokens
Edit API server token file to remove or rotate it.
**Never continue using a compromised kubeconfig.**

# **4️⃣ Beware of Malicious Kubeconfig Files**
Most people believe kubeconfigs are “just YAML”…
But kubeconfigs can contain **executable commands** via auth plugins.

### ⚠️ Example of a malicious kubeconfig:
```yaml
users:
- name: attacker
  user:
    exec:
      command: /tmp/evil.sh
```

When you run *any* kubectl command:
```
kubectl get pods
```

kubectl will execute the attacker’s script automatically.
The script may:
* Steal your AWS/GCP credentials
* Steal SSH private keys
* Modify system files
* Install malware

**Never use kubeconfig files from untrusted sources.
Always inspect FIRST.**
```
cat suspicious-config.yaml
```
Check especially for:
* `exec:`
* `auth-provider:`
* Embedded tokens
* Strange commands

Treat unknown kubeconfig files **as dangerous as shell scripts**.

# **5️⃣ Rotate Credentials Regularly**
To reduce risk:
* Rotate client certificates regularly
* Rotate service account tokens
* Use short-lived OIDC tokens
* Enable automatic token rotation where supported
* Delete stale kubeconfigs after use
This minimizes the damage in case of theft.

# **6️⃣ Tools to Improve Kubeconfig Security and Management**
Here is the continuation of the line “Tools t…” in your text:
### **✔ kubectx**
Fast context switching
```
kubectl krew install ctx
```
### **✔ kubens**

Fast namespace switching
```
kubectl krew install ns
```

### **✔ kube-ps1**
Displays current context + namespace in your shell prompt.

### **✔ stern / k9s**
Useful while working in multiple environments to prevent mistakes.

### **✔ sops or age**
Encrypt kubeconfigs at rest if storing in GitOps or CI/CD.

### **✔ sealed-secrets or external-secrets**
Do NOT store kubeconfigs directly in Git — use secrets-management tools.

### **✔ vault (HashiCorp Vault)**
Store kubeconfigs or dynamically generate short-lived tokens.

### **✔ aws-vault / gcloud auth / azure identity**
Use short-lived credentials instead of static kubeconfig keys.

---
---

```bash
openssl genrsa -out david.key 2048

openssl req -new -key david.key -subj "/CN=david/O=developer" -out david.csr

cat <<EOF> file.yaml
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: <name>
spec:
  request: <csr-base64>
  signerName: kubernetes.io/kube-apiserver-client
  expirationSeconds: 8640000   # minimum - 600sec/10min
  usages:
  - client auth
EOF

CSR_david=$(cat david.csr | base64 -w 0)

curl https://raw.githubusercontent.com/shamimice03/Kubernetes_RBAC/main/CertificateSigningRequest-Template.yaml | sed "s/<name>/david/ ; s/<csr-base64>/$CSR_david/" > david_csr.yaml

kubectl create -f david_csr.yaml

kubectl get csr

kubectl describe csr david

kubectl certificate approve david

kubectl get csr <CSR-NAME> -o jsonpath='{.status.certificate}' | base64 --decode > <client-name>.crt

kubectl config view --raw -o jsonpath='{..cluster.certificate-authority-data}' | base64 --decode > ca.crt

openssl x509 -in david.crt -text -noout
openssl x509 -in ca.crt -text -noout

cat <<EOF>config
#kubeconfig file template
apiVersion: v1
kind: Config
current-context: <context>
clusters:
- name: <cluster-name>
  cluster:
    certificate-authority-data: <ca.crt>
    server: <cluster-endpoint>
contexts:
- name: <context>
  context:
    cluster: <cluster-name>
    user: <user-name>
    namespace: <namespace>
users:
- name: <user-name>
  user:
    client-certificate-data: <user.crt>
    client-key-data: <user.key>
EOF

CA_CRT=$(cat ca.crt | base64 -w 0)
CONTEXT=$(kubectl config current-context)
CLUSTER_ENDPOINT=$(kubectl config view -o jsonpath='{.clusters[?(@.name=="'"$CONTEXT"'")].cluster.server}')
USER=david
NAMESPACE=production
DAVID_CRT=$(cat david.crt | base64 -w 0)
DAVID_KEY=$(cat david.key | base64 -w 0)

curl https://raw.githubusercontent.com/shamimice03/Kubernetes_RBAC/main/kubeconfig-template.yaml | 
sed "s#<context>#$CONTEXT# ;
s#<cluster-name>#$CONTEXT# ;
s#<ca.crt>#$CA_CRT# ;
s#<cluster-endpoint>#$CLUSTER_ENDPOINT# ;
s#<user-name>#$USER# ;
s#<namespace>#$NAMESPACE# ;
s#<user.crt>#$DAVID_CRT# ; 
s#<user.key>#$DAVID_KEY#" > config

```

```bash
#!/bin/sh

read -p 'Enter the Username : ' name
read -p 'Enter the Group Name : ' group
read -p 'Enter the Namespace Name: ' namespace

export CLIENT=$name
export GROUP=$group
export NAMESPACE=$namespace

echo -e "\nUsername is: ${CLIENT}\nGroup Name is: ${GROUP}\nand Namespace is: ${NAMESPACE}"
echo -e "\nIf you want to proceed with above informaton, type \"yes\" or \"no\": " 
read value

if [ $value == "yes" ]
then
    mkdir ${CLIENT}
    cd ${CLIENT}
    
    #Generate key
    openssl genrsa -out ${CLIENT}.key 2048
    
    #Generate csr
    openssl req -new -key ${CLIENT}.key -subj "/CN=${CLIENT}/O=${GROUP}" -out ${CLIENT}.csr
    
    #CSR to base64
    export CSR_CLIENT=$(cat ${CLIENT}.csr | base64 -w 0)
    
    #Create CSR object file
    curl https://raw.githubusercontent.com/shamimice03/Kubernetes_RBAC/main/CertificateSigningRequest-Template.yaml | sed "s/<name>/${CLIENT}/ ; s/<csr-base64>/${CSR_CLIENT}/" > ${CLIENT}_csr.yaml
   
    #Create CSR object
    kubectl create -f ${CLIENT}_csr.yaml
   
    #Approve CSR 
    kubectl certificate approve ${CLIENT}
    
    #extracting client certificate
    kubectl get csr ${CLIENT} -o jsonpath='{.status.certificate}' | base64 --decode > ${CLIENT}.crt
    
    #CA extraction 
    kubectl config view --raw -o jsonpath='{..cluster.certificate-authority-data}' | base64 --decode > ca.crt
   
    #Set ENV
    export CA_CRT=$(cat ca.crt | base64 -w 0)
    export CONTEXT=$(kubectl config current-context)
    export CLUSTER_ENDPOINT=$(kubectl config view -o jsonpath='{.clusters[?(@.name=="'"$CONTEXT"'")].cluster.server}')
    export USER=${CLIENT}
    export CRT=$(cat ${CLIENT}.crt | base64 -w 0)
    export KEY=$(cat ${CLIENT}.key | base64 -w 0)
    export NAMESPACE=$namespace
    
    #Configure kubeconfig file
    curl https://raw.githubusercontent.com/shamimice03/Kubernetes_RBAC/main/kubeconfig-template.yaml | sed "s#<context>#${CONTEXT}# ;
    s#<cluster-name>#${CONTEXT}# ;
    s#<ca.crt>#${CA_CRT}# ;
    s#<cluster-endpoint>#${CLUSTER_ENDPOINT}# ;
    s#<user-name>#${USER}# ;
    s#<namespace>#${NAMESPACE}# ;
    s#<user.crt>#${CRT}# ; 
    s#<user.key>#${KEY}#" > config

else
    echo -e "See you next time, Good Luck.\n" 
    exit
fi
```
```bash
chmod +x generate_kubeconfig.sh
bash generate_kubeconfig.sh
```