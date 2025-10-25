* **Cluster configuration files**
  • By default, the kubeconfig file is written to:

  ```
  ${HOME}/.kube/config
  ```

  • If `$KUBECONFIG` environment variable is set, it’s treated as a **list of paths** (separated by `:` on Linux/macOS or `;` on Windows).
  • Behavior:

  * Values are **merged** from all listed paths.
  * If a value is modified → it updates the file where it originally came from.
  * If a new value is added → it goes into the **first file** that exists.
  * If none exist → it creates the **last file** in the list.

* **Specifying a custom kubeconfig file**
  • You can control where the cluster configuration is written with:

  ```bash
  kind create cluster --kubeconfig=myconfig.yaml
  ```

  • In this case:

  * Only `myconfig.yaml` is used.
  * No merging occurs.
  * The flag can only be set **once**.

* **Listing clusters**
  • To see all clusters created by kind:

  ```bash
  kind get clusters
  ```

  • Example output:

  ```
  kind
  kind-2
  ```

* **Interacting with a specific cluster**
  • Each cluster has its own **context** in kubeconfig.
  • Format: `kind-<cluster-name>`
  • To view cluster info:

  ```bash
  kubectl cluster-info --context kind-kind
  kubectl cluster-info --context kind-kind-2
  ```

