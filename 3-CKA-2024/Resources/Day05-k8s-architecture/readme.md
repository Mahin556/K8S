
# Day 5/40 - What is Kubernetes - Kubernetes Architecture Explained ☸️


## Check out the video below for Day5 👇

[![Day 5/40 - What is Kubernetes - Kubernetes Architecture Explained](https://img.youtube.com/vi/SGGkUCctL4I/sddefault.jpg)](https://youtu.be/SGGkUCctL4I)


## Kubernetes Architecture

![image](https://github.com/piyushsachdeva/CKA-2024/assets/40286378/f15fbf28-5d18-4469-8a28-edd13678cbbf)
- Control plane ---> Run a administrative components
- Worker plane ---> Run a actual workload(pods)
- Pod ---> Encapsulate a container
- For HA we have more then one control node in control plane
- API-server is the center of control plane any request that come to control plane rfirst reach to api-server.
- API-server handle the communication between client-cluster and component-cluster,authenticate, authorize, admission, interact with etcd to store all the data(only api-server can communicate with etcd)
- Scheduler--> Schedule a  workload on a best suitable node(based on resources, request,limits,taint and tolerations, affinity, nodeSelector etc)
- Contorller manager ---> Combination of many diff controllers(monitor the state of workload on the cluster, and take action based on state)
- etcd ---> key/value pair datastore, schemaless, store all the info about cluster, every workload, and every thing about cluster and it's conponents, state, pod, secrets, configMap etc, data first store in etcd and then controllers make state accourding to the desire state in etcd.
- kubelet ---> node agent, receive request from the api-server and then act upon it, delete pods, create pods, responce the api-server request.
- kube-proxy ---> enable n/w with in node, iptable rule, pop-pop communication, and communication through service.
- kubectl ---> client, utility, go, to request to api-server
- every component through REST-api calls, 

## Master/Control plane Node V/s Worker Node ( Node is nothing but a Virtual machine)

![image](https://github.com/piyushsachdeva/CKA-2024/assets/40286378/ef04ec3d-9f3a-4ac5-8a6a-31e877bfabf3)

## ApiServer :- Client interacts with the cluster using ApiServer

![image](https://github.com/piyushsachdeva/CKA-2024/assets/40286378/b8aeb299-9fc9-49da-9c87-0a6eb948ebd1)

## Scheduler: decide which pod to be scheduled on which node based on different factors

![image](https://github.com/piyushsachdeva/CKA-2024/assets/40286378/189208b6-a01e-4e3f-baf9-ae9a9d0f3daf)

## Controller Manager

![image](https://github.com/piyushsachdeva/CKA-2024/assets/40286378/9aece452-6d76-452f-9c89-0f7825151312)

## ETCD Server - Key value database that stores the cluster state and configuration

![image](https://github.com/piyushsachdeva/CKA-2024/assets/40286378/81e037e3-78f0-41a7-8589-f2b4ec3af511)

## Kubelet - Node-level agent that helps container management and receives instructions from Api server

![image](https://github.com/piyushsachdeva/CKA-2024/assets/40286378/bd178509-c49c-4206-bc11-147ac91d2713)

## Kube proxy - Pod to pod communication

![image](https://github.com/piyushsachdeva/CKA-2024/assets/40286378/e99ec3f5-5d73-4554-99d4-a7905d463f64)


