- Deploy application
- Deploy fast, debug fast, deploy at scale.
- It have set of tooling that a devops engineer use to deploy and manage application
- Kubernetes is on of the tool.
- Traditionally:-
    - On-premise infa/data-center(ibm,hp,dell),network-engineer(os,rack,setup), sys-admin(lib,dep,deploy), tooling based on application.
    - If want to deploy new version of monolethic application.
        - Monolethic---> easy, fast, not-flexible, limited tech stack, whole app deploy at once, whole app get down if anything fails.
    - While update(required diff env then prod)--->staging

- Virtualization
    - On top of baremetal we can create a multiple VMs with differnet OS to efficiently utilize the underline infra.

- Cloud
    - For small organizations , start-ups
    - Huge Datacenters
    - UI provide to provision infra
    - Rental
    - Regions, Zones
    - On-demand
    - Meant to be cheap, but from the post it is expensive so some organization are swithing back to there own infra(barematel).

- Container
    - 2013
    - docker hype the containerize eco-system
    - help to deploy app
    - monolithic ---> whole app as single instance(multiple service)
        - easy to build
        - managing and updating hard
        - small change need to deploy whole application
    - microservice ---> whole monolithic app broken down to a multipl services(login,signup,payment,home,checkout,dashboard) each service is saperate:
        - Diff tech stacks
        - flexible
        - easy
        - for large app
        - no downtime
        - Application is not affect if one of the service get down
        - each service scale individuall
        - slower then monolithic
        - take time to build
    - container is run a a linux process on a host
    - VM are slice of the Hardware
    - Container are the slice of OS
    - Container engine(docker engine, podman engine etc)
    - Slicing done using linux namespaces, cgroup, union-file-system

![](https://github.com/Mahin556/K8S-artifects/blob/main/images/namespaces.png)

- docker containerization engine
- k8s orchestration engine
- Born out of the architecture of borg or omega 
- CNCF first project
