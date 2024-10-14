# Node

**Kubernetes runs your workload by placing containers into Pods to run on Nodes. A node may be a virtual or physical machine, depending on the cluster. Each node is managed by the control plane and contains the services necessary to run Pods.**

## Node Management

There are two main way to have Node add to the K8s API server

1. The kubelet on a node self-registers to the control plane
2. Manually add a Node object for the Controller to check. 

An example of json manifest of the Node object

```json
{
  "kind": "Node",
  "apiVersion": "v1",
  "metadata": {
    "name": "10.240.79.157",
    "labels": {
      "name": "my-first-k8s-node"
    }
  }
}
```
>[!Note]
> Kubernetes keeps the object for the invalid Node and continues checking to see whether it becomes healthy.
> The Node object name (`metadata`) must be a valid DNS subdomain name. 