# Day 18/40 - Health Probes in Kubernetes

## Check out the video below for Day18 👇

[![Day 18/40 - Health Probes in kubernetes](https://img.youtube.com/vi/x2e6pIBLKzw/sddefault.jpg)](https://youtu.be/x2e6pIBLKzw)


### What are probes?
- To investigate or monitor something and to take necessary actions

### What are health probes in Kubernetes?
- Health probes monitor your Kubernetes applications and take necessary actions to recover from failure
- To ensure your application is highly available and self-healing

### Type of health probes in Kubernetes
- Readiness ( Ensure application is ready)
- Liveness ( Restart the application if health checks fail)
- Startup ( Probes for legacy applications that need a lot of time to start)

### Types of health checks they perform?
- HTTP/TCP/command

### Health probes

![image](https://github.com/user-attachments/assets/95f34a79-4956-4555-b33d-aeddf86653c5)

### Sample YAML

#### liveness-http and readiness-http
``` yaml
apiVersion: v1
kind: Pod
metadata:
  name: hello
spec:
  containers:
  - name: liveness
    image: registry.k8s.io/e2e-test-images/agnhost:2.40
    args:
    - liveness
    livenessProbe:
      httpGet:
        path: /healthz
        port: 8080
      initialDelaySeconds: 15
      periodSeconds: 3
    readinessProbe:
      httpGet:
        path: /healthz
        port: 8080
      initialDelaySeconds: 15
      periodSeconds: 10
```
* This specific mode (liveness) is designed by Kubernetes for testing probe failures intentionally.
It will start returning HTTP 500 responses after a short while — exactly what your logs show:
```bash
Liveness probe failed: HTTP probe failed with statuscode: 500
```
* So in your case, Kubernetes isn’t misconfigured — it’s behaving correctly based on what the container is doing.
* That container simulates an app that fails liveness checks on purpose (to test restarts).
* https://pkg.go.dev/k8s.io/kubernetes/test/images/agnhost#section-readme

#### liveness command

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness
  name: liveness-exec
spec:
  containers:
  - name: liveness
    image: registry.k8s.io/busybox
    args:
    - /bin/sh
    - -c
    - touch /tmp/healthy; sleep 30; rm -f /tmp/healthy; sleep 600
    livenessProbe:
      exec:
        command:
        - cat
        - /tmp/healthy
      initialDelaySeconds: 5
      periodSeconds: 5
```
```bash
Events:
  Type     Reason     Age                From               Message
  ----     ------     ----               ----               -------
  Normal   Scheduled  76s                default-scheduler  Successfully assigned default/liveness-exec to node01
  Normal   Pulled     72s                kubelet            Successfully pulled image "registry.k8s.io/busybox" in 3.443s (3.443s including waiting). Image size: 1144547 bytes.
  Normal   Created    72s                kubelet            Created container: liveness
  Normal   Started    72s                kubelet            Started container liveness
  Warning  Unhealthy  31s (x3 over 41s)  kubelet            Liveness probe failed: cat: can't open '/tmp/healthy': No such file or directory
  Normal   Killing    31s                kubelet            Container liveness failed liveness probe, will be restarted
  Normal   Pulling    1s (x2 over 76s)   kubelet            Pulling image "registry.k8s.io/busybox"
  Normal   Created    10s (x2 over 84s)  kubelet            Created container: liveness
  Normal   Started    10s (x2 over 84s)  kubelet            Started container liveness
  Normal   Pulled     10s                kubelet            Successfully pulled image "registry.k8s.io/busybox" in 2.743s (2.743s including waiting). Image size: 1144547 bytes.
```

#### liveness-tcp

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: tcp-pod
  labels:
    app: tcp-pod
spec:
  containers:
  - name: goproxy
    image: registry.k8s.io/goproxy:0.1
    ports:
    - containerPort: 8080
    livenessProbe:
      tcpSocket:
        port: 3000 #diff port
      initialDelaySeconds: 10
      periodSeconds: 5
```
```bash
Events:
  Type     Reason     Age                From               Message
  ----     ------     ----               ----               -------
  Normal   Scheduled  42s                default-scheduler  Successfully assigned default/tcp-pod to node01
  Normal   Pulling    42s                kubelet            Pulling image "registry.k8s.io/goproxy:0.1"
  Normal   Pulled     38s                kubelet            Successfully pulled image "registry.k8s.io/goproxy:0.1" in 3.691s (3.691s including waiting). Image size: 1698862 bytes.
  Normal   Created    17s (x2 over 38s)  kubelet            Created container: goproxy
  Normal   Started    17s (x2 over 38s)  kubelet            Started container goproxy
  Normal   Killing    17s                kubelet            Container goproxy failed liveness probe, will be restarted
  Normal   Pulled     17s                kubelet            Container image "registry.k8s.io/goproxy:0.1" already present on machine
  Warning  Unhealthy  2s (x5 over 27s)   kubelet            Liveness probe failed: dial tcp 192.168.1.9:3000: connect: connection refused
  Normal   Killing    2m43s (x6 over 5m8s)    kubelet            Container goproxy failed liveness probe, will be restarted
  Warning  BackOff    11s (x17 over 3m48s)    kubelet            Back-off restarting failed container goproxy in pod tcp-pod_default(d47824b2-fe67-4ca9-b031-2866850d30ca)
```
