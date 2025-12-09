### References:-
- https://freedium-mirror.cfd/https://harsh05.medium.com/kubernetes-service-accounts-simplifying-authentication-and-authorization-07d5c50d2e77

* Using a Service Account with a Kubernetes CronJob
* A CronJob can run tasks on a schedule (like crontab).
* If the CronJob needs access to the Kubernetes API, you must attach a ServiceAccount that has the proper RBAC permissions.
    ```yaml
    apiVersion: batch/v1
    kind: CronJob
    metadata:
    name: kubernetes-cron-job
    namespace: webapps
    spec:
    schedule: "0,15,30,45 * * * *"  # Runs every 15 minutes
    jobTemplate:
        spec:
        template:
            metadata:
            labels:
                app: cron-batch-job
            spec:
            restartPolicy: OnFailure
            serviceAccountName: app-service-account
            containers:
            - name: kube-cron-job
                image: devopscube/kubernetes-job-demo:latest
                args: ["100"]
    ```
* The Pod will receive a short-lived token mounted at: `/var/run/secrets/kubernetes.io/serviceaccount/token`
* The Pod gets permissions defined by your Role + RoleBinding
* It can only perform actions allowed in app-role and ONLY in webapps namespace

* Using Service Account With Kubernetes Deployment
    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
    name: nginx-deployment
    labels:
        app: nginx
    spec:
    replicas: 3
    selector:
        matchLabels:
        app: nginx
    template:
        metadata:
        labels:
            app: nginx
        spec:
        serviceAccountName: app-service-account
        containers:
        - name: nginx
            image: nginx:1.14.2
            ports:
            - containerPort: 80
    ```