```bash
--user  = use a real user from your kubeconfig
          (must exist under "users:")

--as    = impersonate any user (even if not in kubeconfig)
          requires RBAC: impersonate/users, groups, serviceaccounts

Examples:
kubectl --user=developer get pods
# Uses kubeconfig's developer credentials

kubectl --as=mahinder get pods
# Pretend to be user "mahinder"

kubectl --as=system:serviceaccount:dev:sa1 get pods
# Pretend to be service account dev/sa1

TL;DR:
--user = switch kubeconfig user
--as   = impersonate a user or service account
```