### Labels
- Add, update, or remove labels on Kubernetes resources.
- Labels consist of key-value pairs attached to resources, used to organize, filter, and select resources to target them.

```
kubectl label [--overwrite] (-f FILENAME | TYPE NAME) KEY_1=VAL_1 ... KEY_N=VAL_N [--resource-version=version]
```

- Update pod 'foo' with the label 'unhealthy' and the value 'true'
```
kubectl label pods foo unhealthy=true
```

- Update pod 'foo' with the label 'status' and the value 'unhealthy', overwriting any existing value
```
kubectl label --overwrite pods foo status=unhealthy
```  

- Update all pods in the namespace
```
kubectl label pods --all status=unhealthy
```
  
- Update a pod identified by the type and name in "pod.json"
```
kubectl label -f pod.json status=unhealthy
```
 
- Update pod 'foo' only if the resource is unchanged from version 1
```
kubectl label pods foo status=unhealthy --resource-version=1
```
  
- Update pod 'foo' by removing a label named 'bar' if it exists Does not require the --overwrite flag
```
kubectl label pods foo bar-
```

- Apply label to all resources of that type.
```
kubectl label pod --all environment=prod
```

- Apply label to all resources of that type across all namespaces.
```
kubectl label pods -A tier=frontend
```

- --dry-run
  Test without applying:
    - none → actually apply (default)
    - client → simulate locally
    - server → send request, no persistence
```
kubectl label pod foo team=dev --dry-run=client
```

- Track ownership of field changes.
```
--field-manager
kubectl label pods foo role=db --field-manager=my-script
```

- Filter resources by fields.
```
--field-selector
kubectl label pods --field-selector status.phase=Running owner=team1
```

- Apply labels from a file/directory/URL.
```
-f, --filename
kubectl label -f deployment.yaml app=web
```

- Apply labels on resources defined in a kustomize directory.
```
-k, --kustomize
kubectl label -k ./kustomize-dir team=infra
```

- Show labels instead of modifying.
```
--list
kubectl label pods foo --list
```

- Modify YAML locally, no API call.
```
--local
kubectl label -f pod.yaml env=staging --local -o yaml
```

- Output format (json, yaml, name, etc).
```
-o, --output
kubectl label -f pod.yaml app=api --dry-run=client -o yaml
```

- Process all manifests in a directory.
```
-R, --recursive
kubectl label -f ./manifests -R team=backend
```

- Ensure update only if resource version matches.
```
--resource-version
kubectl label pods foo stage=prod --resource-version=3
```

- Select resources by labels.
```
-l, --selector
kubectl label pods -l app=nginx release=stable
```

- Keep managedFields when outputting.
```
--show-managed-fields
kubectl label -f pod.yaml env=test --dry-run=client -o yaml --show-managed-fields
```

- Custom output formatting.
```
--template
kubectl label pods foo app=web -o go-template --template='{{.metadata.labels}}'
```

- Specify namespace
```
-n, --namespace
kubectl label pods foo env=prod -n test
```

- Use a specific kubeconfig file
```
--kubeconfig
kubectl label pods foo app=db --kubeconfig=/path/config
```

- Run as another user
```
--as
kubectl label pods foo owner=team --as admin
```

- Specify API server
```
-s, --server
kubectl label pods foo zone=us-east -s https://my-api:6443
```

- others
```
kubectl label nodes <node-name> app=frontend

kubectl label nodes -l node-role.kubernetes.io/worker=true app=backend

kubectl get nodes --show-labels

kubectl label nodes <node-name> app=database --overwrite

kubectl label nodes <node-name> app-

```


### References
- https://kubernetes.io/docs/reference/kubectl/generated/kubectl_label/
