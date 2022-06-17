# Kubernetes Architecture

## Learning Objectives

By the end of this section, you should be able to:

    - Understand the most common terms related to Kubernetes.
    - Discuss history of Kubernetes.
    - Learn about control plane node components.
    - Learn about worker node components.
    - Understand the Container Network Interface (CNI) configuration and Network Plugins.


### What is Kubernetes?

- An open-source system for automating deployment, scaling, and management of containerized applications
- *The captin of a ship of containers*
- *Kubernetes is an orchestration system to deploy and manage containers*


### Components of Kubernetes

- Works by deploying a large number of small webservers/microservices.
- one or more microservice = replica
- This proces allows for decoupling
- Services tie traffic from one service/agent to another 
- Kubernetes is written in golang

### Challenges

- Managing containers at scal and architecting distributed systems
  - Need CI pipeline to buid and test
  - Another cluster to deploy and run your services
  - Monitoring containers when they fail and recreate
  - Ability to perform updates and rollbacks
  - Tear down when no longer needed

*All of these actions require flexible, scalable, and easy-to-manage network and storage.​ As containers are launched on any worker node, the network must join the resource to other containers, while still keeping the traffic secure from others. We also need a storage structure which provides and keeps or recycles storage in a seamless manner.*

### Kubernetes Architecture

![kubernetes_architecture](kubernetes_architecture.png)

- Made up of one or more master/manager and worked nodes
- The manager run an API server, scheduler, different operators and datastore
- The manager keeps the container settings, cluster state and the network configuration
- You are communicating with the API server via `kubectl`
- Each node in the cluster runs 2 containers
  - __kubelet__: receives spec information for container configuration, downloads and manages any necessary resources and works with the container engine on the local node to ensure the container runs or is restarted upon failure
  - __kube-proxy__: container creates and manages local firewall rules and networking configuration to expose containers on the network

### Terminology

- __Pod__
  - Pod consists of one or more containers which share the same IP, access to storage and namespace
  - Usually, one container in a Pod runs an application, while other containers support the primary application.

- __Orchestration__
  - Managed through a series of watch-loops (AKA Operators or Controllers)
  - Each operator interrogates the kube-apiserver for a particular object state, modifying the object until the declared state matches the current state

- __Deployment__
  - The default, newest, and feature-filled operator for containers is a Deployment.
  - Manages a different operator called a ReplicaSet

- __Replica Set__
  - An operator which deploys multiple pods, each with the same spec information (called replicas)

- __Daemon Set__
    - Ensures that a single pod is deployed on every node
    - Often for logging, metrics and security

- __Stateful Set__
  - Used to deploy pods in a particular order, such that following pods are only deployed if previous pods report a ready status. 
  - This is useful for legacy applications which are not cloud-friendly

- __Label__
  - Part of object metadata
  - Used as selectors which can then be used when checking or changing the state of objects without having to know individual names or UIDs

- __Taints__
  -  An arbitrary string in the node metadata, to inform the scheduler on Pod assignments used along with toleration in Pod metadata, indicating it should be scheduled on a node with the particular taint

- __Multi-tenancy__
  - Multiple users and teams share access to one or more clusters

- __Namespace_
  - A segregation of resources, upon which resource quotas and permissions can be applied. Kubernetes objects may be created in a namespace or cluster-scoped. Users can be limited by the object verbs allowed per namespace. Also the LimitRange admission controller constrains resource usage in that namespace. Two objects cannot have the same Name: value in the same namespace.

- __context__
  - A combination of user, cluster name and namespace. A convenient way to switch between combinations of permissions and restrictions. For example you may have a development cluster and a production cluster, or may be part of both the operations and architecture namespaces. This information is referenced from `~/.kube/config`.

- __Resource Limits__
  - A way to limit the amount of resources consumed by a pod, or to request a minimum amount of resources reserved, but not necessarily consumed, by a pod. Limits can also be set per-namespaces, which have priority over those in the PodSpec.

- __Pod Security Policies__
  - Deprecated. Was a policy to limit the ability of pods to elevate permissions or modify the node upon which they are scheduled. This wide-ranging limitation may prevent a pod from operating properly. This was replaced with Pod Security Admission. Some have gone towards Open Policy Agent, or other tools instead.

- __Pod Security Admission__
    - A beta feature to restrict pod behavior in an easy-to-implement and easy-to-understand manner, applied at the namespace level when a pod is created. These will leverage three profiles: Privileged, Baseline, and Restricted policies.

- __Network Policies__
  - The ability to have an inside-the-cluster firewall. Ingress and Egress traffic can be limited according to namespaces and labels as well as typical network traffic characteristics.

  ### Control Plane Node