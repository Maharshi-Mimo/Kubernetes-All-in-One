# Kubernetes Cluster Architecture 

- We will start with Cluster Architecture

![Kubernetes Cluster Architecture](/images/kubernetes-cluster-architecture.svg)

## Key Attributes of the Clusters 

- **Node** : A node is a worker machine in Kubernetes, that takes care of the workload for a particular application(s). Each node is maintained by Controlplane and has components like Kubelet, Container Runtime Interface(CRI) and Kubeproxy. 

- **Kubelet** : This is the primary "node agent" that runs on each node. The kubelet works in terms of PodSpec (a yaml or json object that describes a pod)

- **kube-proxy** : The kubernetes network proxy runs on each node. This reflects services as defined in the Kubernetes API on each node and can do simple TCP, UDP, and SCTP stream forwarding or round-robin TCP, UDP, and SCTP forwarding across a set of backends. Services can be of many flavors: ClusterIP and NodePort

- **Controlplane** : Container orchestration layer that exposes the API and interfaces to define, deploy, and manage the lifecycle of containers. 
It contains 
  - etcd 
  - API server
  - Scheduler 
  - Controller Manager
  - Cloud Controller Manager

