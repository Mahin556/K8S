```bash
helm version
helm repo add nginx-stable https://helm.nginx.com/stable
helm repo list
helm repo update
helm search repo nginx
helm show chart nginx-stable/nginx-ingress
helm show values nginx-stable/nginx-ingress
```
```bash
kubectl create namespace nginx-ingress

helm install my-nginx-ingress nginx-stable/nginx-ingress \
  --namespace nginx-ingress \
  --set controller.replicaCount=2 \
  --set controller.service.type=NodePort \
  --set controller.service.httpPort.nodePort=30080 \
  --set controller.service.httpsPort.nodePort=30443

helm status my-nginx-ingress -n nginx-ingress

helm list -A

kubectl get svc -n nginx-ingress

helm history my-nginx-ingress -n nginx-ingress

helm get manifest my-nginx-ingress -n nginx-ingress
```
```bash
cat << EOF > /root/nginx-values.yaml
# Custom values for nginx-ingress
controller:
  replicaCount: 3
  
  resources:
    limits:
      cpu: 100m
      memory: 128Mi
    requests:
      cpu: 50m
      memory: 64Mi
  
  service:
    type: NodePort
    nodePorts:
      http: 30080
      https: 30443
    annotations:
      service.kubernetes.io/app-name: "nginx-ingress-controller"
      
  metrics:
    enabled: true
    
  podAnnotations:
    prometheus.io/scrape: "true"
    prometheus.io/port: "9113"
EOF

helm upgrade my-nginx-ingress nginx-stable/nginx-ingress \
  --namespace nginx-ingress \
  --values /root/nginx-values.yaml

helm upgrade my-nginx-ingress nginx-stable/nginx-ingress \
  --namespace nginx-ingress \
  --values /root/nginx-values.yaml \
  --set controller.service.annotations."service\.kubernetes\.io/load-balancer-source-ranges"="10.0.0.0/8"

helm status my-nginx-ingress -n nginx-ingress
helm history my-nginx-ingress -n nginx-ingress

kubectl get pods -n nginx-ingress

kubectl describe deployment -n nginx-ingress
```

