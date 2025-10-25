In high-security environments, leverage hardware-backed encryption.
Use Kubernetes device plugins to expose node-local encryption hardware such as TPMs (Trusted Platform Modules) to specific Pods.
Trusted workloads can then perform cryptographic operations without exposing keys in memory.


### References:
- https://www.perfectscale.io/blog/kubernetes-secrets