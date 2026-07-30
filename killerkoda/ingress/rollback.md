### Rollback and Troubleshoot

```bash
helm upgrade my-nginx-ingress nginx-stable/nginx-ingress \
  --namespace nginx-ingress \
  --set controller.image.tag="invalid-tag" \
  --set controller.replicaCount=5

kubectl get pods -n nginx-ingress -w

helm status my-nginx-ingress -n nginx-ingress
helm history my-nginx-ingress -n nginx-ingress

helm rollback my-nginx-ingress 3 -n nginx-ingress

helm status my-nginx-ingress -n nginx-ingress

helm history my-nginx-ingress -n nginx-ingress

kubectl get pods -n nginx-ingress

helm get values my-nginx-ingress -n nginx-ingress

helm get all my-nginx-ingress -n nginx-ingress
```
```bash
kubectl config current-context

```