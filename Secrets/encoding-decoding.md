```yaml
apiVersion: v1
kind: Secret
metadata:
 name: my-secret
type: Opaque
data:
 password: my-awesome-password
```
* We can't create a secret with plane text data, we first need to encode it.
```bash
kubectl apply -f secret.yaml   
Error from server (BadRequest): error when creating "secret.yaml": Secret in version "v1" cannot be handled as a Secret: illegal base64 data at input byte 2
```
```bash
echo -n "my-awesome-password" | base64               
bXktYXdlc29tZS1wYXNzd29yZA==
```
* Replace
```yaml
apiVersion: v1
kind: Secret
metadata:
 name: my-secret
type: Opaque
data:
 password: bXktYXdlc29tZS1wYXNzd29yZA==
```
```bash
kubectl apply -f secret.yaml         
secret/my-secret created
```
* We can get the Secret from Kubernetes by running the following command:
```bash
kubectl get secret my-secret -o jsonpath='{.data.password}'
bXktYXdlc29tZS1wYXNzd29yZA==%
```

* To decode the Secret, we can easily use base64 again:
```bash
kubectl get secret my-secret -o jsonpath='{.data.password}' | base64 --decode
my-awesome-password%
```

* Imparative way
```bash
kubectl create secret generic my-other-secret --from-literal=password=secret
secret/my-other-secret created

kubectl get secret my-other-secret -o jsonpath='{.data.password}'
c2VjcmV0%

kubectl get secret my-other-secret -o jsonpath='{.data.password}' | base64 --decode
secret%
```


### References:
- https://spacelift.io/blog/kubernetes-secrets