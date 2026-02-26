```bash
kubectl label node workerone app=frontend env=production
kubectl label node workertwo app=frontend env=staging
```
```bash
#More flexible than selectors. Supports hard and soft rules.
#Example: Prefer frontend nodes, fallback to backend nodes.
kubectl apply -f -<<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend-app
spec:
  replicas: 6
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
              - matchExpressions:
                  - key: app
                    operator: In
                    values:
                      - frontend
          preferredDuringSchedulingIgnoredDuringExecution:
            - weight: 4
              preference:
                matchExpressions:
                  - key: env
                    operator: In
                    values:
                      - production
            - weight: 2
              preference:
                matchExpressions:
                  - key: env
                    operator: In
                    values:
                      - staging
      containers:
        - name: nginx
          image: nginx
EOF
```

### Reference:- 
- https://labex.io/tutorials/kubernetes-how-to-assign-and-manage-custom-labels-on-kubernetes-nodes-415736
