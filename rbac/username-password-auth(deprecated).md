```bash
#Create password file for API server
sudo nano /etc/kubernetes/credentials.csv

mypassword,alice,1001,"developers"

#Start API server with basic auth
vim /etc/kubernetes/manifests/kube-apiserver.yaml

- --basic-auth-file=/etc/kubernetes/credentials.csv

kubectl --username=alice --password=mypassword get pods
Forbidden: User "alice" cannot list pods

kubectl create rolebinding alice-admin \
   --clusterrole=admin \
   --user=alice \
   -n default

kubectl --username=alice --password=mypassword get pods
```
* Username/Password Auth is Deprecated
  Kubernetes versions ≥ 1.19 deprecated:
  ```bash
  --basic-auth-file
  --token-auth-file
  ```
