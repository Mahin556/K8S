- Edit the RS template to the new image (this only updates the template, not existing pods):
```bash
kubectl patch rs nginx-rs-new \
  -p '{"spec":{"template":{"spec":{"containers":[{"name":"nginx","image":"nginx:1.22"}]}}}}'
```

Now check pod images:
```bash
kubectl get pods -l app=nginx -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.spec.containers[0].image}{"\n"}{end}'
# existing pods STILL show nginx:1.19 — template changed but pods not replaced
```
