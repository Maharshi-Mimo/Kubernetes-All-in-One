# Container Runtime Interface 

> **CRI (Container Runtime Interface) is a fundamental component that empowers Kubernetes to run containers effectively. It is responsible for managing the execution and lifecycle of containers within the Kubernetes environment.**

## History of CRI

Prior to the existence of CRI, container runtimes (e.g., `docker`, `rkt`) were
integrated with kubelet through implementing an internal, high-level interface
in kubelet. The entrance barrier for runtimes was high because the integration
required understanding the internals of kubelet and contributing to the main
Kubernetes repository. More importantly, this would not scale because every new
addition incurs a significant maintenance overhead in the main Kubernetes
repository.


> [!Note] 
> The old, pre-CRI Docker integration was removed in 1.7.

## Common CRI in Kubernetes

 - [Containerd](https://containerd.io/)
 - [CRI-O](https://cri-o.io/)
 - [Docker](https://www.docker.com/)
 - [Mirantis](https://www.mirantis.com/)