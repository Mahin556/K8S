* **Basic ConfigMap (key-value in YAML)**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: mysql-config
data:
  MYSQL_HOST: "mysql.dev.local"
  MYSQL_PORT: "3306"
  MYSQL_DATABASE: "mydb"
```
```bash
kubectl apply -f configmap.yaml
```
* Most common method
* Used in production

---

* **Using `stringData` (alternative to data)**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
stringData:
  APP_ENV: "dev"
  DEBUG: "true"
```
* Kubernetes converts it into `data` internally

---

* **File-style ConfigMap (multi-line config)**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-config
data:
  nginx.conf: |
    server {
      listen 80;
      location / {
        return 200 "Hello";
      }
    }
```
* Used for:
  * nginx configs
  * app config files

---

* **Binary data (rare)**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: binary-config
binaryData:
  file.bin: "aGVsbG8="   # base64 encoded
```
* For non-text data

---

* **Multiple ConfigMaps in one YAML**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: config1
data:
  key1: value1
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: config2
data:
  key2: value2
```

---

* **Generate YAML (imperative → declarative hybrid)**
```bash
kubectl create configmap mysql-config \
  --from-literal=MYSQL_HOST=mysql.dev.local \
  --dry-run=client -o yaml > configmap.yaml
```

* Then edit + apply
* Common in real DevOps workflows

---

* **Kustomize (advanced declarative)**
```yaml
configMapGenerator:
  - name: mysql-config
    literals:
      - MYSQL_HOST=mysql.dev.local
      - MYSQL_PORT=3306
```
```bash id="cmd3"
kubectl apply -k .
```
* Used in CI/CD pipelines

---
**Update configmap value from imparative command**
```bash
kubectl create configmap demo1 --from-literal=app=demo1 --from-literal=env=prod -o yaml --dry-run=client | kubectl apply -f -

kubectl edit configmap my-configmap
```

---

**ConfigMap with both data and binaryData**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: demo-config
data:
  app_name: "demo-app"
  environment: "production"
binaryData:
  file_template: RGVtbwo=    # Base64 for "Demo\n"
```
**data** ---> Stores key-value pairs as plain text strings, uses for configs, env vars, YAML, JSON, INI, etc. (plain text).
**binaryData** ---> Stores Base64-encoded binary values (e.g., compiled files, templates, images) uses for certificates, images, or other binary content.

* You can decode the binaryData manually:
```bash
echo "RGVtbwo=" | base64 --decode #Demo
```