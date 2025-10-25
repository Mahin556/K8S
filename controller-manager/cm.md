- CM is the k8s control plane component that manages the desire state of k8s cluster using a set of controllers.
- CMs run a non-terminating **controller-loop** through which it monitor the current state(current no of replicas) of the cluster and compare it with the desire state(desired no of replicas).
- Automatically create and delete pod based on the desired count.

- Pod are not reliable way to run the containers so we use controllers what manage the pods and itself managed by CM

- List of controllers:
    - **Deployment:** A higher-level controller for managing stateless applications, like web servers. A Deployment uses ReplicaSets to ensure a specified number of identical pods are running. It handles rolling updates and rollbacks, so you can update an application without downtime.

    - **ReplicaSet:** A lower-level controller whose sole purpose is to ensure that a stable, specific number of pod replicas are running at any given time. Deployments manage ReplicaSets, so it is generally not recommended to use ReplicaSets directly.

    - **StatefulSet:** Used for managing stateful applications, like databases or message queues. It provides stable, unique pod identities and persistent storage volumes. Unlike Deployments, StatefulSets offer ordered deployment, scaling, and termination of pods.

    - **DaemonSet:** Ensures that all (or some) nodes in a cluster run a copy of a specific pod. It is used for running system-level tasks such as log collection daemons (fluentd), node monitoring (Prometheus Node Exporter), or storage daemons.

    - **Job:** Creates one or more pods to run a specific task to completion. A Job tracks the success of the pods and will retry on failure.

    - **CronJob:** Automates the creation of Jobs on a repeating schedule, similar to the cron utility in Unix-like systems. It is suitable for tasks like nightly backups or scheduled reports. 

    - **Node Controller:** Watches nodes for health issues and is responsible for noticing and responding when a node goes down.

    - **Endpoint Controller:** Populates the Endpoints object, which is used by services to determine the IP addresses of the pods that back them.

    - **Service Controller:** Interacts with cloud provider APIs to create and configure load balancers.  

### references
- https://kubernetes.io/docs/concepts/architecture/controller/