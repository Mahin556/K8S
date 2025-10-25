```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  namespace: sandbox
spec:
  selector:
    matchLabels:
      run: nginx
      app: dsp
      tier: frontend
  replicas: 2
  template:
    metadata:
      labels:
        run: nginx
        app: dsp
        tier: frontend
    spec:
      containers:
      - name: nginx
        image: nginx
        env:
        # Define the environment variable
        - name: nginx-conf
          valueFrom:
            configMapKeyRef:
              # The ConfigMap containing the value you want to assign to SPECIAL_LEVEL_KEY
              name: nginx-conf
              # Specify the key associated with the value
              key: nginx.conf
        resources:
          limits:
            memory: "128Mi"
            cpu: "500m"
        ports:
containerPort: 80 
```

* nginx-conf
```bash
 # The identifier Backend is internal to nginx, and used to name this specific upstream
upstream Backend {
    # hello is the internal DNS name used by the backend Service inside Kubernetes
    server dsp;
}

server {
    listen 80;

    location / {
        # The following statement will proxy traffic to the upstream named Backend
        proxy_pass http://Backend;
    }
} 
```
```bash
kubectl create configmap -n sandbox nginx-conf --from-file=apps/nginx.conf
```

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-conf
data:
  nginx.conf: |
    # The identifier Backend is internal to nginx, and used to name this specific upstream
    upstream Backend {
        # hello is the internal DNS name used by the backend Service inside Kubernetes
        server dsp;
    }
    ...
} 
```
```yaml
containers:
- name: nginx
  image: nginx
  volumeMounts:
  - mountPath: /etc/nginx
    name: nginx-conf
volumes:
- name: nginx-conf
  configMap: 
    name: nginx-conf
    items:
      - key: nginx.conf
        path: nginx.conf
```
```bash
kubectl create configmap nginx-conf --from-file=nginx.conf
```


### References:
- https://stackoverflow.com/questions/71058097/how-do-i-attach-a-configmap-to-a-deployment-in-kubernetes