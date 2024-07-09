# Kubelet 

- The kubelet is the primary "node agent" on each node.
- It can register the node with the apiserver using:
    - The hostname
    - A flag to override the hostname
    - Specific logic for a cloud provider
- The kubelet operates with a PodSpec, a YAML or JSON object that describes a pod.
- It takes a set of PodSpecs (primarily from the apiserver) and ensures the described containers are running and healthy.
- The kubelet doesn't manage containers not created by Kubernetes.
- A container manifest can be provided to the kubelet in two other ways:
    - File: Path passed as a flag on the command line. Files are monitored periodically (every 20 seconds by default, configurable via a flag).
    - HTTP endpoint: Passed as a parameter on the command line. This endpoint is checked every 20 seconds (also configurable with a flag).