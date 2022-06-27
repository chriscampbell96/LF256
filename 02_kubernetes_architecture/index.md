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
- The whole point of Kubernetes is to orchestrate the life cycle of a container.
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

The Kubernetes master runs various server and manager processes for the cluster. Among the components of the master node are the kube-apiserver, the kube-scheduler, and the etcd database. As the software has matured, new components have been created to handle dedicated needs, such as the cloud-controller-manager; it handles tasks once handled by the kube-controller-manager to interact with other tools, such as Rancher or DigitalOcean for third-party cluster management and reporting.

There are several add-ons which have become essential to a typical production cluster, such as DNS services. Others are third-party solutions where Kubernetes has not yet developed a local component, such as cluster-level logging and resource monitoring.

- Components:
    - __kube-apiserver__
        - The kube-apiserver is central to the operation of the Kubernetes cluster. 
        - All calls, both internal and external traffic, are handled via this agent. All actions are accepted and validated by this agent, and it is the only agent which connects to the etcd database. As a result, it acts as a master process for the entire cluster, and acts as a frontend of the cluster's shared state. Each API call goes through three steps: authentication, authorization, and several admission controllers.
    - __kube-scheduler__
        - The kube-scheduler uses an algorithm to determine which node will host a Pod of containers. The scheduler will try to view available resources (such as available CPU) to bind, and then assign the Pod based on availability and success. The scheduler uses pod-count by default, but complex configuration is often done if cluster-wide metrics are collected.
        - There are several ways you can affect the algorithm, or a custom scheduler could be used simultaneously instead. A Pod can also be assigned bind to a particular node in the pod spec, though the Pod may remain in a pending state if the node or other declared resource is unavailable.
        - One of the first configurations referenced during creation is if the Pod can be deployed within the current quota restrictions. If so, then the taints and tolerations, and labels of the Pods are used along with those of the nodes to determine the proper placement. Some is done as an admission controller in the kube-apiserver, the rest is done by the chosen scheduler.
    - __etc-database__
        - +tree key-value store. Rather than finding and changing an entry, values are always appended to the end. Previous copies of the data are then marked for future removal by a compaction process. It works with curl and other HTTP libraries, and provides reliable watch queries.
        - Simultaneous requests to update a particular value all travel via the kube-apiserver, which then passes along the request to etcd in a series. The first request would update the database. The second request would no longer have the same version number as found in the object, in which case the kube-apiserver would reply with an error 409 to the requester. There is no logic past that response on the server side, meaning the client needs to expect this and act upon the denial to update. 
        - There is a cp database along with possible followers. They communicate with each other on an ongoing basis to determine which will be master, and determine another in the event of failure. While very fast and potentially durable, there have been some hiccups with some features like whole cluster upgrades. The kubeadm cluster creation tool allows easy deployment of a multi-master cluster with stacked etcd or an external database cluster.
    - __kube-controller-manager__
        - The kube-controller-manager is a core control loop daemon which interacts with the kube-apiserver to determine the state of the cluster. If the state does not match, the manager will contact the necessary controller to match the desired state. There are several controllers in use, such as endpoints, namespace, and replication. The full list has expanded as Kubernetes has matured. 

### Worker Nodes

- Run the kublet, kube-proxy, container engine
- The kubelet interacts with the underlying Docker Engine also installed on all the nodes
- The kube-proxy is in charge of managing the network connectivity to the containers. It does so through the use of iptables entries. It also has the userspace mode, in which it monitors Services and Endpoints using a random high-number port to proxy traffic. Use of ipvs can be enabled, with the expectation it will become the default, replacing iptables.


### Pods
- Smallest unit of work Kubernetes can work with
- Pods typcally follow a one-process-per-container architecture
- Containers in a Pod are started in parallel by default. As a result, there is no way to determine which container becomes available first inside a Pod. initContainers can be used to ensure some containers are ready before others in a pod. To support a single process running in a container, you may need logging, a proxy, or special adapter. These tasks are often handled by other containers in the same Pod.
- 1 IP per Pod

### Services
- Each Service is a microservice handling a particular bit of traffic, such as a single NodePort or a LoadBalancer to distribute inbound requests among many Pods.
- Handle access policies for inbound reuests
- A service, as well as kubectl, uses a selector in order to know which objects to connect. There are two selectors currently supported:

    - __equality-based__: Filters by label keys and their values. Three operators can be used, such as =, ==, and !=. If multiple values or keys are used, all must be included for a match.
    - __set-based__: Filters according to a set of values. The operators are in, notin, and exists. For example, the use of status notin (dev, test, maint) would select resources with the key of status which did not have a value of dev, test, nor maint.

### Operators
- AKA watch-loop and controlles
- Query current statue and compare to spec
- The endpoints, namespace, and serviceaccounts operators each manage the eponymous resources for Pods.

### Single IP per Pod