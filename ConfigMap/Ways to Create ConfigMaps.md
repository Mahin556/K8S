```bash
#From Literal Values
kubectl create configmap app-config --from-literal=key1=value1 --from-literal=key2=value2

#From a File
kubectl create configmap app-config --from-file=app.properties

#From an Env File (.env)
kubectl create configmap app-config --from-env-file=application.env

```

### References:
- https://www.groundcover.com/blog/kubernetes-configmap