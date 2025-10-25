```bash
kind create cluster 
# Creates a Kubernetes cluster using a prebuilt node image.
# By default, the cluster is named kind.

kind create cluster --image=<image> #With specific version
# Prebuilt images are hosted at kindest/node.
# To find the right image for your kind version → check the release notes (kind version shows your version).


kind create cluster --image=<image> --name=<name> #With name(as context)
# Default name: kind.

kubectl cluster-info --context kind-kind
kubectl cluster-info --context kind-kind-2

```

##### Waiting for cluster readiness
• By default, the command finishes before the cluster is fully ready.
• To block until control plane is ready, use `--wait <time>`:
```bash
--wait 30s → wait 30 seconds
--wait 5m → wait 5 minutes
```

##### Help command
```bash
kind create cluster --help
```

#### Runtime providers
• kind auto-detects container runtimes (Docker, Podman, nerdctl).
• To force a specific one, set environment variables:
```bash
KIND_EXPERIMENTAL_PROVIDER=docker
KIND_EXPERIMENTAL_PROVIDER=podman
KIND_EXPERIMENTAL_PROVIDER=nerdctl
```

