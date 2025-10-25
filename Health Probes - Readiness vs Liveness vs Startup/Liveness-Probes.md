### References:-
- https://spacelift.io/blog/kubernetes-liveness-probe

---

* A Kubernetes liveness probe is a health-check mechanism that determines whether a container is still running and functioning correctly.

* If a liveness probe fails, Kubernetes assumes the container is unhealthy and automatically restarts it to restore normal operation — ensuring application reliability and high availability.

* Purpose of Liveness Probes
    * Liveness probes help Kubernetes detect and fix problems that the application cannot recover from on its own, such as:
        * Deadlocks (when an application is stuck and not progressing)
        * Crashes or unresponsive processes
        * Memory leaks or infinite loops causing hangs
    * By restarting the container when these conditions occur, Kubernetes ensures minimal downtime and maintains service health automatically.

