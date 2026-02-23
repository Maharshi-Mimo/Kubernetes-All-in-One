# Kubernetes Cluster Architecture 

- We will start with Cluster Architecture

![Kubernetes Cluster Architecture](/images/kubernetes-cluster-architecture.svg)

## Key Attributes of the Clusters 

- [**Node**](/docs/02%20-%20Core%20Concepts/10-Nodes.md) : A node is a worker machine in Kubernetes, that takes care of the workload for a particular application(s). Each node is maintained by Controlplane and has components like Kubelet, Container Runtime Interface(CRI) and Kubeproxy. 

- [**Kubelet**](/docs/02%20-%20Core%20Concepts/07-Kubelet.md) : This is the primary "node agent" that runs on each node. The kubelet works in terms of PodSpec (a yaml or json object that describes a pod)

- **kube-proxy** : The kubernetes network proxy runs on each node. This reflects services as defined in the Kubernetes API on each node and can do simple TCP, UDP, and SCTP stream forwarding or round-robin TCP, UDP, and SCTP forwarding across a set of backends. Services can be of many flavors: ClusterIP and NodePort

- **Controlplane** : Container orchestration layer that exposes the API and interfaces to define, deploy, and manage the lifecycle of containers. 
It contains 
  - etcd 
  - API server
  - Scheduler 
  - Controller Manager
  - Cloud Controller Manager

## Kubernetes Components 

![Kubernetes Components](images/components-of-kubernetes.svg)

### Core Component 

- Controlplane Components - manages overall state of the cluster
  - [Kube-apiserver](/docs/02%20-%20Core%20Concepts/02-Kube-API-Server.md) 
  - [etcd](/docs/02%20-%20Core%20Concepts/03-ETCD-server.md)
  - [kube-scheduler](/docs/02%20-%20Core%20Concepts/04-Kube-Scheduler.md)
  - [kube-controller-manager](/docs/02%20-%20Core%20Concepts/05-Kube-Controller-Manager.md)
  - [cloud-controller-manager](/docs/02%20-%20Core%20Concepts/)

### Node Components 
- [Kubelet](/docs/02%20-%20Core%20Concepts/07-Kubelet.md)
- [Kube-proxy](/docs/02%20-%20Core%20Concepts/) 
- [Container Runtime](/docs/02%20-%20Core%20Concepts/)

### Addons 
- [DNS](/docs/02%20-%20Core%20Concepts/)
- [Container Resource Monitoring](/docs/02%20-%20Core%20Concepts/)
- [Cluster-level Logging](/docs/02%20-%20Core%20Concepts/)