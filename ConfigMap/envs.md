* In Kubernetes (and OpenShift), ConfigMaps are used to manage non-confidential configuration data that your applications need at runtime — such as environment variables, configuration parameters, or feature flags.

If your application uses a .env or .properties file (common in Spring Boot, Node.js, etc.), you don’t have to manually convert each key-value pair into YAML.

Instead, you can create a ConfigMap directly from that file using a single command.

| Use Case                                                            | Object Type   | Command                    |
| ------------------------------------------------------------------- | ------------- | -------------------------- |
| Non-sensitive configuration (e.g., ports, hostnames, feature flags) | **ConfigMap** | `kubectl create configmap` |
| Sensitive configuration (e.g., passwords, tokens, keys)             | **Secret**    | `kubectl create secret`    |


Use ConfigMaps for non-sensitive configuration, and Secrets for anything that should be encrypted (like database passwords or API keys).

* Example: Creating a ConfigMap from an .env or .properties File
```bash
controlplane:~$ cat <<EOF>application.properties
> PORT=8080
MYSQL_HOST=192.168.1.100
MYSQL_DATABASE=my_db
> EOF
```

* **Create the ConfigMap(--from-env-file)**
```bash
kubectl create configmap app-configs --from-env-file=application.properties #K8S
oc create configmap app-configs --from-env-file=application.properties #Openshift
```

* Verify the ConfigMap
```bash
controlplane:~$ kubectl get configmap app-configs -o yaml

apiVersion: v1
kind: ConfigMap
metadata:
  name: app-configs
  namespace: default
data:
  PORT: "8080"
  MYSQL_HOST: "192.168.1.100"
  MYSQL_DATABASE: "my_db"
```
```bash
controlplane:~$ kubectl describe configmap demo2

Name:         demo2
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
MYSQL_DATABASE:
----
my_db

MYSQL_HOST:
----
192.168.1.100

PORT:
----
8080


BinaryData
====

Events:  <none>
```


* **Create the ConfigMap(--from-file)**
```bash
controlplane:~$ kubectl create configmap app-configs --from-file=application.properties
```
```bash
controlplane:~$ kubectl describe configmap demo1

Name:         demo1
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
demo:
----
PORT=8080
MYSQL_HOST=192.168.1.100
MYSQL_DATABASE=my_db



BinaryData
====

Events:  <none>
```

* Namespaced
```bash
kubectl create configmap app-configs --from-env-file=application.properties -n dev-namespace
```

* Environments
```bash
kubectl create configmap app-config-dev --from-env-file=dev.env
kubectl create configmap app-config-prod --from-env-file=prod.env
```

### References:
- https://dulaj.medium.com/kubernetes-env-file-to-config-maps-and-secrets-5bfdb37d8934